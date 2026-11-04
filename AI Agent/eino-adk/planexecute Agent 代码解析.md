## Plan-Execute

### Planer
### 构造方式
- NewPlanner
```
 planexecute.NewPlanner(ctx ,&planexecute.PlannerConfig{
		ChatModelWithFormattedOutput: chatModel,
		ToolInfo:                     nil,
		GenInputFn: func(ctx context.Context, userInput []adk.Message) ([]adk.Message, error) {
			
		},
		NewPlan: types.NewPlan(),
	})
```
- planexecute.PlannerConfig{} 配置详解
```
type PlannerConfig struct {
    //  指定了返回内容格式的 chatModel。 内容格式为： 创建chatModel时给 openai.ChatModelConfig.ResponseFormat  声明了返回结构体的结构。
    ChatModelWithFormattedOutput model.BaseChatModel   
    //  普通chatModel，没有指定返回的内容格式， 所以需要结合 下方的ToolInfo字段 ParamsOneOf(返回结构体结构) 来达到指定返回的效果。 
    ToolCallingChatModel         model.ToolCallingChatModel  // 
    // 和上方的ToolCallingChatModel 结合使用 
    ToolInfo                     *schema.ToolInfo
    // 负责设置调整 得到最终 提示词，给到 planner 大模型
    GenInputFn                   GenPlannerModelInputFn
    // 响应对象的 NewXX 函数。 响应对象需要实现 planexecute.Plan 接口 
    NewPlan                      NewPlan
}
```
  - 所以指定大模型输出内容格式，有两种方式 
    1.  ChatModelWithFormattedOutput
    2.  ToolCallingChatModel+ToolInfo
  - NewPlan 则是具体的结构体实例， 将模型的输出序列化为对象 。
    - 和上面是上下游关系
    - 上游负责指定大模型的输出格式（json内容）
    - 下游负责将输出的内容序列化为指定对象。

### Executer


### Replanner