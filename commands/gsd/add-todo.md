---
name: gsd:add-todo
description: Записать идею или задачу как заметку
argument-hint: "[описание]"
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Записать идею или задачу как заметку из текущего разговора.

- Извлекает контекст из разговора (или использует указанное описание)
- Создаёт структурированный файл заметки в `.planning/todos/pending/`
- Определяет область по путям файлов для группировки
- Проверяет дубликаты перед созданием
- Обновляет счётчик заметок в STATE.md

Использование: `/gsd:add-todo` (определяет из разговора)
Использование: `/gsd:add-todo Добавить обновление токена авторизации`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/add-todo.md
</execution_context>

<process>
Выполни рабочий процесс добавления заметки из @~/.claude/get-shit-done/workflows/add-todo.md.
</process>
