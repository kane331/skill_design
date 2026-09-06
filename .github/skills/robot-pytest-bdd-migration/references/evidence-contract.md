# 证据规范

每一个 agent 输出都必须返回一个由 `<<<MIGRATION_OUTCOME_JSON>>>` 与 `<<<END_MIGRATION_OUTCOME_JSON>>>` 包围的 JSON 对象。对象至少包含：

```json
{
  "case_id": "CASE-001",
  "status": "approved",
  "summary": "中文、无敏感原值的结论摘要。",
  "evidence": [
    {
      "id": "E-MAP-001",
      "source_type": "source",
      "path": "/绝对或相对路径",
      "location": "函数名、行号或 Sheet!单元格",
      "summary": "可复核事实，不包含敏感原值。",
      "source_digest": "来源文件摘要"
    }
  ],
  "findings": [],
  "data": {}
}
```

可用证据 ID 前缀：`E-MAP`、`E-DATA`、`E-ROBOT`、`E-LOCATOR`、`E-ASSERT`、`E-TARGET`、`E-RUN`。每一个迁移方案项、代码变更、审查发现和云端结论必须引用一个或多个证据 ID。

以下不是证据：文件名猜测、目录名称、相似字符串、模型常识、未运行的命令假设、推断出的页面结构或未验证的 Excel 语义。

敏感值仅可写为来源位置及脱敏摘要。不得复制单元格原值、账号、密码、Token、请求头或手机号。