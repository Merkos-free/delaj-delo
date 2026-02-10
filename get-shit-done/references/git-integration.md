<overview>
Интеграция с Git для фреймворка ДД.
</overview>

<core_principle>

**Коммитить результаты, а не процесс.**

Git-лог должен читаться как журнал изменений того, что выпущено, а не как дневник планировочной деятельности.
</core_principle>

<commit_points>

| Событие                   | Коммит? | Почему                                              |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP созданы | ДА     | Инициализация проекта                           |
| PLAN.md создан         | НЕТ      | Промежуточный — коммитится с завершением плана       |
| RESEARCH.md создан     | НЕТ      | Промежуточный                                     |
| DISCOVERY.md создан    | НЕТ      | Промежуточный                                     |
| **Задача завершена**      | ДА     | Атомарная единица работы (1 коммит на задачу)         |
| **План завершён**      | ДА     | Мета-коммит (SUMMARY + STATE + ROADMAP)     |
| Хендофф создан         | ДА     | WIP-состояние сохранено                              |

</commit_points>

<git_check>

```bash
[ -d .git ] && echo "GIT_EXISTS" || echo "NO_GIT"
```

Если NO_GIT: Запустить `git init` тихо. Проекты ДД всегда получают собственный репозиторий.
</git_check>

<commit_formats>

<format name="initialization">
## Инициализация проекта (brief + roadmap вместе)

```
docs: initialize [project-name] ([N] phases)

[Однострочное описание из PROJECT.md]

Phases:
1. [phase-name]: [goal]
2. [phase-name]: [goal]
3. [phase-name]: [goal]
```

Что коммитить:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: initialize [project-name] ([N] phases)" --files .planning/
```

</format>

<format name="task-completion">
## Завершение задачи (во время выполнения плана)

Каждая задача получает собственный коммит сразу после завершения.

```
{type}({phase}-{plan}): {task-name}

- [Ключевое изменение 1]
- [Ключевое изменение 2]
- [Ключевое изменение 3]
```

**Типы коммитов:**
- `feat` — Новая фича/функциональность
- `fix` — Исправление бага
- `test` — Только тесты (TDD КРАСНАЯ фаза)
- `refactor` — Очистка кода (TDD фаза РЕФАКТОРИНГА)
- `perf` — Улучшение производительности
- `chore` — Зависимости, конфиг, инструментарий

**Примеры:**

```bash
# Стандартная задача
git add src/api/auth.ts src/types/user.ts
git commit -m "feat(08-02): create user registration endpoint

- POST /auth/register валидирует email и пароль
- Проверяет дублирование пользователей
- Возвращает JWT-токен при успехе
"

# TDD задача — КРАСНАЯ фаза
git add src/__tests__/jwt.test.ts
git commit -m "test(07-02): add failing test for JWT generation

- Тест: токен содержит claim с ID пользователя
- Тест: токен истекает через 1 час
- Тест: верификация подписи
"

# TDD задача — ЗЕЛЁНАЯ фаза
git add src/utils/jwt.ts
git commit -m "feat(07-02): implement JWT generation

- Использует библиотеку jose для подписи
- Включает claims ID пользователя и срока действия
- Подписывает алгоритмом HS256
"
```

</format>

<format name="plan-completion">
## Завершение плана (после выполнения всех задач)

После коммита всех задач, один финальный мета-коммит фиксирует завершение плана.

```
docs({phase}-{plan}): complete [plan-name] plan

Задачи завершены: [N]/[N]
- [Название задачи 1]
- [Название задачи 2]
- [Название задачи 3]

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
```

Что коммитить:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs({phase}-{plan}): complete [plan-name] plan" --files .planning/phases/XX-name/{phase}-{plan}-PLAN.md .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md .planning/STATE.md .planning/ROADMAP.md
```

**Примечание:** Файлы кода НЕ включаются — они уже закоммичены по-задачно.

</format>

<format name="handoff">
## Хендофф (WIP)

```
wip: [phase-name] paused at task [X]/[Y]

Текущая: [название задачи]
[Если заблокировано:] Заблокировано: [причина]
```

Что коммитить:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "wip: [phase-name] paused at task [X]/[Y]" --files .planning/
```

</format>
</commit_formats>

<example_log>

**Старый подход (коммиты по планам):**
```
a7f2d1 feat(checkout): Stripe-оплата с верификацией вебхуков
3e9c4b feat(products): каталог с поиском, фильтрами и пагинацией
8a1b2c feat(auth): JWT с ротацией refresh через jose
5c3d7e feat(foundation): Next.js 15 + Prisma + Tailwind каркас
2f4a8d docs: initialize ecommerce-app (5 phases)
```

**Новый подход (коммиты по задачам):**
```
# Фаза 04 — Оформление заказа
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# Фаза 03 — Товары
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# Фаза 02 — Авторизация
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# Фаза 01 — Фундамент
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# Инициализация
5c6d7e docs: initialize ecommerce-app (5 phases)
```

Каждый план создаёт 2-4 коммита (задачи + метаданные). Чётко, гранулярно, bisectable.

</example_log>

<anti_patterns>

**По-прежнему НЕ коммитить (промежуточные артефакты):**
- Создание PLAN.md (коммитить с завершением плана)
- RESEARCH.md (промежуточный)
- DISCOVERY.md (промежуточный)
- Мелкие правки планирования
- "Исправлена опечатка в дорожной карте"

**Коммитить (результаты):**
- Завершение каждой задачи (feat/fix/test/refactor)
- Метаданные завершения плана (docs)
- Инициализация проекта (docs)

**Ключевой принцип:** Коммитить рабочий код и выпущенные результаты, а не процесс планирования.

</anti_patterns>

<commit_strategy_rationale>

## Почему коммиты по задачам?

**Контекстная инженерия для ИИ:**
- Git-история становится основным источником контекста для будущих сессий Claude
- `git log --grep="{phase}-{plan}"` показывает всю работу по плану
- `git diff <hash>^..<hash>` показывает точные изменения по задаче
- Меньше зависимости от парсинга SUMMARY.md = больше контекста для реальной работы

**Восстановление после сбоев:**
- Задача 1 закоммичена ✅, задача 2 упала ❌
- Claude в следующей сессии: видит задачу 1 выполненной, может повторить задачу 2
- Можно `git reset --hard` до последней успешной задачи

**Отладка:**
- `git bisect` находит точную проблемную задачу, а не просто проблемный план
- `git blame` связывает строку с конкретным контекстом задачи
- Каждый коммит можно откатить независимо

**Наблюдаемость:**
- Рабочий процесс «соло-разработчик + Claude» выигрывает от гранулярной атрибуции
- Атомарные коммиты — лучшая практика git
- «Шум коммитов» не важен когда потребитель — Claude, а не люди

</commit_strategy_rationale>
