---
name: gsd:quick
description: Быстрое выполнение мелкой задачи с гарантиями ДД
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<objective>
Выполнить небольшую задачу с гарантиями ДелайДело, но пропуская опциональные агенты.

- Использует планировщик + исполнитель (пропускает исследователя, проверщика, верификатора)
- Быстрые задачи живут в `.planning/quick/` отдельно от запланированных фаз
- Обновляет STATE.md (не ROADMAP.md)

Используйте когда точно знаете что делать и задача достаточно маленькая.

**Создаёт:** `.planning/quick/NNN-slug/PLAN.md`, `.planning/quick/NNN-slug/SUMMARY.md`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/quick.md
</execution_context>

<process>
Выполни рабочий процесс быстрого режима из @~/.claude/get-shit-done/workflows/quick.md.
</process>
