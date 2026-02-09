---
name: gsd:remove-phase
description: Удалить будущую фазу и перенумеровать последующие
argument-hint: "<номер>"
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Удалить будущую фазу и перенумеровать последующие фазы.

- Удаляет директорию фазы и все ссылки
- Перенумеровывает все последующие фазы для закрытия пробела
- Работает только с будущими (не начатыми) фазами
- Git-коммит сохраняет историческую запись

Использование: `/gsd:remove-phase 17`
Результат: Фаза 17 удалена, фазы 18-20 становятся 17-19
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/remove-phase.md
</execution_context>

<process>
Выполни рабочий процесс удаления фазы из @~/.claude/get-shit-done/workflows/remove-phase.md.
</process>
