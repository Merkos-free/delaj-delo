<purpose>
Оркестрировать параллельных агентов-картографов кодовой базы для анализа кода и создания структурированных документов в .planning/codebase/

Каждый агент имеет свежий контекст, исследует конкретную область фокуса и **записывает документы напрямую**. Оркестратор получает только подтверждение + количество строк, затем пишет резюме.

Вывод: папка .planning/codebase/ с 7 структурированными документами о состоянии кодовой базы.
</purpose>

<philosophy>
**Почему выделенные агенты-картографы:**
- Свежий контекст на домен (без загрязнения токенами)
- Агенты пишут документы напрямую (без передачи контекста обратно оркестратору)
- Оркестратор только резюмирует что было создано (минимальное использование контекста)
- Быстрее выполнение (агенты работают параллельно)

**Качество документа важнее длины:**
Включайте достаточно деталей для полезности как справочника. Приоритизируйте практические примеры (особенно паттерны кода) над произвольной краткостью.

**Всегда включайте пути к файлам:**
Документы — справочный материал для Claude при планировании/выполнении. Всегда включайте реальные пути файлов в обратных кавычках: `src/services/user.ts`.
</philosophy>

<process>

<step name="init_context" priority="first">
Загрузить контекст картирования кодовой базы:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init map-codebase)
```

Извлечь из JSON инициализации: `mapper_model`, `commit_docs`, `codebase_dir`, `existing_maps`, `has_maps`, `codebase_dir_exists`.
</step>

<step name="check_existing">
Проверить существование .planning/codebase/ используя `has_maps` из контекста init.

Если `codebase_dir_exists` — true:
```bash
ls -la .planning/codebase/
```

**Если существует:**

```
.planning/codebase/ уже существует с этими документами:
[Список найденных файлов]

Что дальше?
1. Обновить — Удалить существующие и перекартировать кодовую базу
2. Дополнить — Оставить существующие, обновить только конкретные документы
3. Пропустить — Использовать существующую карту как есть
```

Дождитесь ответа пользователя.

Если "Обновить": Удалить .planning/codebase/, перейти к create_structure
Если "Дополнить": Спросить какие документы обновить, перейти к spawn_agents (фильтрованно)
Если "Пропустить": Выход из рабочего процесса

**Если не существует:**
Перейти к create_structure.
</step>

<step name="create_structure">
Создать каталог .planning/codebase/:

```bash
mkdir -p .planning/codebase
```

**Ожидаемые выходные файлы:**
- STACK.md (от техкартографа)
- INTEGRATIONS.md (от техкартографа)
- ARCHITECTURE.md (от архкартографа)
- STRUCTURE.md (от архкартографа)
- CONVENTIONS.md (от картографа качества)
- TESTING.md (от картографа качества)
- CONCERNS.md (от картографа проблем)

Перейти к spawn_agents.
</step>

<step name="spawn_agents">
Запустить 4 параллельных агента gsd-codebase-mapper.

Использовать инструмент Task с `subagent_type="gsd-codebase-mapper"`, `model="{mapper_model}"` и `run_in_background=true` для параллельного выполнения.

**КРИТИЧЕСКИ:** Используйте выделенного агента `gsd-codebase-mapper`, НЕ `Explore`. Агент-картограф пишет документы напрямую.

**Агент 1: Фокус на технологиях**

Параметры инструмента Task:
```
subagent_type: "gsd-codebase-mapper"
model: "{mapper_model}"
run_in_background: true
description: "Картирование техстека кодовой базы"
```

Промпт:
```
Focus: tech

Analyze this codebase for technology stack and external integrations.

Write these documents to .planning/codebase/:
- STACK.md - Languages, runtime, frameworks, dependencies, configuration
- INTEGRATIONS.md - External APIs, databases, auth providers, webhooks

Explore thoroughly. Write documents directly using templates. Return confirmation only.
```

**Агент 2: Фокус на архитектуре**

Параметры инструмента Task:
```
subagent_type: "gsd-codebase-mapper"
model: "{mapper_model}"
run_in_background: true
description: "Картирование архитектуры кодовой базы"
```

Промпт:
```
Focus: arch

Analyze this codebase architecture and directory structure.

Write these documents to .planning/codebase/:
- ARCHITECTURE.md - Pattern, layers, data flow, abstractions, entry points
- STRUCTURE.md - Directory layout, key locations, naming conventions

Explore thoroughly. Write documents directly using templates. Return confirmation only.
```

**Агент 3: Фокус на качестве**

Параметры инструмента Task:
```
subagent_type: "gsd-codebase-mapper"
model: "{mapper_model}"
run_in_background: true
description: "Картирование конвенций кодовой базы"
```

Промпт:
```
Focus: quality

Analyze this codebase for coding conventions and testing patterns.

Write these documents to .planning/codebase/:
- CONVENTIONS.md - Code style, naming, patterns, error handling
- TESTING.md - Framework, structure, mocking, coverage

Explore thoroughly. Write documents directly using templates. Return confirmation only.
```

**Агент 4: Фокус на проблемах**

Параметры инструмента Task:
```
subagent_type: "gsd-codebase-mapper"
model: "{mapper_model}"
run_in_background: true
description: "Картирование проблем кодовой базы"
```

Промпт:
```
Focus: concerns

Analyze this codebase for technical debt, known issues, and areas of concern.

Write this document to .planning/codebase/:
- CONCERNS.md - Tech debt, bugs, security, performance, fragile areas

Explore thoroughly. Write document directly using template. Return confirmation only.
```

Перейти к collect_confirmations.
</step>

<step name="collect_confirmations">
Дождаться завершения всех 4 агентов.

Прочитать вывод каждого агента для сбора подтверждений.

**Ожидаемый формат подтверждения от каждого агента:**
```
## Картирование завершено

**Фокус:** {focus}
**Записанные документы:**
- `.planning/codebase/{DOC1}.md` ({N} строк)
- `.planning/codebase/{DOC2}.md` ({N} строк)

Готов для резюме оркестратора.
```

**Что вы получаете:** Только пути файлов и количество строк. НЕ содержимое документов.

Если какой-либо агент провалился, отметить ошибку и продолжить с успешными документами.

Перейти к verify_output.
</step>

<step name="verify_output">
Проверить что все документы созданы успешно:

```bash
ls -la .planning/codebase/
wc -l .planning/codebase/*.md
```

**Чеклист проверки:**
- Все 7 документов существуют
- Нет пустых документов (каждый должен иметь >20 строк)

Если какие-то документы отсутствуют или пусты, отметить какие агенты могли провалиться.

Перейти к scan_for_secrets.
</step>

<step name="scan_for_secrets">
**КРИТИЧЕСКАЯ ПРОВЕРКА БЕЗОПАСНОСТИ:** Просканировать выходные файлы на случайно утечённые секреты перед коммитом.

Запустить обнаружение паттернов секретов:

```bash
# Проверить паттерны API ключей в сгенерированных документах
grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.)' .planning/codebase/*.md 2>/dev/null && SECRETS_FOUND=true || SECRETS_FOUND=false
```

**Если SECRETS_FOUND=true:**

```
⚠️  ПРЕДУПРЕЖДЕНИЕ БЕЗОПАСНОСТИ: Обнаружены потенциальные секреты в документах кодовой базы!

Найдены паттерны, похожие на API ключи или токены в:
[показать вывод grep]

Это раскроет учётные данные при коммите.

**Требуется действие:**
1. Просмотрите отмеченный контент выше
2. Если это реальные секреты, они должны быть удалены перед коммитом
3. Рассмотрите добавление чувствительных файлов в разрешения "Запретить" Claude Code

Пауза перед коммитом. Ответьте "безопасно продолжить" если отмеченный контент не является чувствительным, или отредактируйте файлы сначала.
```

Дождитесь подтверждения пользователя перед переходом к commit_codebase_map.

**Если SECRETS_FOUND=false:**

Перейти к commit_codebase_map.
</step>

<step name="commit_codebase_map">
Закоммитить карту кодовой базы:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: map existing codebase" --files .planning/codebase/*.md
```

Перейти к offer_next.
</step>

<step name="offer_next">
Представить резюме завершения и следующие шаги.

**Получить количество строк:**
```bash
wc -l .planning/codebase/*.md
```

**Формат вывода:**

```
Картирование кодовой базы завершено.

Создано .planning/codebase/:
- STACK.md ([N] строк) - Технологии и зависимости
- ARCHITECTURE.md ([N] строк) - Дизайн системы и паттерны
- STRUCTURE.md ([N] строк) - Структура каталогов и организация
- CONVENTIONS.md ([N] строк) - Стиль кода и паттерны
- TESTING.md ([N] строк) - Структура тестов и практики
- INTEGRATIONS.md ([N] строк) - Внешние сервисы и API
- CONCERNS.md ([N] строк) - Техдолг и проблемы


---

## ▶ Далее

**Инициализация проекта** — использовать контекст кодовой базы для планирования

`/gsd:new-project`

<sub>`/clear` сначала → свежее контекстное окно</sub>

---

**Также доступно:**
- Перезапустить картирование: `/gsd:map-codebase`
- Просмотреть конкретный файл: `cat .planning/codebase/STACK.md`
- Отредактировать любой документ перед продолжением

---
```

Конец рабочего процесса.
</step>

</process>

<success_criteria>
- Каталог .planning/codebase/ создан
- 4 параллельных агента gsd-codebase-mapper запущены с run_in_background=true
- Агенты пишут документы напрямую (оркестратор не получает содержимое документов)
- Вывод агентов прочитан для сбора подтверждений
- Все 7 документов кодовой базы существуют
- Чёткое резюме завершения с количеством строк
- Пользователю предложены чёткие следующие шаги в стиле ДД
</success_criteria>