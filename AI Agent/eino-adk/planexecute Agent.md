## 预构建的 Multi-Agent 范式
### Plan-Execute
- Plan-Execute 模式：结构化问题解决
### 
> Plan-Execute Agent 是 ADK 提供的基于「规划-执行-反思」范式的 Multi-Agent 协作模式（参考论文 Plan-and-Solve Prompting），旨在解决复杂任务的分步拆解、执行与动态调整问题，通过 Planner（规划器）、Executor（执行器）和 Replanner（重规划器） 三个核心智能体的协同工作，实现任务的结构化规划、工具调用执行、进度评估与动态重规划，最终达成用户目标，其中：
###  流程：
- Planner：根据用户目标，生成一个包含详细步骤且结构化的初始任务计划
- Executor：执行当前计划中的首个步骤
- Replanner：评估执行进度，决定是修正计划继续交由 Executor 运行，或是结束任务

#### Plan-Execute 模式有如下特点：
- 明确的分层架构：通过将任务拆解为规划、执行和反思重规划三个阶段，形成层次分明的认知流程，体现了 “先思考再行动，再根据反馈调整” 的闭环认知策略，在各类场景中都能达到较好的效果。
- 动态迭代优化：Replanner 根据执行结果和当前进度，实时判断任务是否完成或需调整计划，支持动态重规划。该机制有效解决了传统单次规划难以应对环境变化和任务不确定性的瓶颈，提升了系统的鲁棒性和灵活性。
- 职责分明且松耦合：Plan-Execute 模式由多个智能体协同工作，支持独立开发、测试和替换。模块化设计方便扩展和维护，符合工程最佳实践。
- 具备良好扩展性：不依赖特定的语言模型、工具或 Agent，方便集成多样化外部资源，满足不同应用场景需求。

####  示例场景：
> Plan-Execute 模式的「规划 → 执行 → 重规划」闭环结构非常适合需要多步骤推理、动态调整和工具集成的复杂任务场景
###### 例如：
- 复杂研究分析：通过规划分解研究问题，执行多轮数据检索与计算，动态调整研究方向和假设，提升分析深度和准确性。
- 自动化工作流管理：将复杂业务流程拆解为结构化步骤，结合多种工具（如数据库查询、API 调用、计算引擎）逐步执行，并根据执行结果动态优化流程。
- 多步骤问题解决：适用于需要分步推理和多工具协作的场景，如法律咨询、技术诊断、策略制定等，确保每一步执行都有反馈和调整。
- 智能助理任务执行：支持智能助理根据用户目标规划任务步骤，调用外部工具完成具体操作，并根据重规划思考结合用户反馈调整后续计划，提升任务完成的完整性和准确性。
```golang
import github.com/cloudwego/eino/adk/prebuilt/planexecute

// Plan-Execute 模式的科研助手
researchAssistant := planexecute.New(ctx, &planexecute.Config{
    Planner: adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Name: "research_planner",
        Instruction: "制定详细的研究计划，包括文献调研、数据收集、分析方法等",
        Model: gpt4Model,
    }),
    Executor: adk.NewChatModelAgent(ctx, &adk.ChatModelAgentConfig{
        Name: "research_executor",
        ToolsConfig: adk.ToolsConfig{
            Tools: []tool.BaseTool{
                scholarSearchTool,
                dataAnalysisTool,
                citationTool,
            },
        },
    }),
    Replanner: replannerAgent,
})
```