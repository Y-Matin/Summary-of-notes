# 工具调用：ToolsNode

## 目标

工具调用让大模型具备访问外部世界的能力，例如查询天气、调用 HTTP 接口、读取数据库、执行业务函数等。

大模型本身不能直接获取实时信息，也不能真正执行代码。工具调用的本质是：模型根据工具描述生成调用意图，Eino 根据意图执行真实工具，再把结果返回给模型。

这也是 AI Agent 和普通聊天机器人的关键分水岭：普通聊天机器人主要“回答”，带工具的 Agent 可以“行动”。模型负责判断任务需要什么能力，程序负责把能力安全、可控地暴露成工具。

工具调用特别适合下面这些场景：

- 实时信息：天气、股票、新闻、订单状态。
- 私有系统：CRM、数据库、内部工单、文件系统。
- 确定性计算：数学计算、格式转换、规则校验。
- 业务动作：创建任务、发送通知、生成报表。

## 核心概念

### ToolsNode

`ToolsNode` 是 Eino 中执行工具调用的组件。它管理一组工具，并根据模型输出的 `ToolCall` 找到对应工具执行。

工作流程：

```text
注册工具到 ToolsNode
  -> 把 ToolInfo 绑定给 ChatModel
  -> 模型生成 ToolCall
  -> ToolsNode 根据名称和参数调用工具
  -> 工具结果封装成 Message
  -> 再交给模型生成最终回答
```

## 工具接口

### BaseTool

所有工具的基础接口，用于向模型暴露工具元信息。

```go
type BaseTool interface {
    Info(ctx context.Context) (*schema.ToolInfo, error)
}
```

`ToolInfo` 重点字段：

```go
type ToolInfo struct {
    Name string
    Desc string
    *ParamsOneOf
}
```

- `Name`：工具名，模型会通过这个名字发起调用。
- `Desc`：告诉模型这个工具何时、为什么、如何使用。
- `ParamsOneOf`：定义工具参数结构、类型、是否必填、枚举等。

### InvokableTool

同步工具接口。

```go
type InvokableTool interface {
    BaseTool
    InvokableRun(ctx context.Context, argumentsInJSON string, opts ...tool.Option) (string, error)
}
```

### StreamableTool

流式工具接口。

```go
type StreamableTool interface {
    BaseTool
    StreamableRun(ctx context.Context, argumentsInJSON string, opts ...tool.Option) (*schema.StreamReader[string], error)
}
```

## 自定义天气工具骨架

```go
type WeatherTool struct {
    apiKey string
}

func (w *WeatherTool) Info(ctx context.Context) (*schema.ToolInfo, error) {
    return &schema.ToolInfo{
        Name: "get_weather",
        Desc: "获取指定城市的天气信息",
        ParamsOneOf: schema.NewParamsOneOfByParams(map[string]*schema.ParameterInfo{
            "city": {
                Type:     schema.String,
                Required: true,
                Desc:     "城市名称",
            },
            "extensions": {
                Type: schema.String,
                Enum: []string{"base", "all"},
                Desc: "base 表示实况天气，all 表示预报天气",
            },
        }),
    }, nil
}

func (w *WeatherTool) InvokableRun(ctx context.Context, args string, opts ...tool.Option) (string, error) {
    var params struct {
        City       string `json:"city"`
        Extensions string `json:"extensions"`
    }
    if err := json.Unmarshal([]byte(args), &params); err != nil {
        return "", err
    }
    if params.City == "" {
        return "", fmt.Errorf("city is required")
    }

    // 这里调用真实天气 API，并返回 JSON 字符串。
    return callWeatherAPI(ctx, w.apiKey, params.City, params.Extensions)
}
```

## 创建 ToolsNode

```go
weatherTool := &WeatherTool{apiKey: "your amap api key"}

toolsNode, err := compose.NewToolNode(ctx, &compose.ToolsNodeConfig{
    Tools: []tool.BaseTool{weatherTool},
})
if err != nil {
    panic(err)
}
```

## 手动模拟 ToolCall

这个步骤用于理解 `ToolsNode` 如何根据模型输出执行工具。

```go
input := &schema.Message{
    Role: schema.Assistant,
    ToolCalls: []schema.ToolCall{
        {
            Function: schema.FunctionCall{
                Name:      "get_weather",
                Arguments: `{"city":"上海","extensions":"base"}`,
            },
        },
    },
}

toolMessages, err := toolsNode.Invoke(ctx, input)
```

## 与 ChatModel 结合

要让模型知道有哪些工具，需要把工具信息绑定给模型。

```go
info, err := weatherTool.Info(ctx)
if err != nil {
    panic(err)
}

toolCallingModel, err := model.WithTools([]*schema.ToolInfo{info})
if err != nil {
    panic(err)
}

messages := []*schema.Message{
    schema.SystemMessage("你必须调用工具获取天气信息"),
    schema.UserMessage("查询北京今天的天气"),
}

assistantMsg, err := toolCallingModel.Generate(ctx, messages)
toolMessages, err := toolsNode.Invoke(ctx, assistantMsg)
```

完整应用通常还会把 `toolMessages` 追加回上下文，再调用模型生成自然语言总结。

## 记忆点

- `ToolInfo` 是给模型看的，`InvokableRun` 是给程序执行的。
- 工具描述越清晰，模型越容易正确选择工具和参数。
- `WithTools` / `BindTools` 负责把工具能力告诉模型。
- `ToolsNode` 不负责推理，只负责执行模型已经决定的 ToolCall。
- 工具参数建议使用结构体或 JSON schema 思维设计，避免字符串随意拼接。
