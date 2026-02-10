<purpose>
Выполнить небольшие разовые задачи с гарантиями GSD (атомарные коммиты, отслеживание в STATE.md) при пропуске опциональных агентов (исследование, проверщик плана, верификатор). Быстрый режим запускает gsd-planner (быстрый режим) + gsd-executor(ы), отслеживает задачи в `.planning/quick/` и обновляет таблицу "Quick Tasks Completed" в STATE.md.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>
**Шаг 1: Получить описание задачи**

Интерактивно запросите у пользователя описание задачи:

```
AskUserQuestion(
  header: "Быстрая задача",
  question: "Что вы хотите сделать?",
  followUp: null
)
```

Сохраните ответ как `$DESCRIPTION`.

Если пусто, запросите повторно: "Пожалуйста, укажите описание задачи."

---

**Шаг 2: Инициализация**

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init quick "$DESCRIPTION")
```

Распарсите JSON: `planner_model`, `executor_model`, `commit_docs`, `next_num`, `slug`, `date`, `timestamp`, `quick_dir`, `task_dir`, `roadmap_exists`, `planning_exists`.

**Если `roadmap_exists` равно false:** Ошибка — Быстрый режим требует активного проекта с ROADMAP.md. Сначала запустите `/gsd:new-project`.

Быстрые задачи могут запускаться в середине фазы — валидация проверяет только существование ROADMAP.md, а не статус фазы.

---

**Шаг 3: Создание каталога задачи**

```bash
mkdir -p "${task_dir}"
```

---

**Шаг 4: Создание каталога быстрой задачи**

Создайте каталог для этой быстрой задачи:

```bash
QUICK_DIR=".planning/quick/${next_num}-${slug}"
mkdir -p "$QUICK_DIR"
```

Сообщите пользователю:
```
Создание быстрой задачи ${next_num}: ${DESCRIPTION}
Каталог: ${QUICK_DIR}
```

Сохраните `$QUICK_DIR` для использования в оркестрации.

---

**Шаг 5: Запуск планировщика (быстрый режим)**

Запустите gsd-planner с контекстом быстрого режима:

```
Task(
  prompt="
<planning_context>

**Режим:** quick
**Каталог:** ${QUICK_DIR}
**Описание:** ${DESCRIPTION}

**Состояние проекта:**
@.planning/STATE.md

</planning_context>

<constraints>
- Создать ОДИН план с 1-3 сфокусированными задачами
- Быстрые задачи должны быть атомарными и самодостаточными
- Без фазы исследования, без фазы проверки
- Цель ~30% использования контекста (просто, сфокусированно)
</constraints>

<o>
Записать план в: ${QUICK_DIR}/${next_num}-PLAN.md
Вернуть: ## PLANNING COMPLETE с путём к плану
</o>
",
  subagent_type="gsd-planner",
  model="{planner_model}",
  description="Быстрый план: ${DESCRIPTION}"
)
```

После возврата планировщика:
1. Убедитесь, что план существует по пути `${QUICK_DIR}/${next_num}-PLAN.md`
2. Извлеките количество планов (обычно 1 для быстрых задач)
3. Сообщите: "План создан: ${QUICK_DIR}/${next_num}-PLAN.md"

Если план не найден, ошибка: "Планировщик не смог создать ${next_num}-PLAN.md"

---

**Шаг 6: Запуск исполнителя**

Запустите gsd-executor со ссылкой на план:

```
Task(
  prompt="
Выполнить быструю задачу ${next_num}.

План: @${QUICK_DIR}/${next_num}-PLAN.md
Состояние проекта: @.planning/STATE.md

<constraints>
- Выполнить все задачи в плане
- Коммитить каждую задачу атомарно
- Создать отчёт: ${QUICK_DIR}/${next_num}-SUMMARY.md
- НЕ обновлять ROADMAP.md (быстрые задачи отделены от запланированных фаз)
</constraints>
",
  subagent_type="gsd-executor",
  model="{executor_model}",
  description="Выполнить: ${DESCRIPTION}"
)
```

После возврата исполнителя:
1. Убедитесь, что отчёт существует по пути `${QUICK_DIR}/${next_num}-SUMMARY.md`
2. Извлеките хеш коммита из вывода исполнителя
3. Сообщите статус завершения

**Известный баг Claude Code (classifyHandoffIfNeeded):** Если исполнитель сообщает "failed" с ошибкой `classifyHandoffIfNeeded is not defined`, это баг рантайма Claude Code — не реальный сбой. Проверьте, существует ли файл summary и показывает ли git log коммиты. Если да, считайте успешным.

Если отчёт не найден, ошибка: "Исполнитель не смог создать ${next_num}-SUMMARY.md"

Примечание: Для быстрых задач с несколькими планами (редко) запускайте исполнителей параллельными волнами по шаблонам execute-phase.

---

**Шаг 7: Обновление STATE.md**

Обновите STATE.md записью о завершении быстрой задачи.

**7a. Проверьте существование секции "Quick Tasks Completed":**

Прочитайте STATE.md и проверьте наличие секции `### Quick Tasks Completed`.

**7b. Если секция не существует, создайте её:**

Вставьте после секции `### Blockers/Concerns`:

```markdown
### Quick Tasks Completed

| # | Description | Date | Commit | Directory |
|---|-------------|------|--------|-----------|
```

**7c. Добавьте новую строку в таблицу:**

Используйте `date` из инициализации:
```markdown
| ${next_num} | ${DESCRIPTION} | ${date} | ${commit_hash} | [${next_num}-${slug}](./quick/${next_num}-${slug}/) |
```

**7d. Обновите строку "Last activity":**

Используйте `date` из инициализации:
```
Last activity: ${date} - Completed quick task ${next_num}: ${DESCRIPTION}
```

Используйте инструмент Edit для атомарного выполнения этих изменений

---

**Шаг 8: Финальный коммит и завершение**

Проиндексируйте и закоммитьте артефакты быстрой задачи:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(quick-${next_num}): ${DESCRIPTION}" --files ${QUICK_DIR}/${next_num}-PLAN.md ${QUICK_DIR}/${next_num}-SUMMARY.md .planning/STATE.md
```

Получите финальный хеш коммита:
```bash
commit_hash=$(git rev-parse --short HEAD)
```

Покажите вывод завершения:
```
---

GSD > БЫСТРАЯ ЗАДАЧА ВЫПОЛНЕНА

Быстрая задача ${next_num}: ${DESCRIPTION}

Отчёт: ${QUICK_DIR}/${next_num}-SUMMARY.md
Коммит: ${commit_hash}

---

Готовы к следующей задаче: /gsd:quick
```

</process>

<success_criteria>
- [ ] Валидация ROADMAP.md пройдена
- [ ] Пользователь предоставил описание задачи
- [ ] Slug сгенерирован (нижний регистр, дефисы, макс. 40 символов)
- [ ] Следующий номер вычислен (001, 002, 003...)
- [ ] Каталог создан по пути `.planning/quick/NNN-slug/`
- [ ] `${next_num}-PLAN.md` создан планировщиком
- [ ] `${next_num}-SUMMARY.md` создан исполнителем
- [ ] STATE.md обновлён строкой быстрой задачи
- [ ] Артефакты закоммичены
</success_criteria>
