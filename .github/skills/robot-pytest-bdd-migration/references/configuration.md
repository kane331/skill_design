# 配置说明

用户在自己的工作区提供迁移配置 JSON。所有相对路径以配置文件所在目录为基准。最小结构如下：

```json
{
	"legacy_robot_root": "/path/to/legacy-robot",
	"target_pytest_bdd_root": "/path/to/new-pytest-bdd",
	"test_case_excel": "/path/to/test-case.xlsx",
	"test_data_excel": "/path/to/test-data.xlsx",
	"artifacts_root": "/path/to/migration-artifacts",
	"max_attempts": 3
}
```

- `legacy_robot_root`：旧 Robot Framework 项目根目录，只读。
- `target_pytest_bdd_root`：新 pytest-bdd 项目根目录；只有实施阶段可写。
- `test_case_excel`：旧项目参数配置 Excel；其中的执行 flag Sheet 完全不进入选择契约、迁移数据或控制条件。
- `test_data_excel`：多 Sheet、多行实际测试数据 Excel。
- `artifacts_root`：脱敏工件根目录。
- `max_attempts`：每类循环的上限，只能是 1 到 3。
- `sensitive_field_patterns`：额外需要脱敏的字段名正则表达式。

总控在 `artifacts_root/project-baseline/` 自动维护以下可复用档案，无需额外配置字段：

- `project-analysis.json`：供 agent 读取的结构化项目级证据索引。
- `project-analysis.md`：由 JSON 派生的脱敏人工审阅文档。

基线档案不是当前 case 的迁移结论。每次执行都必须核验其格式版本、规范化后的配置路径，以及所有已记录来源的 SHA-256 指纹；任一项不匹配、文件不可访问、档案不可解析或关键证据缺失时，必须视为无效并重建。

总控还会在 `artifacts_root/cases/<case_id>/<run_id>/case-evidence.json` 归档当前执行的脱敏 case 证据包。它仅在本次运行的后续阶段使用，不可作为后续运行的缓存或跳过 case 级分析的依据。

配置不得包含凭据。总控 agent 必须把配置值视为运行参数，而不是业务逻辑或证据。