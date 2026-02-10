<purpose>
Интерактивная настройка агентов рабочего процесса GSD (research, plan_check, verifier) и выбор профиля модели через многовопросный промпт. Обновляет .planning/config.json предпочтениями пользователя.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="ensure_and_load_config">
Убедитесь, что конфиг существует, и загрузите текущее состояние:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js config-ensure-section
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
```

Создаёт `.planning/config.json` с значениями по умолчанию, если файл отсутствует, и загружает текущие значения конфигурации.
</step>

<step name="read_current">
```bash
cat .planning/config.json
```

Распарсите текущие значения (по умолчанию `true`, если не указано):
- `workflow.research` — запускать исследователя во время plan-phase
- `workflow.plan_check` — запускать проверщика плана во время plan-phase
- `workflow.verifier` — запускать верификатора во время execute-phase
- `model_profile` — какую модель использует каждый агент (по умолчанию: `balanced`)
- `git.branching_strategy` — стратегия ветвления (по умолчанию: `"none"`)
</step>

<step name="present_settings">
Используйте AskUserQuestion с предварительно выбранными текущими значениями:

```
AskUserQuestion([
  {
    question: "Какой профиль модели для агентов?",
    header: "Модель",
    multiSelect: false,
    options: [
      { label: "Quality", description: "Opus везде кроме верификации (наибольшая стоимость)" },
      { label: "Balanced (Рекомендуется)", description: "Opus для планирования, Sonnet для выполнения/верификации" },
      { label: "Budget", description: "Sonnet для написания, Haiku для исследования/верификации (наименьшая стоимость)" }
    ]
  },
  {
    question: "Запускать исследователя плана? (исследует предметную область перед планированием)",
    header: "Исследование",
    multiSelect: false,
    options: [
      { label: "Да", description: "Исследовать цели фазы перед планированием" },
      { label: "Нет", description: "Пропустить исследование, планировать сразу" }
    ]
  },
  {
    question: "Запускать проверщика плана? (проверяет планы перед выполнением)",
    header: "Проверка плана",
    multiSelect: false,
    options: [
      { label: "Да", description: "Проверить, что планы соответствуют целям фазы" },
      { label: "Нет", description: "Пропустить проверку плана" }
    ]
  },
  {
    question: "Запускать верификатора выполнения? (проверяет завершение фазы)",
    header: "Верификатор",
    multiSelect: false,
    options: [
      { label: "Да", description: "Проверить обязательные требования после выполнения" },
      { label: "Нет", description: "Пропустить пост-верификацию" }
    ]
  },
  {
    question: "Стратегия ветвления Git?",
    header: "Ветвление",
    multiSelect: false,
    options: [
      { label: "Нет (Рекомендуется)", description: "Коммитить прямо в текущую ветку" },
      { label: "По фазе", description: "Создавать ветку для каждой фазы (gsd/phase-{N}-{name})" },
      { label: "По этапу", description: "Создавать ветку для всего этапа (gsd/{version}-{name})" }
    ]
  }
])
```
</step>

<step name="update_config">
Объедините новые настройки с существующим config.json:

```json
{
  ...existing_config,
  "model_profile": "quality" | "balanced" | "budget",
  "workflow": {
    "research": true/false,
    "plan_check": true/false,
    "verifier": true/false
  },
  "git": {
    "branching_strategy": "none" | "phase" | "milestone"
  }
}
```

Запишите обновлённый конфиг в `.planning/config.json`.
</step>

<step name="confirm">
Покажите:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 GSD ► НАСТРОЙКИ ОБНОВЛЕНЫ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Настройка              | Значение |
|------------------------|----------|
| Профиль модели         | {quality/balanced/budget} |
| Исследователь плана    | {Вкл/Выкл} |
| Проверщик плана        | {Вкл/Выкл} |
| Верификатор выполнения | {Вкл/Выкл} |
| Git ветвление          | {Нет/По фазе/По этапу} |

Эти настройки применяются к будущим запускам /gsd:plan-phase и /gsd:execute-phase.

Быстрые команды:
- /gsd:set-profile <profile> — переключить профиль модели
- /gsd:plan-phase --research — принудительно исследовать
- /gsd:plan-phase --skip-research — пропустить исследование
- /gsd:plan-phase --skip-verify — пропустить проверку плана
```
</step>

</process>

<success_criteria>
- [ ] Текущий конфиг прочитан
- [ ] Пользователю представлены 5 настроек (профиль + 3 переключателя рабочего процесса + git ветвление)
- [ ] Конфиг обновлён секциями model_profile, workflow и git
- [ ] Изменения подтверждены пользователю
</success_criteria>
