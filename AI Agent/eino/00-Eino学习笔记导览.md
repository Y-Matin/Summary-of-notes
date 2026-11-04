# Eino 学习笔记导览

资料来源：[码神之路 Eino 教程](https://www.mszlu.com/docs/go/eino/02/02.html#_1-%E6%A8%A1%E7%89%88%E5%8C%96%E6%96%B9%E5%BC%8F)，并按该系列后续章节顺序整理。

配套图片存放在 [../../images/eino](../../images/eino) 下，笔记中使用相对路径引用，避免依赖远程图片地址。

## 学习顺序

1. [01-第一个AI应用.md](./01-第一个AI应用.md)
2. [02-ChatTemplate.md](./02-ChatTemplate.md)
3. [03-工具调用-ToolsNode.md](./03-工具调用-ToolsNode.md)
4. [04-知识库-RAG.md](./04-知识库-RAG.md)
5. [05-Agent-ReAct与MultiAgent.md](./05-Agent-ReAct与MultiAgent.md)
6. [06-MCP.md](./06-MCP.md)
7. [07-编排-Graph-Chain-Workflow.md](./07-编排-Graph-Chain-Workflow.md)
8. [08-A2A协议.md](./08-A2A协议.md)

## 主线理解

Eino 是 Go 生态下构建 AI 应用的框架。学习时可以按下面这条能力线推进：

```text
ChatModel 基础对话
  -> ChatTemplate 管理提示词
  -> ToolsNode 扩展模型能力
  -> RAG 知识库接入私有数据
  -> Agent 自动推理和行动
  -> MCP 接入外部工具协议
  -> Graph / Chain / Workflow 做业务编排
  -> A2A 实现 Agent 之间通信
```

这套教程的重点不是单独学习某个 API，而是逐步理解 AI 应用从“调用模型”到“能使用工具、能接入知识库、能自动规划、能和其他 Agent 协作”的完整演进过程。

## 核心组件速查

| 组件 | 作用 | 典型输入 | 典型输出 |
| --- | --- | --- | --- |
| ChatModel | 调用大模型生成回复 | `[]*schema.Message` | `*schema.Message` |
| ChatTemplate | 动态生成提示词消息 | 变量 `map[string]any` | `[]*schema.Message` |
| Tool / ToolsNode | 让模型调用外部程序能力 | ToolCall 消息 | Tool 结果消息 |
| Loader | 加载本地文件或 URL 文档 | `document.Source` | `[]*schema.Document` |
| Transformer | 切分和处理文档 | 文档列表 | 切分后的文档 |
| Embedder | 文本向量化 | 字符串列表 | 向量列表 |
| Indexer | 存储文档和向量索引 | 文档列表 | 文档 ID |
| Retriever | 根据 query 检索相关文档 | 用户问题 | 相关文档 |
| Agent | 组合模型和工具自动完成任务 | 用户消息 | 最终回答 |
| Graph / Chain / Workflow | 将模型、工具、检索等节点编排起来 | 业务输入 | 业务输出 |

## 复习建议

- 先把 `ChatModel -> ChatTemplate -> ToolsNode` 跑通，这是后面 Agent 和编排的基础。
- RAG 重点记住两个阶段：知识准备阶段和检索增强阶段。
- Agent 重点理解 ReAct 循环：思考、行动、观察、再思考。
- 编排部分重点区分：Graph 适合复杂拓扑，Chain 适合线性流程，Workflow 适合字段映射清晰的业务流。
