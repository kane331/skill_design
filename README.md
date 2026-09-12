# Robot 到 pytest-bdd 迁移 Skill

本指南用于运行 [Robot 到 pytest-bdd 用例迁移 Skill](.github/skills/robot-pytest-bdd-migration/SKILL.md)。每次任务只迁移一个由 `case_id` 唯一指定的 Robot Framework 移动端用例。

该 skill 使用可复核证据迁移代码：不会根据名称或经验猜测流程、数据、定位器或断言。完成条件是本地已证明的静态/定向验证和代码审查均通过；**不执行或等待云端验证**。

## 1. 启动位置与登录

在 `skill_design` 目录启动 GitHub Copilot CLI，使其能够发现 `.github/skills` 和 `.github/agents` 中的自定义内容。

确认已登录 Copilot CLI。首次访问旧项目、新项目或 Excel 所在的外部目录时，只授权本次迁移实际需要的明确路径；不要使用 `--allow-all` 或 `--yolo`。

## 2. 准备迁移配置

在你自己的工作区创建一份 JSON 配置。相对路径以该 JSON 文件所在目录为基准。

```json
{
  "legacy_robot_root": "/path/to/legacy-robot",
  "target_pytest_bdd_root": "/path/to/new-pytest-bdd",
  "test_case_excel": "/path/to/test-case.xlsx",
  "test_data_excel": "/path/to/test-data.xlsx",
  "artifacts_root": "/path/to/migration-artifacts",
  "max_attempts": 3,
  "sensitive_field_patterns": ["自定义敏感字段名"]
}
```

配置只提供路径和运行上下文，不是业务逻辑证据。不要将密码、Token、密钥、手机号或其他个人数据写入配置。

## 3. 检查路径与环境

- `legacy_robot_root`：旧 Robot Framework 项目，只读。
- `target_pytest_bdd_root`：新 pytest-bdd 项目；只有实施阶段可写。建议使用干净分支或独立 worktree。
- `test_case_excel`：旧项目的参数配置 Excel。其执行 flag Sheet 不参与迁移范围、pytest skip 或成功统计。
- `test_data_excel`：包含多 Sheet、多行参数化运行数据的 Excel。
- `artifacts_root`：可写的脱敏工件目录；建议加入实际项目的 `.gitignore`。
- `max_attempts`：审计、静态修复与审查循环的上限，只能是 1 到 3。
- `sensitive_field_patterns`：可选的额外敏感字段名正则表达式。

确认目标 pytest-bdd 项目中已存在可执行的本地收集、语法、格式、类型或定向测试命令。skill 只能运行项目分析阶段实际证明存在的命令，不能凭经验创建命令。

## 4. 调用总控 Agent

在 Copilot CLI 中选择 `robot-migration-orchestrator`，然后输入：

```text
case_id=CASE-001；配置=/你的路径/migration.json
```

也可以从命令行直接指定总控 agent：

```shell
copilot --agent robot-migration-orchestrator --prompt "case_id=CASE-001；配置=/你的路径/migration.json"
```

## 5. 首次运行：完整项目分析与基线归档

首次运行时，`artifacts_root/project-baseline/project-analysis.json` 不存在。总控会执行完整项目分析，确认：

- `case_id` 映射的读取方式；
- `test case` 和 `test data` Excel 的读取、Sheet、行及组合规则；
- 新 pytest-bdd 项目的 Feature、Step Definition、Page Object、Fixture、参数化和本地验证模式；
- 可复用的目标项目方法；
- 可由源码证明的调用方、导出边界、继承和覆写关系。

随后自动生成：

```text
artifacts_root/
  project-baseline/
    project-analysis.json
    project-analysis.md
```

- `project-analysis.json`：结构化项目级证据索引，供后续 agent 使用。
- `project-analysis.md`：由 JSON 派生的脱敏人工审阅文档。

基线会保存直接支撑项目级结论的最小来源文件集及 SHA-256 指纹，而不是扫描或记录整个项目。

## 6. 后续运行：复用项目基线

后续迁移同一项目的 case 时，总控会先验证基线：

- 档案可解析且 `schema_version` 兼容；
- 旧 Robot、新 pytest-bdd、两个 Excel 的规范化路径一致；
- 所有基线来源文件仍可访问；
- 记录的 SHA-256 指纹仍一致；
- 关键证据字段完整。

检查通过时，基线为 `valid`，总控复用项目级分析，仅对当前 case 做增量分析。任一检查失败时，基线为 `missing`、`stale` 或 `invalid`，总控会自动重新做完整项目分析并覆盖基线档案。

基线不能替代当前 case 的事实。每次运行仍必须重新确认：

- `case_id` 到 Robot suite/test 的精确唯一映射；
- 当前 case 的 Excel 数据行和数据集；
- Keyword 调用链、变量、定位器、等待、断言和异常路径。

## 7. 单条 case 的迁移流程

```text
配置与路径预检
  ↓
项目基线校验
  ↓
完整项目分析（基线无效时）或当前 case 增量分析（基线有效时）
  ↓
精确映射门禁
  ↓
Robot 流程追踪
  ↓
流程完整性审计
  ↓
生成本次运行的 case 证据包
  ↓
迁移方案
  ↓
pytest-bdd 实施
  ↓
静态/定向验证 ─┐
                ├─ 结果汇总 → passed
代码审查 ───────┘
```

### 映射门禁

只有当前 case 的结果同时为：

```json
{
  "resolution": "unique",
  "match_method": "exact"
}
```

才能继续。映射不存在、多义、动态且无法求值，或不是精确匹配时，任务立即停止，不会生成或修改迁移代码。

### 流程追踪与审计

追踪 agent 会展开 Setup/Teardown、全部 Keyword 调用、数据集、动作、最终定位器、等待/重试、断言、异常分支和外部依赖。审计 agent 独立检查是否遗漏；追踪和审计最多往返三轮。

审计通过后，总控会写入本次运行专用的证据包：

```text
artifacts_root/cases/<case_id>/<run_id>/case-evidence.json
```

后续方案、实施、验证和审查优先使用该证据包，避免重复读取同一批 Robot、Excel 和项目证据。它只对本次运行有效，不能跨运行复用，也不能跳过下一次运行的映射和追踪。

### 方案与实施：复用和兼容优先

方案 agent 必须优先评估：

```text
reuse → compose → new-case-specific → new-shared → modify-existing
```

已有方法能满足参数、前置条件、等待、断言和行为语义时必须复用；只有没有可证明可复用能力时才允许新增。

已被调用的方法默认不允许修改。确实需要修改时，方案必须说明全部可确认调用方、旧行为、向后兼容策略和调用方验证来源。无法确定动态调用影响范围时，任务会返回 `needs-human`。

新增或修改 BDD 描述时，会参考新项目已有 Feature 的语言、关键字和表述方式。每条描述只写一个简短的业务前提、动作或结果；不会将定位器、等待、函数名或其他实现细节写进 BDD 文案。

实施只能修改批准方案列出的目标文件和符号；不得擅自扩大范围、重复实现已有能力或破坏已有调用方的默认参数、返回、异常、等待时序和 Fixture 生命周期。

### 验证、审查与精准重试

实施完成后，静态/定向验证与代码审查并行执行：

- 验证只运行项目中已证明的本地命令；
- 有多个目标可由既有命令模式一次覆盖时，优先使用结果可区分的最小合并命令；
- 修改已有被调用方法时，除新 case 外，还必须验证每个受影响调用方；
- 审查验证流程保真、数据覆盖、复用决策、兼容性、diff 和回归风险。

每个失败或无法继续的结果都会带有脱敏 `failure_fingerprint`。修复后只重跑失败项和受本次 diff 影响的项目；只有能证明未受影响的已通过项才可复用结果。相同失败在相关代码未变化时再次出现，会停止并标记 `needs-human`，避免无效重试。

## 8. `passed` 的完成条件

只有同时满足以下条件，最终状态才是 `passed`：

1. 当前 `case_id` 已精确且唯一映射；
2. Robot 流程、数据、定位器、等待和断言均有直接证据；
3. 独立流程审计通过；
4. 迁移方案通过，且复用/新增/兼容决策完整；
5. 实施没有超出批准范围；
6. 已证明的本地静态或定向验证命令通过；
7. 修改已有方法时，所有已识别调用方均已覆盖验证；
8. 代码审查通过；
9. `test_selector`、`data_set_ids` 和验证范围与实施报告一致。

所有报告使用中文且不会输出敏感原值。完整配置字段见 [配置说明](.github/skills/robot-pytest-bdd-migration/references/configuration.md)，状态、工件和字段契约见 [阶段契约](.github/skills/robot-pytest-bdd-migration/references/stage-contract.md)，证据输出要求见 [证据规范](.github/skills/robot-pytest-bdd-migration/references/evidence-contract.md)。