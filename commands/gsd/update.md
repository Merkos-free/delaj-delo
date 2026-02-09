---
name: gsd:update
description: Обновить ДелайДело до последней версии с предпросмотром изменений
allowed-tools:
  - Read
  - Bash
  - Write
  - AskUserQuestion
---
<objective>
Обновить ДелайДело до последней версии с предпросмотром изменений.

- Показывает сравнение установленной и последней версий
- Отображает записи изменений для пропущенных версий
- Выделяет критические изменения
- Подтверждает перед запуском установки
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/update.md
</execution_context>

<process>
Выполни рабочий процесс обновления из @~/.claude/get-shit-done/workflows/update.md.
</process>
