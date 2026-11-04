# MCP

## 目标

MCP 是 Model Context Protocol，用于让模型应用以标准方式接入外部工具。Eino 通过 `eino-ext/components/tool/mcp` 可以把 MCP Server 暴露的工具转换成 Eino 的 `tool.BaseTool`，再交给 Agent 使用。

MCP 可以理解为“工具接入协议”。没有 MCP 时，每个应用都要用自己的方式接入文件系统、命令行、HTTP API、数据库或第三方服务；有 MCP 后，这些能力可以由独立的 MCP Server 统一暴露，模型应用通过标准协议发现和调用。

通过 MCP，模型应用可以访问文件系统、执行命令行操作、调用外部 API，也可以和其他服务交互。对 Eino 来说，MCP 的价值是把远程工具转换成统一的 Eino Tool，让本地工具和协议工具在 Agent 中以相同方式使用。

## 核心理解

普通 Eino Tool 是在当前程序里实现的工具；MCP Tool 是由独立 MCP Server 提供的工具。

```text
MCP Server 暴露工具
  -> MCP Client 连接 Server
  -> eino-ext 获取 MCP 工具列表
  -> 转为 Eino BaseTool
  -> React Agent / ToolsNode 调用
```

## 安装依赖

```bash
go get github.com/cloudwego/eino-ext/components/tool/mcp@latest
go get github.com/mark3labs/mcp-go
```

## 创建 MCP Server

示例：创建天气查询服务。

```go
mcpServer := server.NewMCPServer("weather", mcp.LATEST_PROTOCOL_VERSION)

mcpServer.AddTool(WeatherTool(), func(ctx context.Context, req mcp.CallToolRequest) (*mcp.CallToolResult, error) {
    args := req.GetArguments()

    city, ok := args["city"].(string)
    if !ok || city == "" {
        return nil, fmt.Errorf("city is required")
    }

    result, err := queryWeather(ctx, city)
    if err != nil {
        return nil, err
    }

    return mcp.NewToolResultText(result), nil
})

err := server.NewSSEServer(mcpServer).Start("localhost:12345")
```

定义 MCP 工具元信息：

```go
func WeatherTool() mcp.Tool {
    return mcp.NewTool(
        "get_weather",
        mcp.WithDescription("Get weather information for a given city"),
        mcp.WithString("city",
            mcp.Required(),
            mcp.Description("城市名称"),
        ),
        mcp.WithString("extensions",
            mcp.Enum("base", "all"),
            mcp.DefaultString("base"),
            mcp.Description("base 为实况天气，all 为预报天气"),
        ),
    )
}
```

## MCP Client 转 Eino Tool

```go
func getMcpTools() ([]tool.BaseTool, error) {
    ctx := context.Background()

    cli, err := client.NewSSEMCPClient("http://localhost:12345/sse")
    if err != nil {
        return nil, err
    }

    if err := cli.Start(ctx); err != nil {
        return nil, err
    }

    initReq := mcp.InitializeRequest{}
    initReq.Params.ProtocolVersion = mcp.LATEST_PROTOCOL_VERSION
    initReq.Params.ClientInfo = mcp.Implementation{
        Name:    "weather-tool",
        Version: "0.0.1",
    }

    if _, err := cli.Initialize(ctx, initReq); err != nil {
        return nil, err
    }

    return mcptool.GetTools(ctx, &mcptool.Config{
        Cli: cli,
    })
}
```

## 接入 ReAct Agent

```go
chatModel, err := ollama.NewChatModel(ctx, &ollama.ChatModelConfig{
    BaseURL: "http://127.0.0.1:11434",
    Model:   "modelscope.cn/Qwen/Qwen3-32B-GGUF:latest",
})

mcpTools, err := getMcpTools()

agent, err := react.NewAgent(ctx, &react.AgentConfig{
    ToolCallingModel: chatModel,
    ToolsConfig: compose.ToolsNodeConfig{
        Tools: mcpTools,
    },
})

result, err := agent.Generate(ctx, []*schema.Message{
    schema.SystemMessage("请根据提供的天气查询工具回答问题"),
    schema.UserMessage("查询北京今天的天气"),
})
```

## 记忆点

- MCP 把工具服务从当前应用中解耦出去，便于跨语言、跨进程复用。
- MCP Server 负责定义和执行工具。
- MCP Client 负责发现工具并调用服务端。
- `mcpTool.GetTools` 是 MCP 工具接入 Eino 工具系统的关键桥梁。
- 接入 Agent 后，MCP Tool 和普通 Eino Tool 的使用方式基本一致。
