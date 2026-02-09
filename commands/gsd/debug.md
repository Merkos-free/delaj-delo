---
name: gsd:debug
description: Систематическая отладка с сохранением состояния между сессиями
argument-hint: "[описание проблемы]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<context>
**Аргументы:**
- `[описание проблемы]` — Описание проблемы (опционально, без аргументов — возобновление активной сессии)
</context>

<objective>
Систематическая отладка с сохранением состояния между сбросами контекста.

- Собирает симптомы через адаптивные вопросы
- Создаёт `.planning/debug/[slug].md` для отслеживания расследования
- Исследует научным методом (доказательства → гипотеза → тест)
- Выживает после `/clear` — запустите `/gsd:debug` без аргументов для возобновления
- Архивирует решённые проблемы в `.planning/debug/resolved/`

Использование: `/gsd:debug "кнопка входа не работает"`
Использование: `/gsd:debug` (возобновить активную сессию)
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/diagnose-issues.md
@~/.claude/get-shit-done/templates/DEBUG.md
</execution_context>

<process>
Выполни рабочий процесс отладки.
Сохрани все контрольные точки.
</process>
