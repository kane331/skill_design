---
name: robot-pytest-bdd-migration
description: "用于将 Robot Framework 移动端自动化 case 按单个 case ID 迁移到 pytest-bdd。适用于 Appium、Excel 测试数据、Robot 用例分析、pytest-bdd 代码生成、迁移审查和云端执行验证。总控协调中文子 agent，所有结论必须有旧项目、新项目、Excel 或云端报告证据，禁止猜测。"
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
- `test case` Excel 的执行 flag 不参与迁移范围、pytest skip、云端执行或成功统计。
- `test data` Excel 的多 Sheet、多行语义必须由旧项目读取代码证明。
- 只有云端结果证明目标 pytest-bdd 测试及全部已证明数据集通过时，最终状态才是 `passed`。
- 不输出密码、Token、密钥、手机号或其他敏感原值。

## 调用方式

使用 `robot-migration-orchestrator` agent，并在请求中提供单个 case ID 与迁移配置 JSON 的路径。配置仅提供路径、云端命令和必要运行上下文；子 agent 必须自行读取并验证配置指向的实际项目和 Excel，不能把配置文字当作事实。

总控按以下顺序协调：项目分析、映射门禁、旧用例追踪、流程审计、迁移方案、实施、静态验证、代码审查、云端验证。审计、静态修复、审查和云端代码归因修复最多各三轮；云端不可用时保留为“等待云端验证”。

配置字段、阶段产物和状态见 [配置说明](./references/configuration.md)、[证据规范](./references/evidence-contract.md) 与 [阶段契约](./references/stage-contract.md)。