---
name: gsd:execute-phase
description: Выполнить все планы фазы параллельными волнами
argument-hint: "<номер-фазы>"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
---
<context>
**Аргументы:**
- `<номер-фазы>` — Номер фазы для выполнения (обязательно)
</context>

<objective>
Выполнить все планы указанной фазы.

- Группирует планы по волнам, выполняет волны последовательно
- Планы внутри каждой волны запускаются параллельно через инструмент Task
- Проверяет цель фазы после завершения всех планов
- Обновляет REQUIREMENTS.md, ROADMAP.md, STATE.md

**После этой команды:** Запустите `/gsd:verify-work <номер>` для проверки.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/execute-phase.md
</execution_context>

<process>
Выполни рабочий процесс выполнения фазы из @~/.claude/get-shit-done/workflows/execute-phase.md.
Сохрани все контрольные точки (волны, коммиты, верификация).
</process>
