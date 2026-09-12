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

任何 `needs-fix`、`needs-human` 或 `blocked` 结果都必须在相关 `findings` 中提供 `failure_fingerprint`，用于识别同一未解决问题；其内容必须是脱敏的，并包含阶段、失败位置、证据或日志摘要及关联变更摘要。验证失败还必须包含实际命令与退出码。

可用证据 ID 前缀：`E-MAP`、`E-DATA`、`E-ROBOT`、`E-LOCATOR`、`E-ASSERT`、`E-TARGET`、`E-RUN`。每一个迁移方案项、代码变更、审查发现和本地验证结论必须引用一个或多个证据 ID。

以下不是证据：文件名猜测、目录名称、相似字符串、模型常识、未运行的命令假设、推断出的页面结构或未验证的 Excel 语义。

项目基线档案中的每项可复用证据必须保留原始证据 ID、来源路径、位置、脱敏摘要和来源 SHA-256 指纹。总控只有在已核验全部指纹后，才可将其作为当前执行的项目级证据；不能核验时，必须重新从来源取得证据。基线不得保存或引用某个 `case_id` 的映射、数据行、流程步骤、定位器或断言。

流程审计通过后，总控必须将当前 case 的映射、数据选择、追踪流程、审计结论和引用证据写入一次性 `case-evidence.json`。后续 agent 应优先读取该证据包中的证据 ID、来源位置和脱敏摘要，仅在证据包缺失、冲突、过期或需核对具体实现影响时读取原始来源。证据包必须带本次 `run_id`，且不得跨运行复用。

敏感值仅可写为来源位置及脱敏摘要。不得复制单元格原值、账号、密码、Token、请求头或手机号。