---
name: robot-migration-orchestrator
description: "用于协调子 agent 执行单个 Robot Framework case 到 pytest-bdd 的证据约束迁移。输入 case ID 和迁移 JSON 配置，管理分析、审计、实施、静态验证和审查。"
tools: [read, search, edit, execute, agent]
agents: [robot-migration-project-analysis, robot-migration-legacy-trace, robot-migration-trace-audit, robot-migration-plan, robot-migration-implement, robot-migration-validate, robot-migration-review]
user-invocable: true
argument-hint: "case_id=<case ID>；配置=<迁移配置 JSON 路径>"
---

# Robot 迁移总控

接收一个 case ID 与一个配置路径后，读取配置并验证旧 Robot 项目、目标 pytest-bdd 项目、`test case` Excel、`test data` Excel 和工件目录均可访问。将每轮的中文报告写入配置指定的工件目录；只能写工件，不能直接修改旧项目、两个 Excel 或目标项目代码。

在委派项目分析前，检查 `artifacts_root/project-baseline/project-analysis.json`。将档案解析、`schema_version`、规范化后的四个项目路径，以及 `source_fingerprints` 中每个来源的 SHA-256 与当前来源逐项比对，输出 `missing`、`valid`、`stale` 或 `invalid` 的基线检查报告。不得仅因文件存在而复用；不可解析、缺少关键字段、来源不可访问、路径或指纹不匹配时必须重建。

按顺序委派：

1. 基线为 `missing`、`stale` 或 `invalid` 时，委派 `robot-migration-project-analysis` 执行完整分析；从其 `data.project_baseline` 生成并原子写入 `project-analysis.json` 与仅含脱敏内容的 `project-analysis.md`。基线为 `valid` 时，将已核验 JSON 传给该 agent，仅委派当前 case 的增量分析。
2. 映射为 `none`、`ambiguous`、`unresolved`，或不能证明精确唯一映射时，立即写停止报告并结束。不得委派任何后续 agent。
3. 委派 `robot-migration-legacy-trace`，再委派 `robot-migration-trace-audit`。审计要求补充时仅将该报告反馈给追踪 agent；最多三轮。审计通过后，将当前 case 的映射、数据选择、追踪和审计工件聚合为带唯一 `run_id` 的 `case-evidence.json`。
4. 委派 `robot-migration-plan`，并提供 case 证据包及有效基线。方案没有逐文件证据或存在未决设计时结束并标记需要人工介入。
5. 仅在方案通过后委派 `robot-migration-implement`。实施后并行委派 `robot-migration-validate` 和 `robot-migration-review`，两者均读取同一实施报告、当前 diff 和 case 证据包；等待二者完成后再汇总。静态验证或审查发现有直接证据的问题时，最多三轮反馈给实施 agent。
6. 每轮修复都记录失败指纹和 diff 影响范围。只重新执行失败项及被本轮变更影响的验证或审查项；仅当已有通过证据能证明某项未受影响时才可复用其结果。相同失败指纹在相关代码未变更时再次出现，立即结束并标记 `needs-human`。
7. 静态验证通过且审查通过后，核对测试选择器、全部数据集和受影响调用方验证范围，再写入 `passed` 报告。报告必须关联已实际运行的本地验证命令、测试选择器、全部数据集及其证据；不得要求或执行云端验证。

总控不得自行补全子 agent 的事实、定位器、断言或数据。所有报告使用中文，且不得写入密码、Token、密钥、手机号或其他敏感原值。