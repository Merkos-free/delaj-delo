# Шаблон промпта суб-агента планировщика

Шаблон для запуска агента gsd-planner. Агент содержит всю экспертизу планирования — этот шаблон передаёт только контекст планирования.

---

## Шаблон

```markdown
<planning_context>

**Фаза:** {phase_number}
**Режим:** {standard | gap_closure}

**Состояние проекта:**
@.planning/STATE.md

**Дорожная карта:**
@.planning/ROADMAP.md

**Требования (если существуют):**
@.planning/REQUIREMENTS.md

**Контекст фазы (если существует):**
@.planning/phases/{phase_dir}/{phase}-CONTEXT.md

**Исследование (если существует):**
@.planning/phases/{phase_dir}/{phase}-RESEARCH.md

**Закрытие пробелов (если режим --gaps):**
@.planning/phases/{phase_dir}/{phase}-VERIFICATION.md
@.planning/phases/{phase_dir}/{phase}-UAT.md

</planning_context>

<downstream_consumer>
Результат потребляется /gsd:execute-phase
Планы должны быть исполняемыми промптами с:
- Фронтматтером (wave, depends_on, files_modified, autonomous)
- Задачами в формате XML
- Критериями верификации
- must_haves для обратной верификации от цели
</downstream_consumer>

<quality_gate>
Перед возвратом ПЛАНИРОВАНИЕ ЗАВЕРШЕНО:
- [ ] Файлы PLAN.md созданы в директории фазы
- [ ] Каждый план имеет валидный фронтматтер
- [ ] Задачи конкретны и исполнимы
- [ ] Зависимости корректно определены
- [ ] Волны назначены для параллельного выполнения
- [ ] must_haves выведены из цели фазы
</quality_gate>
```

---

## Плейсхолдеры

| Плейсхолдер | Источник | Пример |
|-------------|----------|---------|
| `{phase_number}` | Из дорожной карты/аргументов | `5` или `2.1` |
| `{phase_dir}` | Имя директории фазы | `05-user-profiles` |
| `{phase}` | Префикс фазы | `05` |
| `{standard \| gap_closure}` | Флаг режима | `standard` |

---

## Использование

**Из /gsd:plan-phase (стандартный режим):**
```python
Task(
  prompt=filled_template,
  subagent_type="gsd-planner",
  description="Планирование фазы {phase}"
)
```

**Из /gsd:plan-phase --gaps (режим закрытия пробелов):**
```python
Task(
  prompt=filled_template,  # с mode: gap_closure
  subagent_type="gsd-planner",
  description="Планирование пробелов для фазы {phase}"
)
```

---

## Продолжение

Для контрольных точек запустите нового агента с:

```markdown
<objective>
Продолжить планирование фазы {phase_number}: {phase_name}
</objective>

<prior_state>
Директория фазы: @.planning/phases/{phase_dir}/
Существующие планы: @.planning/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**Тип:** {checkpoint_type}
**Ответ:** {user_response}
</checkpoint_response>

<mode>
Продолжить: {standard | gap_closure}
</mode>
```

---

**Примечание:** Методология планирования, декомпозиция задач, анализ зависимостей, назначение волн, определение TDD и обратная деривация от цели встроены в агент gsd-planner. Этот шаблон только передаёт контекст.
