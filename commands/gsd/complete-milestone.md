---
name: gsd:complete-milestone
description: Архивировать завершённую веху и подготовить следующую версию
argument-hint: "<версия>"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
---
<objective>
Архивировать завершённую веху и подготовить к следующей версии.

- Создаёт запись в MILESTONES.md со статистикой
- Архивирует полные детали в директорию milestones/
- Создаёт git-тег для релиза
- Подготавливает рабочее пространство для следующей версии

Использование: `/gsd:complete-milestone 1.0.0`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/complete-milestone.md
</execution_context>

<process>
Выполни рабочий процесс завершения вехи из @~/.claude/get-shit-done/workflows/complete-milestone.md.
Сохрани все контрольные точки (архивация, тегирование, подготовка).
</process>
