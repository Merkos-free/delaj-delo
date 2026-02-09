---
name: gsd:settings
description: Настроить переключатели рабочего процесса и профиль модели
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
Настроить переключатели рабочего процесса и профиль модели интерактивно.

- Переключить агентов: исследователь, проверщик планов, верификатор
- Выбрать профиль модели (качество/баланс/бюджет)
- Обновляет `.planning/config.json`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/settings.md
</execution_context>

<process>
Выполни рабочий процесс настроек из @~/.claude/get-shit-done/workflows/settings.md.
</process>
