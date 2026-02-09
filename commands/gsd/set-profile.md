---
name: gsd:set-profile
description: Быстро переключить профиль модели для агентов ДД
argument-hint: "<профиль>"
allowed-tools:
  - Read
  - Bash
  - Write
---
<objective>
Быстро переключить профиль модели для агентов ДелайДело.

- `quality` — Opus везде кроме верификации
- `balanced` — Opus для планирования, Sonnet для выполнения (по умолчанию)
- `budget` — Sonnet для написания, Haiku для исследования/верификации

Использование: `/gsd:set-profile budget`
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/set-profile.md
</execution_context>

<process>
Выполни рабочий процесс установки профиля из @~/.claude/get-shit-done/workflows/set-profile.md.
</process>
