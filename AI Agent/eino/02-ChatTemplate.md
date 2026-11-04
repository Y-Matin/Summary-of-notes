# ChatTemplate

## 目标

`ChatTemplate` 用来模板化管理提示词。它把固定的提示词结构和运行时变量分离，适合多场景复用。

如果直接把提示词写成字符串，前期很快，但后面会遇到几个问题：角色、语气、任务、用户输入、历史消息都混在一起；同一个提示词稍微换个场景就要复制一份；引入对话历史或 RAG 上下文后，字符串拼接会变得很脆弱。

`ChatTemplate` 解决的是提示词工程的结构化问题：固定部分写在模板里，变化部分由变量传入，最终仍然生成标准的 `[]*schema.Message`，继续交给 `ChatModel`。

## 核心概念

### ChatTemplate 的作用

在真实项目中，提示词通常不是固定字符串，而是由角色、任务、用户输入、上下文、历史消息等动态拼出来。

`prompt.FromMessages(...)` 可以把多条模板消息组合成一个模板对象，再通过 `Format(ctx, vars)` 渲染为 `[]*schema.Message`。

```text
模板消息 + 变量
  -> Format
  -> 标准 messages
  -> ChatModel
```

## 三种模板格式

模板格式的选择原则：

- 只是替换变量，用 `FString`。
- 需要条件、循环、复杂分支，用 `GoTemplate`。
- 团队已有 Jinja2 模板经验或资产，用 `Jinja2`。

### FString

使用 `{variable}` 做变量替换，语法最简单。

适合：只需要变量替换，没有复杂逻辑的提示词。

```go
tpl := prompt.FromMessages(
    schema.FString,
    schema.SystemMessage("你是一个{role}，请用{tone}的语气回答问题"),
    schema.UserMessage("{question}"),
)

messages, err := tpl.Format(ctx, map[string]any{
    "role":     "技术专家",
    "tone":     "专业严谨",
    "question": "如何优化数据库性能？",
})
```

### GoTemplate

使用 Go 标准库 `text/template` 风格，支持条件判断、循环等逻辑。

适合：需要根据变量选择不同提示词分支的场景。

```go
tpl := prompt.FromMessages(
    schema.GoTemplate,
    schema.SystemMessage(`
{{if .isExpert}}你是一个专家级{{.domain}}顾问。{{else}}你是一个初级{{.domain}}助手。{{end}}
{{if .isFormal}}请使用正式语言。{{else}}请使用友好语言。{{end}}
你的任务是{{.task}}。
`),
    schema.UserMessage("{{.question}}"),
)

messages, err := tpl.Format(ctx, map[string]any{
    "isExpert": false,
    "domain":   "编程",
    "isFormal": false,
    "task":     "帮助初学者理解编程概念",
    "question": "什么是变量？",
})
```

### Jinja2

使用 Jinja2 风格语法。

适合：团队已有 Jinja2 提示词资产，或希望使用更接近 Python 生态的模板语法。

```go
tpl := prompt.FromMessages(
    schema.Jinja2,
    schema.SystemMessage(`
{% if level == 'expert' %}你是专家级顾问。{% else %}你是初级助手。{% endif %}
{% if domain %}你专长于{{ domain }}领域。{% endif %}
请用{% if formal %}正式{% else %}友好{% endif %}的语气回答。
`),
    schema.UserMessage("{{ question }}"),
)
```

## 对话历史

`schema.MessagesPlaceholder(name, optional)` 用于把历史消息插入模板。

```go
tpl := prompt.FromMessages(
    schema.GoTemplate,
    schema.SystemMessage("你是一个{{.role}}，请参考历史回答当前问题。"),
    schema.MessagesPlaceholder("history", false),
    schema.UserMessage("{{.question}}"),
)

messages, err := tpl.Format(ctx, map[string]any{
    "role": "记忆助手",
    "history": []*schema.Message{
        schema.UserMessage("我想了解 Go 并发机制"),
        schema.AssistantMessage("Go 使用 goroutine 和 channel 支持并发。", nil),
    },
    "question": "Go 的并发机制是什么？",
})
```

`optional` 参数含义：

- `false`：变量中必须提供该历史消息。
- `true`：没有历史消息也可以正常格式化。

## 典型流程

```text
定义 ChatTemplate
  -> 准备 vars
  -> template.Format(ctx, vars)
  -> model.Generate / model.Stream
```

## 记忆点

- FString 简单，GoTemplate 灵活，Jinja2 适合跨语言模板资产。
- `Format` 的结果是标准 `[]*schema.Message`，可以直接喂给 `ChatModel`。
- 历史消息不要手动拼字符串，优先用 `MessagesPlaceholder`。
- 系统提示词、历史、用户问题应保持清晰分层，方便后续接入 Agent 或 Graph。
