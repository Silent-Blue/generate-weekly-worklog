# generate-weekly-worklog

This repository contains the `generate-weekly-worklog` Codex skill at the repository root.

## Structure

- `SKILL.md`: skill instructions and trigger conditions
- `agents/openai.yaml`: UI metadata for the skill
- `references/workflow.md`: detailed workflow and validation rules
- `references/workday_calendar.json`: embedded workday calendar source

## Use

Invoke the skill when generating a validated weekly worklog `.xlsx` from explicit inputs:
- `姓名`
- `工作职责`
- `粗略日期范围`
- `项目清单`
