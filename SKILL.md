---
name: generate-weekly-worklog
description: Generate validated weekly worklog .xlsx files from explicit inputs for name, role duties, rough date range, and project list, using project, pre-sales, and daily rules with weekly grouping, stage inference, deduplication, and spreadsheet checks.
---

# Generate Weekly Worklog

## Overview

Use this skill when the user wants a real `.xlsx` workbook of weekly work content and provides all required inputs explicitly. It turns the context into a week-by-week plan, generates project / pre-sales / daily rows, and validates the result before saving.

## Use When

- The user asks for weekly work content, 周报工时, 工时填写, or similar.
- The output must be a real `.xlsx` file.
- The required inputs are present: `姓名`, `工作职责`, `粗略日期范围`, `项目清单`.
- The inputs come from attachments or local source files.
- The workbook must stay realistic, stage-aware, and non-repetitive.

## Required Inputs

Before generating anything, confirm that the user supplied all four fields:

- `姓名`
- `工作职责`
- `粗略日期范围`
- `项目清单`

If any one is missing, stop and tell the user the workbook cannot be generated yet.

Do not infer these four fields from old fixed wording unless the user explicitly includes them in the current request.

## Workflow

1. Read all provided attachments and normalize the facts.
2. Determine the actual generation window from the rough range and the precise workday range in `references/workday_calendar.json`.
3. Split the window into natural weeks and merge tails shorter than 3 days into adjacent weeks.
4. Parse project metadata, execution notes, milestones, and project stage.
5. Allocate weekly hours across project, pre-sales, and daily work with priority `项目 > 售前 > 日常`.
6. Generate task categories, labels, and unique work content.
7. Write intermediate artifacts before the final workbook.
8. Run the validation checklist and regenerate any failed rows.
9. Export the final `.xlsx`.

## Input Substitution Rules

Replace any previously fixed wording with the current user inputs:

- Reporter name comes from `姓名`.
- Role constraints and work-type mix come from `工作职责`.
- Date window starts from `粗略日期范围`.
- Project metadata and cycle facts come from `项目清单`.

If the user supplies these fields in prose instead of labels, map them to the same required inputs before generation.

## Intermediate Artifacts

Create a run folder such as `outputs/<run-id>/` and write these files before final export:

- `01_context.json` - normalized date range, role facts, project facts, assumptions.
- `02_weeks.csv` - week index, start/end dates, workday count.
- `03_allocation.csv` - week x work type x project day allocation.
- `04_category_pool.json` - category history and selection state.
- `05_draft_rows.csv` - row-level workbook draft before formatting.
- `06_validation.json` - pass/fail results for each check.
- `07_validation_report.md` - human-readable issues and fixes.
- `references/workday_calendar.json` - embedded workday calendar source.
- `final.xlsx` - the delivered workbook.

Do not skip these intermediates when the task is non-trivial.

## Validation

Before finalizing, verify:

- The final date window is the intersection of the rough range and the precise workday range.
- Week merging rules were applied correctly.
- Project hours do not exceed the project cycle and no negative hours exist.
- `项目 > 售前 > 日常` priority was respected when workdays were insufficient.
- Task categories obey the history and de-duplication constraints.
- Labels match role, work type, and stage.
- Work content is unique, longer than 25 Chinese characters, and consistent with the tag.
- Weekly total hours never exceed `工作日天数 × 8`.
- The final workbook column order and header color are correct.
- `周次` is written as an integer only, with no week label or extra text.
- If `工作内容` mentions a project name, the project name must appear in full and must not be abbreviated.

If any check fails, revise only the affected rows, rerun validation, and regenerate the workbook.

## Reference

See [workflow.md](references/workflow.md) for the detailed rulebook and output contract.
