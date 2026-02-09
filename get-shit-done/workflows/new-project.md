<purpose>
Инициализировать новый проект через единый поток: опрос, исследование (опционально), требования, дорожная карта. Это самый важный момент в любом проекте — глубокий опрос здесь означает лучшие планы, лучшее выполнение, лучшие результаты. Один рабочий процесс проведёт вас от идеи до готовности к планированию.
</purpose>

<required_reading>
Прочитай все файлы, указанные в execution_context вызывающей команды, перед началом работы.
</required_reading>

<auto_mode>
## Определение автоматического режима

Проверь наличие флага `--auto` в $ARGUMENTS.

**Если автоматический режим:**
- Пропустить предложение картирования браунфилда (считать гринфилд)
- Пропустить глубокий опрос (извлечь контекст из предоставленного документа)
- Вопросы конфигурации по-прежнему обязательны (Шаг 5)
- После конфигурации: запустить Шаги 6-9 автоматически с умными значениями по умолчанию:
  - Исследование: Всегда да
  - Требования: Включить все базовые + фичи из предоставленного документа
  - Утверждение требований: Автоутверждение
  - Утверждение дорожной карты: Автоутверждение

**Требование к документу:**
Автоматический режим требует документ с идеей через @ ссылку (например, `/gsd:new-project --auto @prd.md`). Если документ не предоставлен, ошибка:

```
Ошибка: --auto требует документ с идеей через @ ссылку.

Использование: /gsd:new-project --auto @ваша-идея.md

Документ должен описывать что вы хотите построить.
```
</auto_mode>

<process>

## 1. Настройка

**ОБЯЗАТЕЛЬНЫЙ ПЕРВЫЙ ШАГ — Выполни эти проверки ПЕРЕД любым взаимодействием с пользователем:**

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init new-project)
```

Разбери JSON: `researcher_model`, `synthesizer_model`, `roadmapper_model`, `commit_docs`, `project_exists`, `has_codebase_map`, `planning_exists`, `has_existing_code`, `has_package_file`, `is_brownfield`, `needs_codebase_map`, `has_git`.

**Если `project_exists` — true:** Ошибка — проект уже инициализирован. Используйте `/gsd:progress`.

**Если `has_git` — false:** Инициализировать git:
```bash
git init
```

## 2. Предложение для браунфилда

**Если автоматический режим:** Перейти к Шагу 4 (считать гринфилд, синтезировать PROJECT.md из предоставленного документа).

**Если `needs_codebase_map` — true** (из init — обнаружен существующий код, но нет карты кодовой базы):

Используй AskUserQuestion:
- header: "Существующий код"
- question: "Я обнаружил существующий код в этой директории. Хотите сначала картировать кодовую базу?"
- options:
  - "Картировать кодовую базу" — Запустить /gsd:map-codebase для понимания существующей архитектуры (Рекомендуется)
  - "Пропустить картирование" — Перейти к инициализации проекта

**Если "Картировать кодовую базу":**
```
Запустите `/gsd:map-codebase` сначала, затем вернитесь к `/gsd:new-project`
```
Выход из команды.

**Если "Пропустить картирование" ИЛИ `needs_codebase_map` — false:** Продолжить к Шагу 3.

## 3. Глубокий опрос

**Если автоматический режим:** Пропустить. Извлечь контекст проекта из предоставленного документа и перейти к Шагу 4.

**Показать баннер этапа:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► ОПРОС
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Начать разговор:**

Спроси напрямую (свободная форма, НЕ AskUserQuestion):

"Что вы хотите построить?"

Дождись ответа. Это даст контекст для умных уточняющих вопросов.

**Следуй за нитью:**

На основе ответа задавай уточняющие вопросы, которые копают глубже. Используй AskUserQuestion с вариантами, которые исследуют сказанное — интерпретации, уточнения, конкретные примеры.

Продолжай следовать за нитями. Каждый ответ открывает новые нити. Спрашивай о:
- Что их вдохновляет
- Какая проблема это вызвала
- Что они имеют в виду под расплывчатыми терминами
- Как это будет выглядеть на практике
- Что уже решено

Обращайся к `questioning.md` за техниками:
- Бросай вызов расплывчатости
- Делай абстрактное конкретным
- Выявляй допущения
- Находи границы
- Раскрывай мотивацию

**Проверка контекста (в фоне, не вслух):**

По ходу мысленно проверяй чеклист контекста из `questioning.md`. Если есть пробелы, вплетай вопросы естественно. Не переключайся внезапно на режим чеклиста.

**Гейт решения:**

Когда можешь написать понятный PROJECT.md, используй AskUserQuestion:

- header: "Готово?"
- question: "Кажется, я понимаю что вам нужно. Готовы создать PROJECT.md?"
- options:
  - "Создать PROJECT.md" — Двигаемся дальше
  - "Продолжить обсуждение" — Хочу рассказать больше / спроси ещё

Если "Продолжить обсуждение" — спроси что хотят добавить, или найди пробелы и исследуй естественно.

Цикл пока не выбрано "Создать PROJECT.md".

## 4. Написание PROJECT.md

**Если автоматический режим:** Синтезировать из предоставленного документа. Гейт "Готово?" не показывался — переходи сразу к коммиту.

Синтезируй весь контекст в `.planning/PROJECT.md` используя шаблон из `templates/project.md`.

**Для гринфилд-проектов:**

Инициализируй требования как гипотезы:

```markdown
## Требования

### Подтверждённые

(Пока нет — выпускайте для подтверждения)

### Активные

- [ ] [Требование 1]
- [ ] [Требование 2]
- [ ] [Требование 3]

### За пределами

- [Исключение 1] — [почему]
- [Исключение 2] — [почему]
```

Все Активные требования — гипотезы, пока не выпущены и подтверждены.

**Для браунфилд-проектов (карта кодовой базы существует):**

Определи Подтверждённые требования из существующего кода:

1. Прочитай `.planning/codebase/ARCHITECTURE.md` и `STACK.md`
2. Определи что кодовая база уже делает
3. Это становится начальным набором Подтверждённых

```markdown
## Требования

### Подтверждённые

- ✓ [Существующая возможность 1] — существующее
- ✓ [Существующая возможность 2] — существующее
- ✓ [Существующая возможность 3] — существующее

### Активные

- [ ] [Новое требование 1]
- [ ] [Новое требование 2]

### За пределами

- [Исключение 1] — [почему]
```

**Ключевые решения:**

Инициализируй решениями, принятыми во время опроса:

```markdown
## Ключевые решения

| Решение | Обоснование | Результат |
|---------|-------------|----------|
| [Выбор из опроса] | [Почему] | — В ожидании |
```

**Подвал с датой обновления:**

```markdown
---
*Последнее обновление: [дата] после инициализации*
```

Не сжимай. Зафиксируй всё собранное.

**Коммит PROJECT.md:**

```bash
mkdir -p .planning
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: инициализация проекта" --files .planning/PROJECT.md
```

## 5. Предпочтения рабочего процесса

**Раунд 1 — Основные настройки рабочего процесса (4 вопроса):**

```
questions: [
  {
    header: "Режим",
    question: "Как вы хотите работать?",
    multiSelect: false,
    options: [
      { label: "YOLO (Рекомендуется)", description: "Автоутверждение, просто выполнять" },
      { label: "Интерактивный", description: "Подтверждать каждый шаг" }
    ]
  },
  {
    header: "Глубина",
    question: "Насколько тщательным должно быть планирование?",
    multiSelect: false,
    options: [
      { label: "Быстро", description: "Выпускай быстро (3-5 фаз, 1-3 плана каждая)" },
      { label: "Стандарт", description: "Баланс объёма и скорости (5-8 фаз, 3-5 планов каждая)" },
      { label: "Комплексно", description: "Тщательное покрытие (8-12 фаз, 5-10 планов каждая)" }
    ]
  },
  {
    header: "Выполнение",
    question: "Запускать планы параллельно?",
    multiSelect: false,
    options: [
      { label: "Параллельно (Рекомендуется)", description: "Независимые планы выполняются одновременно" },
      { label: "Последовательно", description: "Один план за раз" }
    ]
  },
  {
    header: "Git-отслеживание",
    question: "Коммитить документы планирования в git?",
    multiSelect: false,
    options: [
      { label: "Да (Рекомендуется)", description: "Документы планирования отслеживаются в контроле версий" },
      { label: "Нет", description: "Оставить .planning/ только локально (добавить в .gitignore)" }
    ]
  }
]
```

**Раунд 2 — Агенты рабочего процесса:**

Эти запускают дополнительных агентов во время планирования/выполнения. Добавляют токены и время, но улучшают качество.

| Агент | Когда запускается | Что делает |
|-------|-------------------|------------|
| **Исследователь** | Перед планированием каждой фазы | Исследует домен, находит паттерны, выявляет подводные камни |
| **Проверщик планов** | После создания плана | Проверяет что план действительно достигает цели фазы |
| **Верификатор** | После выполнения фазы | Подтверждает что обязательные элементы выполнены |

Все рекомендуются для важных проектов. Пропускайте для быстрых экспериментов.

```
questions: [
  {
    header: "Исследование",
    question: "Исследовать перед планированием каждой фазы? (добавляет токены/время)",
    multiSelect: false,
    options: [
      { label: "Да (Рекомендуется)", description: "Исследовать домен, найти паттерны, выявить подводные камни" },
      { label: "Нет", description: "Планировать напрямую из требований" }
    ]
  },
  {
    header: "Проверка планов",
    question: "Проверять что планы достигают целей? (добавляет токены/время)",
    multiSelect: false,
    options: [
      { label: "Да (Рекомендуется)", description: "Ловить пробелы до начала выполнения" },
      { label: "Нет", description: "Выполнять планы без проверки" }
    ]
  },
  {
    header: "Верификатор",
    question: "Проверять что работа соответствует требованиям после каждой фазы? (добавляет токены/время)",
    multiSelect: false,
    options: [
      { label: "Да (Рекомендуется)", description: "Подтвердить что результаты соответствуют целям фазы" },
      { label: "Нет", description: "Довериться выполнению, пропустить проверку" }
    ]
  },
  {
    header: "Профиль модели",
    question: "Какие AI-модели для агентов планирования?",
    multiSelect: false,
    options: [
      { label: "Баланс (Рекомендуется)", description: "Sonnet для большинства агентов — хорошее соотношение качества/стоимости" },
      { label: "Качество", description: "Opus для исследования/дорожной карты — дороже, глубже анализ" },
      { label: "Бюджет", description: "Haiku где возможно — быстрее, дешевле" }
    ]
  }
]
```

Создай `.planning/config.json` со всеми настройками:

```json
{
  "mode": "yolo|interactive",
  "depth": "quick|standard|comprehensive",
  "parallelization": true|false,
  "commit_docs": true|false,
  "model_profile": "quality|balanced|budget",
  "workflow": {
    "research": true|false,
    "plan_check": true|false,
    "verifier": true|false
  }
}
```

**Если commit_docs = Нет:**
- Установить `commit_docs: false` в config.json
- Добавить `.planning/` в `.gitignore` (создать при необходимости)

**Если commit_docs = Да:**
- Никаких дополнительных записей в gitignore не нужно

**Коммит config.json:**

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "chore: добавить конфигурацию проекта" --files .planning/config.json
```

**Примечание:** Запустите `/gsd:settings` в любое время для обновления этих предпочтений.

## 5.5. Определение профиля модели

Используй модели из init: `researcher_model`, `synthesizer_model`, `roadmapper_model`.

## 6. Решение об исследовании

**Если автоматический режим:** По умолчанию "Исследовать сначала" без запроса.

Используй AskUserQuestion:
- header: "Исследование"
- question: "Исследовать экосистему домена перед определением требований?"
- options:
  - "Исследовать сначала (Рекомендуется)" — Обнаружить стандартные стеки, ожидаемые функции, паттерны архитектуры
  - "Пропустить исследование" — Я хорошо знаю этот домен, перейти сразу к требованиям

**Если "Исследовать сначала":**

Показать баннер этапа:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► ИССЛЕДОВАНИЕ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Исследую экосистему [домен]...
```

Создать директорию исследования:
```bash
mkdir -p .planning/research
```

**Определить контекст вехи:**

Проверь это гринфилд или последующая веха:
- Если нет "Подтверждённых" требований в PROJECT.md → Гринфилд (строим с нуля)
- Если "Подтверждённые" требования существуют → Последующая веха (добавляем к существующему)

Показать индикатор запуска:
```
◆ Запускаю 4 исследователей параллельно...
  → Исследование стека
  → Исследование функций
  → Исследование архитектуры
  → Исследование подводных камней
```

Запусти 4 параллельных агента gsd-project-researcher с богатым контекстом:

```
Task(prompt="First, read ~/.claude/agents/gsd-project-researcher.md for your role and instructions.

<research_type>
Project Research — Stack dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: Research the standard stack for building [domain] from scratch.
Subsequent: Research what's needed to add [target features] to an existing [domain] app. Don't re-research the existing system.
</milestone_context>

<question>
What's the standard 2025 stack for [domain]?
</question>

<project_context>
[PROJECT.md summary - core value, constraints, what they're building]
</project_context>

<downstream_consumer>
Your STACK.md feeds into roadmap creation. Be prescriptive:
- Specific libraries with versions
- Clear rationale for each choice
- What NOT to use and why
</downstream_consumer>

<quality_gate>
- [ ] Versions are current (verify with Context7/official docs, not training data)
- [ ] Rationale explains WHY, not just WHAT
- [ ] Confidence levels assigned to each recommendation
</quality_gate>

<o>
Write to: .planning/research/STACK.md
Use template: ~/.claude/get-shit-done/templates/research-project/STACK.md
</o>
", subagent_type="general-purpose", model="{researcher_model}", description="Исследование стека")

Task(prompt="First, read ~/.claude/agents/gsd-project-researcher.md for your role and instructions.

<research_type>
Project Research — Features dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: What features do [domain] products have? What's table stakes vs differentiating?
Subsequent: How do [target features] typically work? What's expected behavior?
</milestone_context>

<question>
What features do [domain] products have? What's table stakes vs differentiating?
</question>

<project_context>
[PROJECT.md summary]
</project_context>

<downstream_consumer>
Your FEATURES.md feeds into requirements definition. Categorize clearly:
- Table stakes (must have or users leave)
- Differentiators (competitive advantage)
- Anti-features (things to deliberately NOT build)
</downstream_consumer>

<quality_gate>
- [ ] Categories are clear (table stakes vs differentiators vs anti-features)
- [ ] Complexity noted for each feature
- [ ] Dependencies between features identified
</quality_gate>

<o>
Write to: .planning/research/FEATURES.md
Use template: ~/.claude/get-shit-done/templates/research-project/FEATURES.md
</o>
", subagent_type="general-purpose", model="{researcher_model}", description="Исследование функций")

Task(prompt="First, read ~/.claude/agents/gsd-project-researcher.md for your role and instructions.

<research_type>
Project Research — Architecture dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: How are [domain] systems typically structured? What are major components?
Subsequent: How do [target features] integrate with existing [domain] architecture?
</milestone_context>

<question>
How are [domain] systems typically structured? What are major components?
</question>

<project_context>
[PROJECT.md summary]
</project_context>

<downstream_consumer>
Your ARCHITECTURE.md informs phase structure in roadmap. Include:
- Component boundaries (what talks to what)
- Data flow (how information moves)
- Suggested build order (dependencies between components)
</downstream_consumer>

<quality_gate>
- [ ] Components clearly defined with boundaries
- [ ] Data flow direction explicit
- [ ] Build order implications noted
</quality_gate>

<o>
Write to: .planning/research/ARCHITECTURE.md
Use template: ~/.claude/get-shit-done/templates/research-project/ARCHITECTURE.md
</o>
", subagent_type="general-purpose", model="{researcher_model}", description="Исследование архитектуры")

Task(prompt="First, read ~/.claude/agents/gsd-project-researcher.md for your role and instructions.

<research_type>
Project Research — Pitfalls dimension for [domain].
</research_type>

<milestone_context>
[greenfield OR subsequent]

Greenfield: What do [domain] projects commonly get wrong? Critical mistakes?
Subsequent: What are common mistakes when adding [target features] to [domain]?
</milestone_context>

<question>
What do [domain] projects commonly get wrong? Critical mistakes?
</question>

<project_context>
[PROJECT.md summary]
</project_context>

<downstream_consumer>
Your PITFALLS.md prevents mistakes in roadmap/planning. For each pitfall:
- Warning signs (how to detect early)
- Prevention strategy (how to avoid)
- Which phase should address it
</downstream_consumer>

<quality_gate>
- [ ] Pitfalls are specific to this domain (not generic advice)
- [ ] Prevention strategies are actionable
- [ ] Phase mapping included where relevant
</quality_gate>

<o>
Write to: .planning/research/PITFALLS.md
Use template: ~/.claude/get-shit-done/templates/research-project/PITFALLS.md
</o>
", subagent_type="general-purpose", model="{researcher_model}", description="Исследование подводных камней")
```

После завершения всех 4 агентов запусти синтезатор для создания SUMMARY.md:

```
Task(prompt="
<task>
Synthesize research outputs into SUMMARY.md.
</task>

<research_files>
Read these files:
- .planning/research/STACK.md
- .planning/research/FEATURES.md
- .planning/research/ARCHITECTURE.md
- .planning/research/PITFALLS.md
</research_files>

<o>
Write to: .planning/research/SUMMARY.md
Use template: ~/.claude/get-shit-done/templates/research-project/SUMMARY.md
Commit after writing.
</o>
", subagent_type="gsd-research-synthesizer", model="{synthesizer_model}", description="Синтез исследования")
```

Показать баннер завершения исследования и ключевые находки:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► ИССЛЕДОВАНИЕ ЗАВЕРШЕНО ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Ключевые находки

**Стек:** [из SUMMARY.md]
**Базовые функции:** [из SUMMARY.md]
**Осторожно с:** [из SUMMARY.md]

Файлы: `.planning/research/`
```

**Если "Пропустить исследование":** Продолжить к Шагу 7.

## 7. Определение требований

Показать баннер этапа:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► ОПРЕДЕЛЕНИЕ ТРЕБОВАНИЙ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Загрузка контекста:**

Прочитай PROJECT.md и извлеки:
- Основную ценность (ТА ЕДИНСТВЕННАЯ вещь, которая должна работать)
- Заявленные ограничения (бюджет, сроки, технические ограничения)
- Любые явные границы объёма

**Если исследование существует:** Прочитай research/FEATURES.md и извлеки категории функций.

**Если автоматический режим:**
- Автоматически включить все базовые функции (пользователи их ожидают)
- Включить функции, явно упомянутые в предоставленном документе
- Автоматически отложить дифференциаторы, не упомянутые в документе
- Пропустить циклы AskUserQuestion по категориям
- Пропустить вопрос "Дополнения?"
- Пропустить гейт утверждения требований
- Сгенерировать REQUIREMENTS.md и закоммитить напрямую

**Представить функции по категориям (только интерактивный режим):**

```
Вот функции для [домен]:

## Аутентификация
**Базовые:**
- Регистрация по email/паролю
- Подтверждение email
- Сброс пароля
- Управление сессиями

**Дифференциаторы:**
- Вход по магической ссылке
- OAuth (Google, GitHub)
- 2FA

**Заметки исследования:** [если есть]

---

## [Следующая категория]
...
```

**Если нет исследования:** Собрать требования через разговор.

Спроси: "Что основные вещи, которые пользователи должны уметь делать?"

Для каждой упомянутой возможности:
- Задай уточняющие вопросы для конкретизации
- Исследуй связанные возможности
- Сгруппируй по категориям

**Разграничение каждой категории:**

Для каждой категории используй AskUserQuestion:

- header: "[Название категории]"
- question: "Какие функции [категории] входят в v1?"
- multiSelect: true
- options:
  - "[Функция 1]" — [краткое описание]
  - "[Функция 2]" — [краткое описание]
  - "[Функция 3]" — [краткое описание]
  - "Ничего для v1" — Отложить всю категорию

Отслеживай ответы:
- Выбранные функции → требования v1
- Невыбранные базовые → v2 (пользователи их ожидают)
- Невыбранные дифференциаторы → за пределами

**Поиск пробелов:**

Используй AskUserQuestion:
- header: "Дополнения"
- question: "Есть требования, которые исследование пропустило? (Функции специфичные для вашего видения)"
- options:
  - "Нет, исследование покрыло всё" — Продолжить
  - "Да, хочу добавить" — Записать дополнения

**Проверка основной ценности:**

Сопоставь требования с Основной ценностью из PROJECT.md. Если обнаружены пробелы, укажи на них.

**Генерация REQUIREMENTS.md:**

Создай `.planning/REQUIREMENTS.md` с:
- Требования v1 сгруппированные по категориям (чекбоксы, REQ-ID)
- Требования v2 (отложенные)
- За пределами (явные исключения с обоснованием)
- Секция прослеживаемости (пустая, заполняется дорожной картой)

**Формат REQ-ID:** `[КАТЕГОРИЯ]-[НОМЕР]` (AUTH-01, CONTENT-02)

**Критерии качества требований:**

Хорошие требования:
- **Конкретные и тестируемые:** "Пользователь может сбросить пароль через ссылку в email" (не "Обработка сброса пароля")
- **Ориентированные на пользователя:** "Пользователь может X" (не "Система делает Y")
- **Атомарные:** Одна возможность на требование (не "Пользователь может войти и управлять профилем")
- **Независимые:** Минимальные зависимости от других требований

Отклоняй расплывчатые требования. Добивайся конкретности:
- "Обработка аутентификации" → "Пользователь может войти по email/паролю и оставаться залогиненным между сессиями"
- "Поддержка шаринга" → "Пользователь может поделиться постом через ссылку, которая открывается в браузере получателя"

**Представить полный список требований (только интерактивный режим):**

Показать каждое требование (не количества) для подтверждения пользователем:

```
## Требования v1

### Аутентификация
- [ ] **AUTH-01**: Пользователь может создать аккаунт по email/паролю
- [ ] **AUTH-02**: Пользователь может войти и оставаться залогиненным между сессиями
- [ ] **AUTH-03**: Пользователь может выйти с любой страницы

### Контент
- [ ] **CONT-01**: Пользователь может создавать посты с текстом
- [ ] **CONT-02**: Пользователь может редактировать свои посты

[... полный список ...]

---

Это описывает то, что вы строите? (да / скорректировать)
```

Если "скорректировать": Вернуться к разграничению.

**Коммит требований:**

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: определить требования v1" --files .planning/REQUIREMENTS.md
```

## 8. Создание дорожной карты

Показать баннер этапа:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► СОЗДАНИЕ ДОРОЖНОЙ КАРТЫ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Запускаю составителя дорожной карты...
```

Запусти агента gsd-roadmapper с контекстом:

```
Task(prompt="
<planning_context>

**Проект:**
@.planning/PROJECT.md

**Требования:**
@.planning/REQUIREMENTS.md

**Исследование (если существует):**
@.planning/research/SUMMARY.md

**Конфигурация:**
@.planning/config.json

</planning_context>

<instructions>
Create roadmap:
1. Derive phases from requirements (don't impose structure)
2. Map every v1 requirement to exactly one phase
3. Derive 2-5 success criteria per phase (observable user behaviors)
4. Validate 100% coverage
5. Write files immediately (ROADMAP.md, STATE.md, update REQUIREMENTS.md traceability)
6. Return ROADMAP CREATED with summary

Write files first, then return. This ensures artifacts persist even if context is lost.
</instructions>
", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Создание дорожной карты")
```

**Обработка возврата составителя дорожной карты:**

**Если `## ROADMAP BLOCKED`:**
- Представить информацию о блокере
- Работать с пользователем для решения
- Перезапустить когда решено

**Если `## ROADMAP CREATED`:**

Прочитай созданный ROADMAP.md и представь его красиво встроенным:

```
---

## Предложенная дорожная карта

**[N] фаз** | **[X] требований привязано** | Все требования v1 покрыты ✓

| # | Фаза | Цель | Требования | Критерии успеха |
|---|------|------|------------|----------------|
| 1 | [Название] | [Цель] | [REQ-ID] | [кол-во] |
| 2 | [Название] | [Цель] | [REQ-ID] | [кол-во] |
| 3 | [Название] | [Цель] | [REQ-ID] | [кол-во] |
...

### Детали фаз

**Фаза 1: [Название]**
Цель: [цель]
Требования: [REQ-ID]
Критерии успеха:
1. [критерий]
2. [критерий]
3. [критерий]

**Фаза 2: [Название]**
Цель: [цель]
Требования: [REQ-ID]
Критерии успеха:
1. [критерий]
2. [критерий]

[... продолжить для всех фаз ...]

---
```

**Если автоматический режим:** Пропустить гейт утверждения — автоутвердить и закоммитить напрямую.

**КРИТИЧЕСКИ: Спроси утверждение перед коммитом (только интерактивный режим):**

Используй AskUserQuestion:
- header: "Дорожная карта"
- question: "Эта структура дорожной карты вам подходит?"
- options:
  - "Утвердить" — Закоммитить и продолжить
  - "Скорректировать фазы" — Расскажите что изменить
  - "Просмотреть полный файл" — Показать сырой ROADMAP.md

**Если "Утвердить":** Продолжить к коммиту.

**Если "Скорректировать фазы":**
- Получить заметки пользователя о корректировке
- Перезапустить составителя дорожной карты с контекстом ревизии:
  ```
  Task(prompt="
  <revision>
  User feedback on roadmap:
  [user's notes]

  Current ROADMAP.md: @.planning/ROADMAP.md

  Update the roadmap based on feedback. Edit files in place.
  Return ROADMAP REVISED with changes made.
  </revision>
  ", subagent_type="gsd-roadmapper", model="{roadmapper_model}", description="Ревизия дорожной карты")
  ```
- Представить пересмотренную дорожную карту
- Цикл пока пользователь не утвердит

**Если "Просмотреть полный файл":** Показать `cat .planning/ROADMAP.md`, затем переспросить.

**Коммит дорожной карты (после утверждения или в автоматическом режиме):**

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: создать дорожную карту ([N] фаз)" --files .planning/ROADMAP.md .planning/STATE.md .planning/REQUIREMENTS.md
```

## 9. Готово

Представить завершение со следующими шагами:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ДД ► ПРОЕКТ ИНИЦИАЛИЗИРОВАН ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**[Название проекта]**

| Артефакт       | Расположение                |
|----------------|-----------------------------|
| Проект         | `.planning/PROJECT.md`      |
| Конфигурация   | `.planning/config.json`     |
| Исследование   | `.planning/research/`       |
| Требования     | `.planning/REQUIREMENTS.md` |
| Дорожная карта | `.planning/ROADMAP.md`      |

**[N] фаз** | **[X] требований** | Готов к разработке ✓

───────────────────────────────────────────────────────────────

## ▶ Далее

**Фаза 1: [Название фазы]** — [Цель из ROADMAP.md]

/gsd:discuss-phase 1 — собрать контекст и уточнить подход

<sub>/clear сначала → свежее окно контекста</sub>

---

**Также доступно:**
- /gsd:plan-phase 1 — пропустить обсуждение, планировать напрямую

───────────────────────────────────────────────────────────────
```

</process>

<o>

- `.planning/PROJECT.md`
- `.planning/config.json`
- `.planning/research/` (если исследование выбрано)
  - `STACK.md`
  - `FEATURES.md`
  - `ARCHITECTURE.md`
  - `PITFALLS.md`
  - `SUMMARY.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`

</o>

<success_criteria>

- [ ] Директория .planning/ создана
- [ ] Git-репозиторий инициализирован
- [ ] Определение браунфилда выполнено
- [ ] Глубокий опрос завершён (нити прослежены, не поспешный)
- [ ] PROJECT.md фиксирует полный контекст → **закоммичен**
- [ ] config.json содержит режим работы, глубину, параллелизацию → **закоммичен**
- [ ] Исследование завершено (если выбрано) — 4 параллельных агента запущены → **закоммичено**
- [ ] Требования собраны (из исследования или разговора)
- [ ] Пользователь разграничил каждую категорию (v1/v2/за пределами)
- [ ] REQUIREMENTS.md создан с REQ-ID → **закоммичен**
- [ ] gsd-roadmapper запущен с контекстом
- [ ] Файлы дорожной карты записаны немедленно (не черновик)
- [ ] Обратная связь пользователя учтена (если была)
- [ ] ROADMAP.md создан с фазами, привязками требований, критериями успеха
- [ ] STATE.md инициализирован
- [ ] Прослеживаемость REQUIREMENTS.md обновлена
- [ ] Пользователь знает что следующий шаг — `/gsd:discuss-phase 1`

**Атомарные коммиты:** Каждый этап коммитит свои артефакты немедленно. Если контекст потерян, артефакты сохраняются.

</success_criteria>
