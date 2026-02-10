<purpose>
Исследовать способ реализации фазы. Запускает gsd-phase-researcher с контекстом фазы.

Отдельная команда исследования. Для большинства рабочих процессов используйте `/gsd:plan-phase`, который автоматически интегрирует исследование.
</purpose>

<process>

## Шаг 0: Определение профиля модели

@~/.claude/get-shit-done/references/model-profile-resolution.md

Определите модель для:
- `gsd-phase-researcher`

## Шаг 1: Нормализация и валидация фазы

@~/.claude/get-shit-done/references/phase-argument-parsing.md

```bash
PHASE_INFO=$(node ~/.claude/get-shit-done/bin/gsd-tools.js roadmap get-phase "${PHASE}")
```

Если `found` равно false: Ошибка и выход.

## Шаг 2: Проверка существующего исследования

```bash
ls .planning/phases/${PHASE}-*/RESEARCH.md 2>/dev/null
```

Если существует: Предложить варианты обновить/просмотреть/пропустить.

## Шаг 3: Сбор контекста фазы

```bash
# Секция фазы из дорожной карты (уже загружена в PHASE_INFO)
echo "$PHASE_INFO" | jq -r '.section'
cat .planning/REQUIREMENTS.md 2>/dev/null
cat .planning/phases/${PHASE}-*/*-CONTEXT.md 2>/dev/null
# Решения из state-snapshot (структурированный JSON)
node ~/.claude/get-shit-done/bin/gsd-tools.js state-snapshot | jq '.decisions'
```

## Шаг 4: Запуск исследователя

```
Task(
  prompt="<objective>
Исследовать подход к реализации Фазы {phase}: {name}
</objective>

<context>
Описание фазы: {description}
Требования: {requirements}
Предыдущие решения: {decisions}
Контекст фазы: {context_md}
</context>

<o>
Записать в: .planning/phases/${PHASE}-{slug}/${PHASE}-RESEARCH.md
</o>",
  subagent_type="gsd-phase-researcher",
  model="{researcher_model}"
)
```

## Шаг 5: Обработка результата

- `## RESEARCH COMPLETE` — Показать сводку, предложить: Планировать/Углубиться/Просмотреть/Готово
- `## CHECKPOINT REACHED` — Представить пользователю, запустить продолжение
- `## RESEARCH INCONCLUSIVE` — Показать попытки, предложить: Добавить контекст/Попробовать другой режим/Вручную

</process>
