---
name: gsd:insert-phase
description: Вставить срочную работу между существующими фазами
argument-hint: "<после> <описание>"
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Вставить срочную работу как промежуточную фазу между существующими.

- Создаёт промежуточную фазу (например, 7.1 между 7 и 8)
- Полезно для обнаруженной работы, которая должна быть выполнена в середине вехи
- Сохраняет порядок фаз

Использование: `/gsd:insert-phase 7 "Исправить критический баг авторизации"`
Результат: Создаёт Фазу 7.1
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/insert-phase.md
</execution_context>

<process>
Выполни рабочий процесс вставки фазы из @~/.claude/get-shit-done/workflows/insert-phase.md.
</process>
