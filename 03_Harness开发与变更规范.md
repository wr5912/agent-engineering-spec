# 03｜Harness 开发与变更规范

## 1. Harness 的定位

Harness 是把模型能力组织成稳定任务能力的工程实现层，可包括：

- Prompt / System Instructions；
- Context / Memory / RAG；
- Skills；
- Tools / MCP；
- Hooks；
- Agent / Subagent；
- Workflow；
- Plugins；
- Model Routing；
- Runtime 配置；
- 结构化输出与校验；
- 重试、超时、恢复和状态管理。

本文件只规范实现、变更、针对性检查和候选 Baseline 准备。正式自测与正式回归按《04｜评估、测试与回归规范》执行，整体交付裁决按《06｜交付评估与发布规范》执行。

## 2. 实现责任与判断依据

开发成员对 Harness 实现、变更记录、针对性检查和候选 Baseline 的完整性负责。交付负责人依据需求、能力、安全和可复现证据判断交付完成度，不逐句审批 Prompt 或指定内部编排方式。

原则：

> **实现方式可以变化，但需求契约、安全边界、正式自测门槛和可追溯性不能被绕过。**

因此，不把以下内容本身作为能力好坏的判据：

- Prompt 写了多少行；
- Skill 如何拆分；
- Agent 数量；
- 等价 Workflow 的节点差异；
- Claude Code、DSH、AgentScope 等框架的目录形态。

这些实现一旦影响运行结果，仍必须进入版本控制和候选 Baseline。

## 3. 设计优先级

先根据真实失败定位问题，再选择最小改动：

| 失败类型 | 优先检查或修改 |
|---|---|
| 不理解任务 | 需求表达 / Prompt / Task Definition |
| 缺少知识 | Context / RAG / Knowledge |
| 不会执行 | Tool / Skill |
| 输出格式不稳定 | Schema / Validator |
| 流程步骤遗漏 | Workflow / State |
| 工具选择错误 | Tool Description / Routing |
| 长任务丢状态 | Checkpoint / Structured State |
| 权限越界 | Policy / Permission / Sandbox |

每轮优化原则上只改变 1～2 个主要变量。若必须同时修改多项，应在变更记录中说明原因，并承认结果无法准确归因到单一变量。

## 4. 最小实验与针对性检查

开发成员可先用探索实验验证任务是否可行，再用针对性检查缩短变更反馈时间。

针对性检查至少覆盖：

- 本次变更直接影响的 Case；
- 与变更共享 Tool、Prompt、Policy、状态或输出结构的相邻 Case；
- 曾在相同位置失败的 `regression` Case；
- 涉及权限或动作边界时，对应的允许、审批和阻断 Case。

针对性检查只回答“本次受影响范围是否出现明显问题”，不能替代完整 Eval Set 的正式自测或正式回归，也不能单独形成交付或发布结论。

## 5. Harness 变更分类

| 类别 | 典型变化 | 最小处理 |
|---|---|---|
| A：普通优化 | Prompt 表达、Context 压缩、Skill 内部逻辑、同权限 Tool 路由 | 记录变更并运行受影响 Case 的针对性检查 |
| B：能力边界 | 新增 Tool、业务系统、自动动作或关键输出逻辑 | 更新需求场景和 Eval Case，重新确认风险等级 |
| C：控制边界 | 扩大数据或写权限、允许 Shell、取消审批、修改沙箱或网络边界 | 更新安全清单和安全 Case，完成控制核验；不得只看能力结果 |

无法明确分类时按影响更高的一类处理。一个变更同时属于多类时，叠加执行各类要求。

## 6. 候选 Baseline

### 6.1 形成时机

开发成员在以下条件满足后形成候选 Baseline：

1. 本轮实现已进入版本控制；
2. 需求、成功标准和风险等级已确认；
3. 受影响范围的针对性检查已完成；
4. Eval Set、Rubric、安全控制和执行环境已准备完成；
5. 已知问题未触碰 `blocking` 门槛，且不会造成正式自测无法执行。

候选 Baseline 只是待正式自测的明确对象，不代表已经通过自测、交付评估或允许发布。

### 6.2 运行前冻结项

候选 Baseline 至少绑定：

- 需求版本；
- Agent/Harness 的 Git Commit 或 Tag；
- Runtime / Framework 及版本；
- 模型和关键推理配置；
- Prompt、Skills、Tool / MCP 版本；
- Policy 和安全控制版本；
- Eval Set 版本、位置和 SHA-256；
- Rubric、阈值和 Trial 聚合规则；
- 执行环境或可复现环境标识。

完整字段和交付记录方式见《06｜交付评估与发布规范》。不存在的组件不为填表而创建。

### 6.3 Baseline 与 Run 规则

- 任一冻结项变化，必须生成新的 `baseline_id`；
- Prompt、模型、Tool、Policy、Eval Set、Rubric、需求或环境变化，都不能沿用旧 Baseline；
- 同一 Baseline 在冻结项不变时重跑，只生成新的 `run_id`；
- 新运行结果不得覆盖旧运行，报告必须能定位采用的 `run_id`；
- 正式自测期间需要修改实现或冻结项时，当前运行停止形成结论，修改后建立新 Baseline 并重新执行完整 Eval Set。

结果文件及其哈希可以在运行完成后关联到 Baseline 记录；仅补充该次运行结果引用不视为冻结项变化。

## 7. 框架或 Runtime 迁移

Claude Code、AgentScope、DSH 等框架迁移不要求配置一一对应，但必须保证：

1. 需求与成功标准不丢；
2. 场景、User Input 和 Eval Case 资产不丢；
3. 安全边界不退化；
4. 关键能力可用同一套判定标准比较；
5. 新 Runtime 的 Baseline 可复现；
6. 迁移引入的新能力和风险已补入需求、安全清单和 Eval Set。

迁移本身属于候选实现变化，必须生成新 Baseline，并对完整 Eval Set 执行正式自测。不得以配置文件看起来相似代替结果证据。

## 8. 反对过度设计

以下能力不作为起步前置条件：

- 完整多 Agent 调度中心；
- 通用记忆平台；
- 自研 Prompt 管理平台；
- 全量可观测平台；
- 自动自进化闭环；
- 大规模 Agent 注册与治理中心。

先用版本控制、JSONL、CSV、脚本和必要运行证据完成最小闭环。只有真实研发瓶颈反复出现，并且能够说明收益、维护成本和替代方案时，才引入新平台。

## 9. 提交正式自测前检查

- 变更目标、范围和分类已记录；
- 受影响 Case 的针对性检查已完成；
- 需求、场景、安全边界和 Eval Set 已按影响更新；
- 候选 Baseline 已创建且冻结项可追溯；
- 不存在用局部检查替代完整正式自测的声明。
