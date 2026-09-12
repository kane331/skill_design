---
name: robot-pytest-bdd-migration
description: "用于将 Robot Framework 移动端自动化 case 按单个 case ID 迁移到 pytest-bdd。适用于 Appium、Excel 测试数据、Robot 用例分析、pytest-bdd 代码生成、迁移审查和本地静态/定向验证。总控协调中文子 agent，所有结论必须有旧项目、新项目或 Excel 证据，禁止猜测。"
argument-hint: "case_id=<case ID>；配置=<迁移配置 JSON 路径>"
user-invocable: true
disable-model-invocation: false
---

# Robot 到 pytest-bdd 用例迁移

## 适用范围

仅迁移一个由 `case_id` 唯一指定的 Robot Framework 移动端用例到已有 pytest-bdd 项目。调用 `robot-migration-orchestrator` 总控 agent 后，由它协调各个子 agent；不要自行跳过阶段或直接生成迁移代码。

## 硬性规则

- 所有事实、定位器、步骤、断言、数据集、目标项目扩展点和测试命令必须有直接证据。
- 禁止猜测、模糊匹配、同名联想、补造业务逻辑、补造定位器或补造运行数据。
- `case_id` 未能通过旧项目映射代码唯一定位 Robot 用例时，立即停止；不得调用后续子 agent，也不得修改目标项目。
- `test case` Excel 的执行 flag 不参与迁移范围、pytest skip 或成功统计。
- `test data` Excel 的多 Sheet、多行语义必须由旧项目读取代码证明。
- 项目基线档案只能复用经指纹验证的项目级证据；不得替代当前 `case_id` 的精确映射、数据选择或流程追踪。
- 迁移必须优先复用有直接行为证据的目标项目能力；仅在不存在可复用能力时才可新增，并在方案中记录候选项与排除理由。
- 已被调用的目标项目方法默认不得修改；确需修改时，必须有已证明调用方、向后兼容方案和覆盖调用方的验证证据，否则停止并标记 `needs-human`。
- 新增或修改的 BDD 描述必须遵循目标项目已有 Feature 的语言、关键字和措辞风格；每个动作描述应简短且只表达一个业务动作，不得塞入定位器、等待或实现细节。
- 只有项目中已证明的本地静态或定向验证命令通过，且代码审查确认测试选择器与全部已证明数据集一致时，最终状态才是 `passed`。
- 不输出密码、Token、密钥、手机号或其他敏感原值。

## 调用方式

使用 `robot-migration-orchestrator` agent，并在请求中提供单个 case ID 与迁移配置 JSON 的路径。配置仅提供路径和必要运行上下文；子 agent 必须自行读取并验证配置指向的实际项目和 Excel，不能把配置文字当作事实。

总控先验证项目基线档案：首次、缺失、无效或过期时执行完整项目分析并归档；基线有效时仅执行当前 case 的增量分析。随后按顺序协调：映射门禁、旧用例追踪、流程审计、case 证据包归档、迁移方案、实施、静态验证与代码审查并行、结果汇总。审计、静态修复和审查最多各三轮；每次重试只重新执行受变更影响的验证与审查，并保留全部通过结果的可复核证据。

配置字段、阶段产物和状态见 [配置说明](./references/configuration.md)、[证据规范](./references/evidence-contract.md) 与 [阶段契约](./references/stage-contract.md)。