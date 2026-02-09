---
name: gsd:map-codebase
description: Проанализировать существующую кодовую базу перед созданием нового проекта
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
---
<objective>
Картирование существующей кодовой базы для браунфилд-проектов.

- Анализирует кодовую базу параллельными агентами-исследователями
- Создаёт `.planning/codebase/` с 7 фокусированными документами
- Покрывает стек, архитектуру, структуру, конвенции, тестирование, интеграции, проблемы
- Используйте перед `/gsd:new-project` на существующих кодовых базах

**Создаёт:**
- `.planning/codebase/STACK.md` — языки, фреймворки, зависимости
- `.planning/codebase/ARCHITECTURE.md` — паттерны, слои, поток данных
- `.planning/codebase/STRUCTURE.md` — структура директорий, ключевые файлы
- `.planning/codebase/CONVENTIONS.md` — стандарты кодирования, именование
- `.planning/codebase/TESTING.md` — настройка тестов, паттерны
- `.planning/codebase/INTEGRATIONS.md` — внешние сервисы, API
- `.planning/codebase/CONCERNS.md` — технический долг, известные проблемы
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/map-codebase.md
</execution_context>

<process>
Выполни рабочий процесс картирования кодовой базы из @~/.claude/get-shit-done/workflows/map-codebase.md.
</process>
