# Workflow Reference

This file expands the source prompt into a reusable operating rulebook.

## 1. Inputs

- Required explicit inputs:
  - `姓名`
  - `工作职责`
  - `粗略日期范围`
  - `项目清单`
- Rough date range: use the user-provided broad window.
- Precise date range: compute from `references/workday_calendar.json`.
- Workday range: use the workday dates stored in `references/workday_calendar.json`.
- Project data: read from `项目清单` and attachments.
- Role facts: use `工作职责` as the active source of truth.

If the precise range is missing or the JSON cannot be read, use the rough range directly.

If any required explicit input is missing, do not generate the workbook. Return a short message that generation is not possible until the missing input is provided.

## 2. Date Window

1. Compute the intersection of the rough range and the precise workday range.
2. Use the intersection as the final generation window.
3. If the precise range is empty, fall back to the rough range.
4. Record the final start and end dates in the context artifact.

The precise range is derived from the JSON workday calendar by taking the earliest and latest dates in the usable workday subset that overlaps the rough range.

## 3. Weekly Splitting

- Split by natural week, Monday to Sunday.
- Record week number, start date, end date, and workday count.
- If a week has fewer than 3 actual days, merge it into the adjacent week and do not emit an independent week row.
- If the week crosses a month or year boundary, use the month/year of the later date in the workbook columns.

## 4. Project Stage Inference

Use the execution record stage if it exists. Otherwise infer from project progress:

- 0% - 20%: 需求分析
- 20% - 40%: 方案设计
- 40% - 75%: 开发实施
- 75% - 90%: 测试验证
- 90% - 100%: 交付验收

## 5. Work Allocation

Per week:

- Project work takes priority.
- Pre-sales work is capped at 1 day when needed.
- Daily work uses the remaining capacity.
- Do not exceed `workday_count * 8` hours in total.
- If the total required days are larger than the workday count, drop lower-priority rows first: daily, then pre-sales.

For each project row:

- Use the project cycle to decide whether the week intersects the project.
- If there is no intersection, assign 0.
- If there is an intersection, allocate the configured project days for that week.
- Convert days to hours with `days * 8`.

## 6. Task Category Pool

Every project row must choose a task category before labels or content are generated.

Priority order:

1. Execution record
2. Important milestone
3. Project stage
4. Project nature
5. Default role category

Suggested category pools:

- 需求分析: 需求收集, 需求分析, 需求确认, 需求拆解, 需求基线管理
- 方案设计: 技术调研, 总体方案设计, 数据模型设计, 接口设计, 风险分析
- 开发实施: 功能开发, 数据处理, 模块联调, 缺陷修复, 性能优化
- 测试验证: 测试准备, 测试执行, 缺陷验证, 测试分析
- 交付验收: 交付准备, 文档整理, 验收支持, 上线保障
- 售前: 商机跟踪, 需求调研, 技术方案编制, 成本测算, 投标支持, 产品演示, 技术答疑
- 日常: 部门资源规划, 项目统计分析, 流程优化, 制度建设, 绩效管理, 质量监督, 风险管控, 数据治理, 资产管理

### De-duplication Rules

Maintain a history pool with:

- week number
- work type
- project name
- project stage
- task category

When selecting a category:

- Do not let the same category appear more than 2 consecutive weeks.
- For the same project, do not repeat a category within the last 4 weeks.
- For the same work type, do not repeat the same category more than 2 times in the last 6 weeks.
- Every 4 consecutive weeks must include at least 2 different categories.
- Prefer the category with the lowest historical frequency.
- If a category repeats, switch to another valid category for the current stage.

## 7. Task Tags

Generate the tag from the combination of work type, project stage, role, and execution record.

Priority order:

1. Execution record
2. Important milestone
3. Project stage
4. Project nature
5. Default role tag

Rules:

- Tags must match the role.
- Tags must match the work type.
- Tags must match the current stage.
- Tags must not stack duplicates.
- The same tag should not repeat for too many consecutive weeks.

## 8. Work Content

Generate content from:

- work type
- project name
- project stage
- task tag
- task record

Process:

1. Draft the content.
2. Extract a short summary as `core action + core object`.
3. Check the historical work-content pool.
4. If the summary is similar to prior content, rewrite the content and recheck.
5. Keep only semantically unique content.

Do not rely only on changing verbs, project names, dates, or counts.

## 9. Workbook Contract

Required columns in this order:

1. 汇报人
2. 年份
3. 月份
4. 周次
5. 周起止时间
6. 工作性质
7. 项目名称
8. 项目性质
9. 项目阶段
10. 周工时
11. 任务标签
12. 工作内容

Formatting rules:

- Header fill color: `#4574C4`
- Body fill color: default white
- Merge the same week's cells in the `周次` column when the workbook groups rows by week.
- Use `YYYY.MM.DD-YYYY.MM.DD` for `周起止时间`.
- If the row is project work, fill `项目名称`, `项目性质`, and `项目阶段`.
- If the row is not project work, leave project-specific columns blank unless a source rule explicitly requires them.

Row order within each week:

1. Project rows
2. Pre-sales row
3. Daily row

Project rows are one row per project.

## 9.1 Input Field Mapping

When the user supplies free-form text, map the content into these slots before processing:

- `姓名` -> reporter name used in the workbook
- `工作职责` -> role and work-type constraints
- `粗略日期范围` -> outer date window
- `项目清单` -> project records and stage inputs

Do not reuse older hardcoded examples as defaults when the user has provided replacements.

## 10. Intermediate Files

Write the following before final export:

- `01_context.json`
- `02_weeks.csv`
- `03_allocation.csv`
- `04_category_pool.json`
- `05_draft_rows.csv`
- `06_validation.json`
- `07_validation_report.md`
- `final.xlsx`

The goal of the intermediates is to make the generation auditable and easy to repair.

## 11. Validation Checklist

Check all of the following:

- Tags match the work type, stage, and role.
- Work content matches the tag and work type.
- There is no stage regression.
- Work content is longer than 25 Chinese characters.
- Project hours are correct.
- Project dates stay within the project cycle.
- No negative hours exist.
- Priority rules are respected.
- Weekly total hours do not exceed `workday_count * 8`.
- The workbook schema and header color match the contract.

If any check fails, fix the affected row set and rerun the check before writing the final workbook.
