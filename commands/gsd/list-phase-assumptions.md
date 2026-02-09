---
name: gsd:list-phase-assumptions
description: Посмотреть предполагаемый подход Claude к фазе перед планированием
argument-hint: "<номер>"
allowed-tools:
  - Read
  - Bash
---
<objective>
Посмотреть что Claude собирается делать до начала планирования.

- Показывает предполагаемый подход Claude к фазе
- Позволяет скорректировать курс если Claude неправильно понял ваше видение
- Файлы не создаются — только текстовый вывод в диалоге
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/list-phase-assumptions.md
</execution_context>

<process>
Выполни рабочий процесс показа предположений из @~/.claude/get-shit-done/workflows/list-phase-assumptions.md.
</process>
