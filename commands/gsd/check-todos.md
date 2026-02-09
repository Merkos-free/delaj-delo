---
name: gsd:check-todos
description: Показать список заметок и выбрать для работы
argument-hint: "[область]"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
Показать список ожидающих заметок и выбрать одну для работы.

- Перечисляет все ожидающие заметки с заголовком, областью, возрастом
- Опциональный фильтр по области (например, `/gsd:check-todos api`)
- Загружает полный контекст для выбранной заметки
- Направляет к подходящему действию (работать сейчас, добавить в фазу, обсудить)
- Перемещает заметку в done/ когда работа начинается

Использование: `/gsd:check-todos`
Использование: `/gsd:check-todos api`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/check-todos.md
</execution_context>

<process>
Выполни рабочий процесс проверки заметок из @~/.claude/get-shit-done/workflows/check-todos.md.
</process>
