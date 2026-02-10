<purpose>
Захватить идею, задачу или проблему, которая всплыла во время сессии GSD, как структурированное задание для дальнейшей работы. Обеспечивает поток «мысль → захват → продолжение» без потери контекста.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="init_context">
Загрузите контекст задач:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init todos)
```

Извлеките из JSON инициализации: `commit_docs`, `date`, `timestamp`, `todo_count`, `todos`, `pending_dir`, `todos_dir_exists`.

Убедитесь, что каталоги существуют:
```bash
mkdir -p .planning/todos/pending .planning/todos/done
```

Запомните существующие области из массива todos для согласованности на шаге infer_area.
</step>

<step name="extract_content">
**С аргументами:** Используйте как заголовок/фокус.
- `/gsd:add-todo Добавить обновление auth-токена` → title = "Добавить обновление auth-токена"

**Без аргументов:** Проанализируйте недавний разговор для извлечения:
- Конкретной проблемы, идеи или задачи, обсуждённой в разговоре
- Упомянутых путей к файлам
- Технических деталей (сообщения об ошибках, номера строк, ограничения)

Сформулируйте:
- `title`: описательный заголовок из 3-10 слов (предпочтительно с глаголом действия)
- `problem`: Что не так или почему это необходимо
- `solution`: Подсказки по подходу или "TBD", если это просто идея
- `files`: Соответствующие пути с номерами строк из разговора
</step>

<step name="infer_area">
Определите область по путям файлов:

| Шаблон пути | Область |
|-------------|--------|
| `src/api/*`, `api/*` | `api` |
| `src/components/*`, `src/ui/*` | `ui` |
| `src/auth/*`, `auth/*` | `auth` |
| `src/db/*`, `database/*` | `database` |
| `tests/*`, `__tests__/*` | `testing` |
| `docs/*` | `docs` |
| `.planning/*` | `planning` |
| `scripts/*`, `bin/*` | `tooling` |
| Нет файлов или неясно | `general` |

Используйте существующую область из шага 2, если есть похожее совпадение.
</step>

<step name="check_duplicates">
```bash
# Поиск ключевых слов из заголовка в существующих задачах
grep -l -i "[ключевые слова из заголовка]" .planning/todos/pending/*.md 2>/dev/null
```

Если найден потенциальный дубликат:
1. Прочитайте существующую задачу
2. Сравните область охвата

При совпадении используйте AskUserQuestion:
- header: "Дубликат?"
- question: "Похожая задача существует: [заголовок]. Что вы хотите сделать?"
- options:
  - "Пропустить" — оставить существующую задачу
  - "Заменить" — обновить существующую новым контекстом
  - "Добавить всё равно" — создать как отдельную задачу
</step>

<step name="create_file">
Используйте значения из контекста инициализации: `timestamp` и `date` уже доступны.

Сгенерируйте slug для заголовка:
```bash
slug=$(node ~/.claude/get-shit-done/bin/gsd-tools.js generate-slug "$title" --raw)
```

Запишите в `.planning/todos/pending/${date}-${slug}.md`:

```markdown
---
created: [timestamp]
title: [title]
area: [area]
files:
  - [file:lines]
---

## Problem

[описание проблемы - достаточно контекста, чтобы будущий Claude понял спустя недели]

## Solution

[подсказки по подходу или "TBD"]
```
</step>

<step name="update_state">
Если `.planning/STATE.md` существует:

1. Используйте `todo_count` из контекста инициализации (или перезапустите `init todos`, если счётчик изменился)
2. Обновите "### Pending Todos" в секции "## Accumulated Context"
</step>

<step name="git_commit">
Закоммитьте задачу и обновлённое состояние:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: capture todo - [title]" --files .planning/todos/pending/[filename] .planning/STATE.md
```

Инструмент автоматически учитывает настройку `commit_docs` и gitignore.

Подтвердите: "Закоммичено: docs: capture todo - [title]"
</step>

<step name="confirm">
```
Задача сохранена: .planning/todos/pending/[filename]

  [title]
  Область: [area]
  Файлы: [количество] упомянуто

---

Что вы хотите сделать:

1. Продолжить текущую работу
2. Добавить ещё одну задачу
3. Просмотреть все задачи (/gsd:check-todos)
```
</step>

</process>

<success_criteria>
- [ ] Структура каталогов существует
- [ ] Файл задачи создан с корректным frontmatter
- [ ] Секция Problem содержит достаточно контекста для будущего Claude
- [ ] Дубликатов нет (проверено и разрешено)
- [ ] Область согласована с существующими задачами
- [ ] STATE.md обновлён, если существует
- [ ] Задача и состояние закоммичены в git
</success_criteria>
