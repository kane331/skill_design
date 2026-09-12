# 阶段契约

总控 agent 的状态顺序为：`preflight`、`baseline-validation`、`project-analysis`（仅在基线需重建时）或 `case-analysis`（基线有效时）、`mapping-gate`、`legacy-trace`、`trace-audit`、`case-evidence`、`migration-plan`、`implementation`、`static-validation` 与 `code-review`（并行）、`result-aggregation`、`passed`。

## 项目基线和分析

基线位于 `artifacts_root/project-baseline/project-analysis.json`。其最小结构如下：

```json
{
  "schema_version": 1,
  "normalized_paths": {
    "legacy_robot_root": "/path/to/legacy-robot",
    "target_pytest_bdd_root": "/path/to/new-pytest-bdd",
    "test_case_excel": "/path/to/test-case.xlsx",
    "test_data_excel": "/path/to/test-data.xlsx"
  },
  "source_fingerprints": [
    {"path": "/path/to/source", "sha256": "..."}
  ],
  "project_evidence": [],
  "project_patterns": {
    "robot_mapping_reader": [],
    "excel_loading": [],
    "pytest_bdd_extensions": [],
    "reusable_target_symbols": [],
    "target_call_relationships": [],
    "validation_commands": []
  }
}
```

`reusable_target_symbols` 必须记录可复用的 Feature、Step Definition、Page Object、Fixture 或 helper 的文件、符号、行为摘要、参数/返回值约束和证据 ID。`target_call_relationships` 仅记录可由源码直接证实的调用方、被调用符号、导出边界、继承或覆写关系及证据 ID。`source_fingerprints` 只能包含直接支撑 `project_evidence` 和 `project_patterns` 结论的最小常规文件集，不得作为项目文件清单。每个引用来源都必须出现在该集合中，且每个集合成员都必须支撑至少一项记录的结论。`project_evidence` 和 `project_patterns` 只能包含与项目通用结构、数据读取规则、目标项目扩展点和验证命令有关的证据。不得写入任何单独 case 的映射、行号、流程、定位器或断言。`project-analysis.md` 必须仅由该 JSON 派生，并使用脱敏摘要。

`baseline-validation` 输出 `missing`、`valid`、`stale` 或 `invalid`。只有路径、`schema_version` 和全部来源指纹均匹配时才是 `valid`；`missing`、`stale` 和 `invalid` 必须进入完整 `project-analysis` 并重建两个基线文件。`valid` 时进入 `case-analysis`，只复用已验证的项目级证据，仍须输出旧项目到新项目的当前 case 证据索引以及 `data.mapping`：

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

`resolution` 只能是 `unique`、`none`、`ambiguous` 或 `unresolved`。除 `unique` 且 `match_method` 为 `exact` 外，编排器会立即停止。即使基线有效，`row_numbers` 也只能列出由旧项目读取代码直接证明的当前 case 数据行，不能根据表面内容推测。`test case` 中的执行 flag Sheet 不属于选择契约，编排器不会读取它。

## 旧用例追踪与审计

追踪输出 `completed` 或 `approved`，并在 `data.workflow` 中列出前后置、完整 Keyword 调用树、每一步操作、最终定位器、等待/重试、断言、异常分支和外部依赖。审计只输出 `approved`、`needs-more` 或 `needs-human`，不会自行补写流程。审计与补充最多三轮。

## Case 证据包、迁移方案与实施

流程审计 `approved` 后，总控在 `artifacts_root/cases/<case_id>/<run_id>/case-evidence.json` 写入当前 run 的映射、`excel_selection`、追踪流程、审计结论和全部引用证据。该档案必须包含 `case_id`、`run_id`、来源证据 ID 和来源摘要；后续方案、实施、验证和审查阶段优先消费它，遇到缺失、冲突或需要确认 diff 影响时才回读来源。每次新运行都必须新建证据包，不能用它替代新运行的映射门禁或流程追踪。

迁移方案只有 `approved` 才能实施。`data.changes` 的每项必须含 `target_file`、`target_symbol`、`change_type`、`legacy_evidence_ids`、`target_evidence_ids`、`validation_source`、`reuse_decision`、`candidate_symbols` 和 `compatibility_impact`。

`reuse_decision` 只能是 `reuse`、`compose`、`new-case-specific`、`new-shared` 或 `modify-existing`。`candidate_symbols` 必须列出已检查的可复用目标符号、证据 ID，以及未选用时基于参数、前置条件、等待、断言或行为语义的排除理由；不得凭名称相似性排除或复用。只有不存在可证明满足需求的候选能力时，才可选择 `new-case-specific` 或 `new-shared`。

`compatibility_impact` 必须说明该变更是否修改已有符号。对 `modify-existing`，必须列出所有可由源码证明的调用方、导出/继承边界、旧行为、向后兼容策略和覆盖每个调用方的验证来源。无法证明调用关系，或存在动态调用而无法确定影响范围时，方案必须返回 `needs-human`。不得通过改变默认参数、返回结构、异常语义、等待时序或全局 Fixture 生命周期破坏已有调用方。

实施只能输出 `implemented`、`needs-human` 或 `blocked`。成功实施时还必须输出精确 `test_selector`、全部 `data_set_ids`、实际 `changed_files`、受影响调用方及其验证范围，以及方案/证据关联。

## 并行静态验证、审查和结果汇总

实施完成后，静态验证与代码审查必须针对同一份实施报告、当前 diff 和 case 证据包并行执行。静态验证状态为 `passed`、`needs-fix` 或 `needs-human`；只可执行项目分析已证明存在的命令。代码审查状态为 `approved`、`needs-fix` 或 `needs-human`。总控只在两者都完成后汇总结论；任一结果不是通过状态时不得进入 `passed`。

静态验证通过后，总控必须确认实施报告中的 `test_selector` 和全部 `data_set_ids` 与已证明的本地验证范围一致。若方案修改了已有调用方法，验证范围还必须覆盖每个已证明调用方的兼容性验证。静态验证命令失败、不可用，或无法证明其覆盖目标测试或受影响调用方时，不得进入 `passed`。

每个失败结果必须写入 `failure_fingerprint`，至少包括阶段、命令或审查规则、退出码（如适用）、失败位置、日志摘要和关联变更摘要。实施修复后，总控只可跳过已通过且经证据证明不受本次 diff 影响的命令或审查项；必须重新执行失败项及所有受影响项。若相同 `failure_fingerprint` 在相关代码未变更时再次出现，立即返回 `needs-human`，不得进行无效重试。