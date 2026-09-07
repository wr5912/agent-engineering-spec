# 03｜Harness 开发与变更规范

## 1. Harness 的定位

Harness 是把模型能力组织成稳定任务能力的工程实现层。

本规范中的 Harness 可包括：

- Prompt / System Instructions；
- Context / Memory；
- Skills；
- Tools / MCP；
- Hooks；
- Agent / Subagent；
- Workflow；
- Plugins；
- Model Routing；
- Runtime 配置；
- 结构化输出与校验；
- 重试、超时、恢复机制。

## 2. 不把 Harness 配置作为主要业务验收对象

原则：

> **只要能力、安全和非功能指标不退化，实现方式允许变化。**

因此，不要求验收方逐项确认：

- Prompt 每一句话；
- Skill 内部实现；
- Agent 到底拆成几个；
- 使用哪种等价编排；
- Claude Code / DSH / AgentScope 的具体目录结构。

但正式发布必须记录运行基线。

## 3. Harness 设计优先级

按问题驱动，而不是“框架有什么功能就用什么”。

| 失败类型 | 优先修改对象 |
|---|---|
| 不理解任务 | Prompt / Task Definition |
| 缺知识 | Context / RAG / Knowledge |
| 不会执行 | Tool / Skill |
| 输出格式不稳定 | Schema / Validator |
| 流程步骤遗漏 | Workflow / State |
| 工具选择错误 | Tool Description / Routing |
| 长任务丢状态 | Checkpoint / Structured State |
| 权限越界 | Policy / Permission / Sandbox |

## 4. 最小变更原则

每轮优化尽量只改变 1～2 个主要变量。

不推荐：

> 同时更换模型、重写 Prompt、增加 3 个 Tool、改 Workflow，再看总分是否提高。

推荐：

> 先定位失败属于“工具选择错误”，只改 Tool 描述和选择规则，再运行受影响用例做针对性检查。

## 5. Harness 变更分类

开发阶段可以先运行受影响的用例子集，称为“针对性检查”；该结果仅用于快速反馈，不能替代正式回归或发布验收。任何变更进入发布候选前，均须按《04_评估测试与回归规范》运行完整 Eval Set。

### A 类：普通优化变更

例如：

- Prompt 表达优化；
- Context 压缩；
- Skill 内部逻辑调整；
- Agent 分工调整；
- 同权限范围内 Tool 选择策略变化。

要求：完成与变更相关的针对性检查。

### B 类：能力边界变更

例如：

- 新增 Tool；
- 新增业务系统；
- 新增自动动作；
- 改变关键输出逻辑。

要求：补充场景、测试，并重新确认风险等级。

### C 类：控制边界变更

例如：

- 扩大数据范围；
- 增加写权限；
- 允许 Shell；
- 取消人工审批；
- 修改沙箱或网络边界。

要求：必须执行安全控制评审和对应测试，不得仅凭能力测试通过发布。

## 6. 框架迁移规范

Claude Code → AgentScope → DSH 等迁移时，目标不是保持配置一一对应，而是保持：

1. 需求契约不丢；
2. 场景与 User Input 资产不丢；
3. 统一 Eval Set 不丢；
4. 安全边界不退化；
5. 关键能力不退化；
6. 新 Runtime 的基线可复现。

迁移验收以同一评估体系为主，而不是比较两个框架的配置文件是否长得一样。

## 7. 反对过度设计

以下方案默认不应作为 MVP 前置条件：

- 完整多 Agent 调度中心；
- 通用记忆平台；
- 自研 Prompt 管理平台；
- 全量可观测平台；
- 自动自进化闭环；
- 大规模 Agent 注册与治理中心。

只有在真实研发瓶颈已经出现、且能说明收益时再建设。
