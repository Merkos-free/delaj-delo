---
name: gsd:plan-milestone-gaps
description: Создать фазы для закрытия пробелов, выявленных аудитом
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Создать фазы для закрытия пробелов, выявленных аудитом.

- Читает MILESTONE-AUDIT.md и группирует пробелы в фазы
- Приоритизирует по приоритету требований (обязательно/желательно/хорошо бы)
- Добавляет фазы закрытия пробелов в ROADMAP.md
- Готов к `/gsd:plan-phase` для новых фаз
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/plan-milestone-gaps.md
</execution_context>

<process>
Выполни рабочий процесс планирования пробелов вехи из @~/.claude/get-shit-done/workflows/plan-milestone-gaps.md.
</process>
