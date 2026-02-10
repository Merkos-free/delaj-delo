# Шаблон промпта для субагента отладки

Шаблон для запуска агента gsd-debugger. Агент содержит всю экспертизу по отладке — этот шаблон предоставляет только контекст проблемы.

---

## Шаблон

```markdown
<objective>
Исследовать проблему: {issue_id}

**Описание:** {issue_summary}
</objective>

<symptoms>
ожидаемое: {expected}
фактическое: {actual}
ошибки: {errors}
воспроизведение: {reproduction}
хронология: {timeline}
</symptoms>

<mode>
symptoms_prefilled: {true_or_false}
goal: {find_root_cause_only | find_and_fix}
</mode>

<debug_file>
Создать: .planning/debug/{slug}.md
</debug_file>
```

---

## Заполнители

| Заполнитель | Источник | Пример |
|-------------|----------|---------|
| `{issue_id}` | Назначен оркестратором | `auth-screen-dark` |
| `{issue_summary}` | Описание пользователя | `Экран авторизации слишком тёмный` |
| `{expected}` | Из симптомов | `Логотип чётко виден` |
| `{actual}` | Из симптомов | `Экран тёмный` |
| `{errors}` | Из симптомов | `Нет в консоли` |
| `{reproduction}` | Из симптомов | `Открыть /auth` |
| `{timeline}` | Из симптомов | `После недавнего деплоя` |
| `{goal}` | Устанавливает оркестратор | `find_and_fix` |
| `{slug}` | Генерируется | `auth-screen-dark` |

---

## Использование

**Из /gsd:debug:**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-debugger",
  description="Отладка {slug}"
)
```

**Из diagnose-issues (UAT):**
```python
Task(prompt=template, subagent_type="gsd-debugger", description="Отладка UAT-001")
```

---

## Продолжение

Для контрольных точек запустите нового агента с:

```markdown
<objective>
Продолжить отладку {slug}. Доказательства в файле отладки.
</objective>

<prior_state>
Файл отладки: @.planning/debug/{slug}.md
</prior_state>

<checkpoint_response>
**Тип:** {checkpoint_type}
**Ответ:** {user_response}
</checkpoint_response>

<mode>
goal: {goal}
</mode>
```
