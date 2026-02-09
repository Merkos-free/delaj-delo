---
name: gsd:pause-work
description: Сохранить контекст при приостановке работы
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Создать передачу контекста при приостановке работы посреди фазы.

- Создаёт файл .continue-here с текущим состоянием
- Обновляет секцию непрерывности сессии в STATE.md
- Фиксирует контекст незавершённой работы
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/pause-work.md
</execution_context>

<process>
Выполни рабочий процесс паузы из @~/.claude/get-shit-done/workflows/pause-work.md.
</process>
