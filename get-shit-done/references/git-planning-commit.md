# Git-коммит планирования

Коммит артефактов планирования через CLI gsd-tools, который автоматически проверяет конфигурацию `commit_docs` и статус gitignore.

## Коммит через CLI

Всегда используйте `gsd-tools.js commit` для файлов `.planning/` — он автоматически обрабатывает `commit_docs` и проверки gitignore:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs({scope}): {описание}" --files .planning/STATE.md .planning/ROADMAP.md
```

CLI вернёт `skipped` (с причиной), если `commit_docs` равно `false` или `.planning/` в gitignore. Ручные условные проверки не нужны.

## Дополнение предыдущего коммита

Чтобы добавить изменения файлов `.planning/` в предыдущий коммит:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "" --files .planning/codebase/*.md --amend
```

## Паттерны сообщений коммитов

| Команда | Область | Пример |
|---------|---------|--------|
| plan-phase | фаза | `docs(phase-03): создать планы аутентификации` |
| execute-phase | фаза | `docs(phase-03): завершить фазу аутентификации` |
| new-milestone | веха | `docs: начать веху v1.1` |
| remove-phase | chore | `chore: удалить фазу 17 (панель)` |
| insert-phase | фаза | `docs: вставить фазу 16.1 (критическое исправление)` |
| add-phase | фаза | `docs: добавить фазу 07 (страница настроек)` |

## Когда пропускать

- `commit_docs: false` в конфигурации
- `.planning/` в gitignore
- Нет изменений для коммита (проверить через `git status --porcelain .planning/`)