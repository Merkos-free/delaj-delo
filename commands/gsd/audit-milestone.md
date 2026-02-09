---
name: gsd:audit-milestone
description: Аудит завершения вехи относительно первоначального замысла
argument-hint: "[версия]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
---
<objective>
Аудит завершения вехи относительно первоначального замысла.

- Читает все файлы VERIFICATION.md фаз
- Проверяет покрытие требований
- Запускает проверку интеграции для межфазных связей
- Создаёт MILESTONE-AUDIT.md с пробелами и техническим долгом
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/audit-milestone.md
</execution_context>

<process>
Выполни рабочий процесс аудита вехи из @~/.claude/get-shit-done/workflows/audit-milestone.md.
</process>
