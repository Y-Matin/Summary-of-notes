# 编排：Graph、Chain、Workflow

## 目标

AI 应用通常不是单次模型调用，而是多个组件按业务逻辑组合：提示词、模型、工具、检索、转换、后处理等。Eino 提供 `Graph`、`Chain`、`Workflow` 三种编排方式。

随着应用复杂度上升，代码里会出现很多“先调模板、再调模型、再调工具、再转换结果、再调模型”的流程。如果这些流程全部手写，组件之间的输入输出关系会散落在业务代码里，很难复用、调试和扩展。

编排解决的就是“组件之间如何连接”的问题。它把模型、工具、模板、检索器、Lambda 转换节点都看成节点，再通过边或字段映射描述执行顺序和数据流向。

## 三者区别

| 编排方式 | 特点 | 适用场景 |
| --- | --- | --- |
| Graph | 节点 + 边，能力最强，支持复杂拓扑 | 分支、并行、循环、复杂 Agent 流程 |
| Chain | Graph 的线性封装，写法更简单 | 简单串联流程 |
| Workflow | 支持字段级映射 | 输入输出字段明确的业务流程 |

## 1. Graph

### 概念

Graph 基于节点和边描述数据流。

```text
START
  -> ChatTemplate
  -> ChatModel
  -> ToolsNode
  -> Lambda
  -> ChatModel
  -> END
```

### 房产经纪人示例流程

需求：用户提供姓名和邮箱，系统查询用户信息，再根据职位、薪资推荐房产。

核心节点：

- `prompt`：根据用户输入生成模型消息。
- `chat`：模型判断并生成工具调用。
- `tools`：调用 `user_info` 工具。
- `lambda`：把工具结果转换成下游可用格式。
- `lambdaPrompt`：把工具结果放入推荐提示词。
- `chat_recommend`：模型生成最终推荐。

### Graph 代码骨架

```go
g := compose.NewGraph[map[string]any, *schema.Message]()

_ = g.AddChatTemplateNode("prompt", chatTpl)
_ = g.AddChatModelNode("chat", chatModel)
_ = g.AddToolsNode("tools", toolsNode)
_ = g.AddLambdaNode("lambda", transformLambda)
_ = g.AddLambdaNode("lambdaPrompt", promptLambda)
_ = g.AddChatModelNode("chat_recommend", chatModel)

_ = g.AddEdge(compose.START, "prompt")
_ = g.AddEdge("prompt", "chat")
_ = g.AddEdge("chat", "tools")
_ = g.AddEdge("tools", "lambda")
_ = g.AddEdge("lambda", "lambdaPrompt")
_ = g.AddEdge("lambdaPrompt", "chat_recommend")
_ = g.AddEdge("chat_recommend", compose.END)

runnable, err := g.Compile(ctx)
```

运行：

```go
output, err := runnable.Invoke(ctx, map[string]any{
    "histories":  []*schema.Message{},
    "user_query": "我叫 zhangsan，邮箱是 zhangsan@bytedance.com，帮我推荐房产",
})
```

### Lambda 的作用

Lambda 用于在节点之间做数据转换。

例如 `ToolsNode` 输出 `[]*schema.Message`，但下游模型可能需要 `*schema.Message` 或 `[]*schema.Message`：

```go
lambda := compose.TransformableLambda[[]*schema.Message, *schema.Message](
    func(ctx context.Context, input *schema.StreamReader[[]*schema.Message]) (*schema.StreamReader[*schema.Message], error) {
        return schema.StreamReaderWithConvert(input, func(msgs []*schema.Message) (*schema.Message, error) {
            if len(msgs) == 0 {
                return nil, errors.New("no message")
            }
            return msgs[0], nil
        }), nil
    },
)
```

## 2. Chain

### 概念

Chain 是对 Graph 的线性封装。它适合没有复杂分支的流程。

```text
输入
  -> 节点 A
  -> 节点 B
  -> 节点 C
  -> 输出
```

### 代码骨架

```go
chain := compose.NewChain[map[string]any, *schema.Message]()

chain.
    AppendChatTemplate(chatTpl).
    AppendChatModel(chatModel).
    AppendToolsNode(toolsNode).
    AppendLambda(transformLambda).
    AppendLambda(promptLambda).
    AppendChatModel(chatModel)

runnable, err := chain.Compile(ctx)
output, err := runnable.Invoke(ctx, input)
```

### 适合使用 Chain 的情况

- 节点顺序固定。
- 没有条件分支。
- 没有回路。
- 需要快速搭出一个可运行流程。

## 3. Workflow

### 概念

Workflow 更强调字段级别的输入输出映射。它适合业务流程中每个节点输入输出结构都比较明确的场景。

### 典型用途

```text
用户输入字段
  -> 模板节点需要 user_query / histories
  -> 工具节点输出 toolsInfo
  -> 推荐节点需要 toolsInfo
  -> 最终输出 Message
```

### 代码骨架

```go
wf := compose.NewWorkflow[map[string]any, *schema.Message]()

_ = wf.AddChatTemplateNode("prompt", chatTpl)
_ = wf.AddChatModelNode("chat", chatModel)
_ = wf.AddToolsNode("tools", toolsNode)
_ = wf.AddLambdaNode("build_recommend_prompt", promptLambda)
_ = wf.AddChatModelNode("recommend", chatModel)

// Workflow 会通过字段映射描述节点之间的数据依赖。
runnable, err := wf.Compile(ctx)
```

## 选择建议

- 能用 Chain 就先用 Chain，代码最少。
- 需要分支、循环、并行时用 Graph。
- 数据字段映射复杂，且业务流程更像 DAG 时用 Workflow。
- Agent 底层也可以理解为对 Graph 的封装，学会 Graph 有助于理解 Agent 执行过程。

## 记忆点

- 编排解决的是“组件如何组织成业务流程”。
- Graph 最强，Chain 最顺手，Workflow 最适合字段映射。
- Lambda 是连接不同节点类型的重要胶水。
- 编排时要明确每个节点的输入输出类型，否则泛型和转换会很难排错。
