---
name: gsd:plan-phase
description: Создать детальный план выполнения для конкретной фазы
argument-hint: "<номер> [--skip-research] [--skip-verify]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<context>
**Аргументы:**
- `<номер>` — Номер фазы для планирования (обязательно)
- `--skip-research` — Пропустить этап исследования
- `--skip-verify` — Пропустить проверку плана
</context>

<objective>
Создать детальный план выполнения для указанной фазы.

**Создаёт:**
- `.planning/phases/XX-название/XX-YY-PLAN.md` — атомарные планы задач
- `.planning/phases/XX-название/XX-RESEARCH.md` — исследование фазы (если не пропущено)

**После этой команды:** Запустите `/gsd:execute-phase <номер>` для выполнения.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/plan-phase.md
</execution_context>

<process>
Выполни рабочий процесс планирования фазы из @~/.claude/get-shit-done/workflows/plan-phase.md.
Сохрани все контрольные точки (исследование, планирование, проверка плана).
</process>
