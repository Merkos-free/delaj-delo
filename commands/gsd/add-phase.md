---
name: gsd:add-phase
description: Добавить новую фазу в конец текущей вехи
argument-hint: "<описание>"
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Добавить новую фазу в конец текущей вехи.

- Добавляет в ROADMAP.md
- Использует следующий порядковый номер
- Обновляет структуру директорий фаз

Использование: `/gsd:add-phase "Добавить панель администратора"`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/add-phase.md
</execution_context>

<process>
Выполни рабочий процесс добавления фазы из @~/.claude/get-shit-done/workflows/add-phase.md.
</process>
