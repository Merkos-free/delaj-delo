---
name: gsd:discuss-phase
description: Обсудить детали реализации фазы перед планированием
argument-hint: "<номер>"
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<context>
**Аргументы:**
- `<номер>` — Номер фазы для обсуждения
</context>

<objective>
Помочь сформулировать видение фазы перед планированием.

- Фиксирует как вы представляете работу этой фазы
- Создаёт CONTEXT.md с вашим видением, обязательными элементами и границами
- Используйте когда у вас есть идеи о том, как что-то должно выглядеть/ощущаться

**Создаёт:** `{фаза}-CONTEXT.md`

**После этой команды:** Запустите `/gsd:plan-phase <номер>` для создания плана.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/discuss-phase.md
</execution_context>

<process>
Выполни рабочий процесс обсуждения фазы из @~/.claude/get-shit-done/workflows/discuss-phase.md.
Сохрани все контрольные точки.
</process>
