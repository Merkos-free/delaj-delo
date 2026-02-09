---
name: gsd:new-milestone
description: Начать новую веху через единый поток
argument-hint: "[название]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<objective>
Начать новую веху через единый поток.

- Глубокий опрос для понимания что строим дальше
- Опциональное исследование предметной области (4 параллельных агента-исследователя)
- Определение требований с разграничением объёма
- Создание дорожной карты с разбивкой по фазам

Зеркалит поток `/gsd:new-project` для браунфилд-проектов (существующий PROJECT.md).

Использование: `/gsd:new-milestone "Фичи v2.0"`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/new-milestone.md
</execution_context>

<process>
Выполни рабочий процесс новой вехи из @~/.claude/get-shit-done/workflows/new-milestone.md.
Сохрани все контрольные точки.
</process>
