# 配置说明

用户在自己的工作区提供迁移配置 JSON。所有相对路径以配置文件所在目录为基准。最小结构如下：

```json
{
	"legacy_robot_root": "/path/to/legacy-robot",
	"target_pytest_bdd_root": "/path/to/new-pytest-bdd",
	"test_case_excel": "/path/to/test-case.xlsx",
	"test_data_excel": "/path/to/test-data.xlsx",
	"artifacts_root": "/path/to/migration-artifacts",
	"cloud": {
		"trigger": {"executable": "company-cloud-cli", "args": ["trigger", "{case_id}", "{test_selector}"]},
		"poll": {"executable": "company-cloud-cli", "args": ["poll", "{case_id}"]}
	},
	"max_attempts": 3
}
```

- `legacy_robot_root`：旧 Robot Framework 项目根目录，只读。
- `target_pytest_bdd_root`：新 pytest-bdd 项目根目录；只有实施阶段可写。
- `test_case_excel`：旧项目参数配置 Excel；其中的执行 flag Sheet 完全不进入选择契约、迁移数据或控制条件。
- `test_data_excel`：多 Sheet、多行实际测试数据 Excel。
- `artifacts_root`：脱敏工件根目录。
- `cloud.trigger`、`cloud.poll`：必须分别由 `executable` 和 `args` 表达，禁止 shell 字符串。允许占位符：`{case_id}`、`{test_selector}`、`{run_dir}`、`{config_path}`。
- `max_attempts`：每类循环的上限，只能是 1 到 3。
- `sensitive_field_patterns`：额外需要脱敏的字段名正则表达式。

云端命令的标准输出必须是单个 JSON 对象。不要将凭据写入配置；应由企业已有的身份、密钥链或受控运行环境提供。总控 agent 必须把配置值视为运行参数，而不是业务逻辑或证据。