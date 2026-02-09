---
name: gsd:new-project
description: Инициализация нового проекта с глубоким сбором контекста и PROJECT.md
argument-hint: "[--auto]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Task
  - AskUserQuestion
---
<context>
**Флаги:**
- `--auto` — Автоматический режим. После вопросов по конфигурации запускает исследование → требования → дорожная карта без дальнейшего взаимодействия. Ожидает документ с идеей через @ ссылку.
</context>

<objective>
Инициализировать новый проект через единый поток: опрос → исследование (опционально) → требования → дорожная карта.

**Создаёт:**
- `.planning/PROJECT.md` — контекст проекта
- `.planning/config.json` — настройки рабочего процесса
- `.planning/research/` — исследование предметной области (опционально)
- `.planning/REQUIREMENTS.md` — скопированные требования
- `.planning/ROADMAP.md` — структура фаз
- `.planning/STATE.md` — память проекта

**После этой команды:** Запустите `/gsd:plan-phase 1` чтобы начать выполнение.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/new-project.md
@~/.claude/get-shit-done/references/questioning.md
@~/.claude/get-shit-done/references/ui-brand.md
@~/.claude/get-shit-done/templates/project.md
@~/.claude/get-shit-done/templates/requirements.md
</execution_context>

<process>
Выполни рабочий процесс нового проекта из @~/.claude/get-shit-done/workflows/new-project.md от начала до конца.
Сохрани все контрольные точки рабочего процесса (валидация, подтверждения, коммиты, маршрутизация).
</process>
