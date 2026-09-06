# 阶段契约

总控 agent 的状态顺序为：`preflight`、`project-analysis`、`mapping-gate`、`legacy-trace`、`trace-audit`、`migration-plan`、`implementation`、`static-validation`、`code-review`、`cloud-validation`、`passed`。

## 项目分析

输出旧项目到新项目的证据索引，以及 `data.mapping`：

```json
{
  "resolution": "unique",
  "match_method": "exact",
  "robot_suite_path": "...",
  "robot_test_name": "...",
  "evidence_ids": ["E-MAP-001"],
  "excel_selection": {
    "test_case": {
      "sheet": "参数配置",
      "header_row": 1,
      "case_id_header": "case_id",
      "fields": ["数据Sheet"],
      "case_id_match": "exact-string"
    },
    "test_data": [
      {
        "sheet": "登录数据",
        "header_row": 1,
        "row_numbers": [2, 3],
        "fields": ["用户名", "密码"]
      }
    ]
  }
}
```

`resolution` 只能是 `unique`、`none`、`ambiguous` 或 `unresolved`。除 `unique` 且 `match_method` 为 `exact` 外，编排器会立即停止。`row_numbers` 只能列出由旧项目读取代码直接证明的行，不能根据表面内容推测。`test case` 中的执行 flag Sheet 不属于选择契约，编排器不会读取它。

## 旧用例追踪与审计

追踪输出 `completed` 或 `approved`，并在 `data.workflow` 中列出前后置、完整 Keyword 调用树、每一步操作、最终定位器、等待/重试、断言、异常分支和外部依赖。审计只输出 `approved`、`needs-more` 或 `needs-human`，不会自行补写流程。审计与补充最多三轮。

## 迁移方案与实施

迁移方案只有 `approved` 才能实施。`data.changes` 的每项必须含 `target_file`、`target_symbol`、`change_type`、`legacy_evidence_ids`、`target_evidence_ids` 和 `validation_source`。

实施只能输出 `implemented`、`needs-human` 或 `blocked`。成功实施时还必须输出精确 `test_selector`、全部 `data_set_ids`、实际 `changed_files` 和方案/证据关联。

## 静态验证、审查和云端验证

静态验证状态为 `passed`、`needs-fix` 或 `needs-human`；只可执行项目分析已证明存在的命令。代码审查状态为 `approved`、`needs-fix` 或 `needs-human`。静态修复和审查均最多三轮。

云端命令必须输出 JSON，`passed` 必须有 `report`、`executed_test_ids` 和 `executed_data_set_ids`。结果为 `queued` 或 `unavailable` 时只能得到 `awaiting-cloud-validation`。只有云端实际执行的测试选择器和数据集集合与实施报告一致时可进入 `passed`。