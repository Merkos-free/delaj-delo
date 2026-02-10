<planning_config>

Опции конфигурации для поведения каталога `.planning/`.

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"git": {
  "branching_strategy": "none",
  "phase_branch_template": "gsd/phase-{phase}-{slug}",
  "milestone_branch_template": "gsd/{milestone}-{slug}"
}
```

| Опция | По умолчанию | Описание |
|--------|---------|-------------|
| `commit_docs` | `true` | Коммитить ли артефакты планирования в git |
| `search_gitignored` | `false` | Добавлять `--no-ignore` к широким rg-поискам |
| `git.branching_strategy` | `"none"` | Стратегия ветвления: `"none"`, `"phase"` или `"milestone"` |
| `git.phase_branch_template` | `"gsd/phase-{phase}-{slug}"` | Шаблон ветки для стратегии фаз |
| `git.milestone_branch_template` | `"gsd/{milestone}-{slug}"` | Шаблон ветки для стратегии вех |
</config_schema>

<commit_docs_behavior>

**Когда `commit_docs: true` (по умолчанию):**
- Файлы планирования коммитятся обычным образом
- SUMMARY.md, STATE.md, ROADMAP.md отслеживаются в git
- Полная история решений планирования сохраняется

**Когда `commit_docs: false`:**
- Пропуск всех `git add`/`git commit` для файлов `.planning/`
- Пользователь должен добавить `.planning/` в `.gitignore`
- Полезно для: вкладов в OSS, клиентских проектов, приватного планирования

**Через gsd-tools.js (предпочтительно):**

```bash
# Коммит с автоматической проверкой commit_docs + gitignore:
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: update state" --files .planning/STATE.md

# Загрузка конфига через state load (возвращает JSON):
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
# commit_docs доступен в JSON-выводе

# Или через команды init, которые включают commit_docs:
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init execute-phase "1")
# commit_docs включён во все выводы init-команд
```

**Автоопределение:** Если `.planning/` в gitignore, `commit_docs` автоматически `false` независимо от config.json. Это предотвращает ошибки git когда у пользователей `.planning/` в `.gitignore`.

**Коммит через CLI (проверки выполняются автоматически):**

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: update state" --files .planning/STATE.md
```

CLI проверяет конфигурацию `commit_docs` и статус gitignore внутри — ручные условия не нужны.

</commit_docs_behavior>

<search_behavior>

**Когда `search_gitignored: false` (по умолчанию):**
- Стандартное поведение rg (учитывает .gitignore)
- Поиск по прямому пути работает: `rg "pattern" .planning/` находит файлы
- Широкие поиски пропускают gitignored: `rg "pattern"` пропускает `.planning/`

**Когда `search_gitignored: true`:**
- Добавлять `--no-ignore` к широким rg-поискам, которые должны включать `.planning/`
- Нужно только при поиске по всему репозиторию с ожиданием совпадений в `.planning/`

**Примечание:** Большинство операций ДД используют прямое чтение файлов или явные пути, которые работают независимо от статуса gitignore.

</search_behavior>

<setup_uncommitted_mode>

Для использования режима без коммитов:

1. **Настроить конфиг:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Добавить в .gitignore:**
   ```
   .planning/
   ```

3. **Существующие отслеживаемые файлы:** Если `.planning/` ранее отслеживался:
   ```bash
   git rm -r --cached .planning/
   git commit -m "chore: stop tracking planning docs"
   ```

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Стратегии ветвления:**

| Стратегия | Когда создаётся ветка | Область ветки | Точка слияния |
|----------|---------------------|--------------|-------------|
| `none` | Никогда | Н/Д | Н/Д |
| `phase` | При старте `execute-phase` | Одна фаза | Пользователь сливает после фазы |
| `milestone` | При первом `execute-phase` вехи | Вся веха | При `complete-milestone` |

**Когда `git.branching_strategy: "none"` (по умолчанию):**
- Вся работа коммитится в текущую ветку
- Стандартное поведение ДД

**Когда `git.branching_strategy: "phase"`:**
- `execute-phase` создаёт/переключается на ветку перед выполнением
- Имя ветки из `phase_branch_template` (напр., `gsd/phase-03-authentication`)
- Все коммиты планов идут в эту ветку
- Пользователь сливает ветки вручную после завершения фазы
- `complete-milestone` предлагает слить все ветки фаз

**Когда `git.branching_strategy: "milestone"`:**
- Первый `execute-phase` вехи создаёт ветку вехи
- Имя ветки из `milestone_branch_template` (напр., `gsd/v1.0-mvp`)
- Все фазы в вехе коммитятся в одну ветку
- `complete-milestone` предлагает слить ветку вехи в main

**Переменные шаблонов:**

| Переменная | Доступна в | Описание |
|----------|--------------|-------------|
| `{phase}` | phase_branch_template | Номер фазы с нулями (напр., "03") |
| `{slug}` | Оба | Нижний регистр, через дефис |
| `{milestone}` | milestone_branch_template | Версия вехи (напр., "v1.0") |

**Проверка конфига:**

Используйте `init execute-phase`, который возвращает весь конфиг в JSON:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init execute-phase "1")
# JSON-вывод включает: branching_strategy, phase_branch_template, milestone_branch_template
```

Или используйте `state load` для значений конфига:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
# Извлеките branching_strategy, phase_branch_template, milestone_branch_template из JSON
```

**Создание ветки:**

```bash
# Для стратегии фаз
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# Для стратегии вех
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**Опции слияния при complete-milestone:**

| Опция | Git-команда | Результат |
|--------|-------------|--------|
| Squash merge (рекомендуется) | `git merge --squash` | Один чистый коммит на ветку |
| Merge с историей | `git merge --no-ff` | Сохраняет все отдельные коммиты |
| Удалить без слияния | `git branch -D` | Отбросить работу ветки |
| Оставить ветки | (нет) | Ручная обработка позже |

Squash merge рекомендуется — поддерживает чистую историю main, сохраняя полную историю разработки в ветке (до удаления).

**Варианты использования:**

| Стратегия | Лучше всего для |
|----------|----------|
| `none` | Соло-разработка, простые проекты |
| `phase` | Код-ревью по фазам, гранулярный откат, командная работа |
| `milestone` | Ветки релизов, стейджинг-среды, PR на версию |

</branching_strategy_behavior>

</planning_config>
