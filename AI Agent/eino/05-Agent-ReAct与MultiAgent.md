# Agent：ReAct 与 Multi-Agent

## 目标

Agent 是把模型、工具、提示词和执行控制组合起来的更高层抽象。它不只是调用一次模型，而是可以根据任务自动决定是否调用工具、调用哪个工具、是否继续推理。

前面的 `ChatModel` 更像“一次问答”，`ToolsNode` 更像“执行一个工具调用”。Agent 则把二者组合起来，让模型围绕目标进行多步决策：先理解任务，再决定是否调用工具，观察工具结果后继续推理，直到得到最终答案。

学习 Agent 时要注意一个边界：Agent 并不等于“自动万能”。它仍然依赖清晰的系统提示词、可靠的工具定义、可控的最大步数和明确的终止条件。工具描述不清晰、任务边界太模糊，都会让 Agent 表现不稳定。

## 1. ReAct

### 概念

ReAct 是 Reasoning + Acting，即“推理 + 行动”的循环。

它的核心价值是把大模型的语言推理能力和外部工具的确定性能力结合起来。模型负责判断“下一步该做什么”，工具负责提供真实数据或执行真实动作。

流程：

```text
接收用户问题
  -> 模型思考是否需要工具
  -> 如果需要，生成 ToolCall
  -> ToolsNode 执行工具
  -> 模型观察工具结果
  -> 继续思考或生成最终回答
```

### Eino ReAct Agent

Eino 在 `flow/agent/react` 中封装了 ReAct Agent。

```go
agent, err := react.NewAgent(ctx, &react.AgentConfig{
    ToolCallingModel: model,
    ToolsConfig: compose.ToolsNodeConfig{
        Tools: []tool.BaseTool{userInfoTool},
    },
    MaxStep: 12,
})
```

`AgentConfig` 重点字段：

- `ToolCallingModel`：支持工具调用的模型。
- `ToolsConfig`：工具节点配置。
- `MessageModifier`：调用模型前修改消息，可插入上下文或系统提示词。
- `MaxStep`：最大推理步数，防止死循环。
- `ToolReturnDirectly`：部分工具结果可直接返回，不再继续让模型总结。
- `GraphName`、`ModelNodeName`、`ToolsNodeName`：底层 Graph 的命名配置。

### ReAct 示例：房产经纪人

需求：用户提供姓名和邮箱，Agent 调用 `user_info` 工具获取公司、职位、薪资，再推荐房产。

工具定义骨架：

```go
type userInfoRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

type userInfoResponse struct {
    Name     string `json:"name"`
    Email    string `json:"email"`
    Company  string `json:"company"`
    Position string `json:"position"`
    Salary   string `json:"salary"`
}

userInfoTool := utils.NewTool(
    &schema.ToolInfo{
        Name: "user_info",
        Desc: "根据用户姓名和邮箱查询公司、职位、月薪信息",
        ParamsOneOf: schema.NewParamsOneOfByParams(map[string]*schema.ParameterInfo{
            "name":  {Type: schema.String, Desc: "用户姓名"},
            "email": {Type: schema.String, Desc: "用户邮箱"},
        }),
    },
    func(ctx context.Context, input *userInfoRequest) (*userInfoResponse, error) {
        return &userInfoResponse{
            Name:     input.Name,
            Email:    input.Email,
            Company:  "字节跳动",
            Position: "高级工程师",
            Salary:   "60000",
        }, nil
    },
)
```

运行 Agent：

```go
agent, err := react.NewAgent(ctx, &react.AgentConfig{
    ToolCallingModel: model,
    ToolsConfig: compose.ToolsNodeConfig{
        Tools: []tool.BaseTool{userInfoTool},
    },
})

result, err := agent.Generate(ctx, []*schema.Message{
    schema.SystemMessage("你是房产经纪人，请结合用户收入和职位推荐房产。"),
    schema.UserMessage("我叫 zhangsan，邮箱 zhangsan@bytedance.com，帮我推荐房产"),
})
```

## 2. Multi-Agent

### 概念

复杂任务可以拆给多个 Agent 协作。教程中的 Host Multi-Agent 是“主控 + 专家”模式。

当任务涉及多个专业方向时，一个 Agent 的提示词会越来越臃肿，工具也会越来越多，模型选择工具和保持上下文的难度都会上升。Multi-Agent 通过角色分工降低复杂度：主控 Agent 只负责理解和分发任务，专家 Agent 只关注自己的职责范围。

- Host Agent：理解用户意图，决定把任务交给哪个专家。
- Specialist Agents：处理特定领域任务。
- Summarizer：可选，把多个专家结果汇总成最终报告。

### 配置骨架

```go
multiAgent, err := host.NewMultiAgent(ctx, &host.MultiAgentConfig{
    Host:        hostAgent,
    Specialists: []*host.Specialist{writeJournal, readJournal},
    Name:        "journal_assistant",
})
```

`MultiAgentConfig` 重点字段：

- `Host`：主控 Agent。
- `Specialists`：专家 Agent 列表。
- `HostNodeName`：主控节点名称，默认 `host`。
- `StreamToolCallChecker`：流式输出时检查是否存在工具调用。
- `Summarizer`：汇总多个专家输出。

### Multi-Agent 示例：日记助手

需求：用户输入可能是写日记，也可能是读日记。Host 判断意图，再路由给对应专家。

```go
func newHost(ctx context.Context) (*host.Host, error) {
    chatModel, err := newOllamaModel(ctx)
    if err != nil {
        return nil, err
    }

    return &host.Host{
        ToolCallingModel: chatModel,
        SystemPrompt:    "你是日记助手，判断用户是要写日记还是读日记，并调用合适专家。",
    }, nil
}
```

写日记专家：

```go
writeJournal := &host.Specialist{
    ChatModel:     chatModel,
    SystemPrompt:  "请将用户输入写入日记。",
    AgentMeta: host.AgentMeta{
        Name:        "write_journal",
        IntendedUse: "保存用户日记",
    },
    Invokable: func(ctx context.Context, input []*schema.Message, opts ...agent.AgentOption) (*schema.Message, error) {
        return schema.AssistantMessage("日记已保存", nil), nil
    },
}
```

读日记专家与之类似，只是 `IntendedUse` 和执行逻辑不同。

## ReAct 与 Multi-Agent 对比

| 模式 | 适用场景 | 核心关注点 |
| --- | --- | --- |
| ReAct | 单个 Agent 通过工具完成任务 | 工具选择、循环推理、最大步数 |
| Multi-Agent | 多领域、多角色协作 | 任务路由、专家职责、结果汇总 |

## 记忆点

- ReAct 解决“一个 Agent 如何边想边做”。
- Multi-Agent 解决“多个 Agent 如何分工协作”。
- Agent 的能力上限很依赖工具描述、系统提示词和任务边界。
- 一定要设置最大步数或终止条件，避免 Agent 循环调用。
