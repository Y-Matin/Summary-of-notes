# Golang AI Agent 系统化学习路线文档

## 文档信息

| 项目 | 内容 |
| --- | --- |
| 整理日期 | 2026-05-11 |
| 适配人群 | 已熟练掌握 Golang 开发，无需补 Go 基础 |
| 适用目标 | AI Agent 企业级开发、项目实战、BOSS 直聘求职上岗 |
| 核心主线 | 字节 Eino 框架（国内企业 / 招聘生产环境主流） |
| 学习周期 | 4 周速成，每天 1.5~3 小时 |
| 定位 | 全程免费资源、实战驱动、可直接写简历、对标招聘 JD |

## 一、核心学习原则

- 无需重复补习 Go 语法、并发、Web 基础，直接切入 AI Agent 开发。
- 以 Eino 框架作为唯一主线学深学透，后期再横向对比其他框架。
- 每周必须产出可运行代码 / 完整项目，拒绝只看视频不敲代码。
- 全程只用官方文档 + B 站免费优质教程，不买任何付费课程。
- 所学技能、做的项目完全匹配 BOSS 直聘 Golang AI Agent 岗位要求。

## 二、开发环境依赖

- Go 版本要求：`1.21` 及以上
- Eino 安装命令：

```bash
go get github.com/cloudwego/eino
```

- 推荐本地大模型：Ollama（离线免费、无需充值 API、本地开发首选）

## 三、4 周精准学习路线

### 第 1 周：Eino 入门 + 最简 Agent 跑通

#### 学习目标

建立 AI Agent 思维，熟悉 Eino 整体架构，分清 Agent / Graph / Tool / Workflow 概念，手写第一个命令行对话智能体。

#### 本周必看教程和资料

- [Eino 入门主课](https://www.bilibili.com/video/BV1uoY3zSE75/)
- [Eino 官网首页](https://www.cloudwego.io/zh/docs/eino/)
- [快速开始入门](https://www.cloudwego.io/zh/docs/eino/quick_start/)
- [Agent 与 Graph 工作流取舍](https://www.cloudwego.io/zh/docs/eino/overview/graph_or_agent/)
- [掘金入门好文](https://juejin.cn/post/7577239264308822016)

#### 每日学习安排

- 观看 B 站 Eino 入门视频，建立整体认知。
- 浏览 Eino 官方首页 + 快速开始文档。
- 跑通官方 ChatModel 基础对话示例。
- 跑通 ChatModelAgent + Runner 完整执行流程。
- 研读《Agent 还是 Graph》文档，理解架构设计取舍。
- 手写命令行流式对话 Agent（全程自己编码，不复制粘贴）。
- 复盘整理 Eino 核心组件关系笔记。

#### 本周产出

- 自研 CLI 流式聊天 Agent 可运行源码。
- 能清晰区分 Agent、Workflow、Graph、Tool 的适用业务场景。

### 第 2 周：精通 Tool Calling 工具调用

#### 学习目标

掌握 Eino 自定义工具开发、大模型自动函数调用、参数结构体绑定、参数校验、异常捕获、调用重试。

#### 本周必看教程和资料

- [字节 Eino 框架精讲](https://www.bilibili.com/video/BV1uV1yBHEAc/)
- [AI Agent 核心设计模式](https://www.cloudwego.io/zh/docs/eino/overview/eino_adk0_1/)
- [码神之路 Eino 教程](https://www.mszlu.com/docs/go/eino/05/05.html)

#### 学习重点

- ToolsNode 底层运行原理。
- 自定义 Tool 标准接口规范。
- LLM 自动判断何时触发工具调用。
- 工具入参结构体映射、自动解析。
- 工具调用失败处理、超时、重试机制。

#### 实战必做 3 个自定义工具

- 计算器工具（加减乘除复杂表达式）。
- 本地文件读取 / 解析 TXT/MD 工具。
- HTTP 接口查询工具（天气、公共开放 API）。

#### 本周产出

- 支持全自动工具调用的完整 Go Agent。
- 熟练掌握 Go 结构体绑定大模型工具入参开发模式。

### 第 3 周：进阶能力 + 简历级完整项目实战

#### 学习目标

掌握多轮对话记忆、会话上下文持久化、Graph 固定工作流编排、DeepAgents 复杂任务拆解，完成可写简历的企业级项目。

#### 本周必看教程和资料

- [Agent 进阶编排 / MoE 多专家](https://www.bilibili.com/video/BV1PrbszYEK4/)
- [Excel Agent 官方实战](https://www.cloudwego.io/zh/docs/eino/overview/eino_adk_excel_agent/)
- [Agent 与 Graph 工作流取舍](https://www.cloudwego.io/zh/docs/eino/overview/graph_or_agent/)
- [AI Agent 核心设计模式](https://www.cloudwego.io/zh/docs/eino/overview/eino_adk0_1/)

#### 学习内容

- Memory 会话记忆、多轮上下文管理。
- Graph 结构化工作流编排。
- DeepAgents 多步骤复杂任务执行。
- 跟着官方文档实战 Excel / CSV 数据分析 Agent。

#### 本周产出（可直接写进简历）

**Excel 智能数据分析 Agent**

功能：文件上传 -> 自动解析表格 -> 智能统计分析 -> 自然语言问答输出。

### 第 4 周：工程化落地 + 多框架对标

#### 学习目标

把 Demo 改成生产级可上线服务，掌握 Gin 接口、日志链路追踪、权限安全、Docker 部署，横向了解 Go 主流 Agent 框架。

#### 本周必看教程和资料

- [字节 Eino 框架精讲](https://www.bilibili.com/video/BV1uV1yBHEAc/)
- [Eino 官网首页](https://www.cloudwego.io/zh/docs/eino/)
- [Agent 与 Graph 工作流取舍](https://www.cloudwego.io/zh/docs/eino/overview/graph_or_agent/)
- 横向了解：LangChainGo、Google ADK for Go 官方文档。

#### 学习内容

- 基于 Gin 把 Agent 封装成 HTTP 接口服务。
- 接入 Eino Callback 事件、完整日志 & 链路追踪。
- 工具访问权限管控、文件目录安全限制。
- 编写 Dockerfile 实现容器化一键部署。
- 横向学习：LangChainGo、Google ADK for Go。
- 整理三大 Go Agent 框架优缺点、适用场景笔记。

#### 本周产出

- 可线上部署的 HTTP 接口版企业级 Agent 服务。
- 完整 Dockerfile 部署脚本。
- Go 主流 Agent 框架对比笔记（面试加分项）。

## 四、必做 5 个简历级实战项目

按难度递进：

1. 命令行流式聊天 Agent。
2. 多工具调用综合 Agent（计算器 + 文件解析 + HTTP 接口查询）。
3. 带持久化记忆的多轮会话 Agent。
4. Excel / CSV 智能数据分析 Agent。
5. 企业级 HTTP 接口版、可 Docker 部署 Agent 服务。

## 五、BOSS 直聘简历 & 投递关键词

```text
Golang、Eino 框架、AI Agent、智能体开发、Tool Calling、函数调用、LLM 大模型接入、RAG、Graph 工作流编排、多轮对话记忆、会话管理、Gin、Docker、云原生 AI 应用
```

## 六、学习避坑小贴士

- 优先啃官方文档，B 站视频做辅助，官方示例必须亲手敲一遍。
- 全程用 Ollama 本地模型开发，零成本、不用充值任何 API。
- 绝不复制粘贴示例代码，手动敲代码才能吃透底层逻辑。
- 4 周完整学完，即可直接投递 BOSS 直聘 Golang AI Agent 相关岗位。
