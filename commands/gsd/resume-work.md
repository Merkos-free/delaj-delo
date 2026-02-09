---
name: gsd:resume-work
description: Возобновить работу с полным восстановлением контекста
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
Возобновить работу с предыдущей сессии с полным восстановлением контекста.

- Читает STATE.md для контекста проекта
- Показывает текущую позицию и недавний прогресс
- Предлагает следующие действия на основе состояния проекта
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/resume-project.md
</execution_context>

<process>
Выполни рабочий процесс возобновления из @~/.claude/get-shit-done/workflows/resume-project.md.
</process>
