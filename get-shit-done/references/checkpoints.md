<overview>
Планы выполняются автономно. Контрольные точки формализуют моменты взаимодействия, где необходима проверка или решения со стороны человека.

**Основной принцип:** Claude автоматизирует всё через CLI/API. Контрольные точки предназначены для проверки и принятия решений, а не для ручной работы.

**Золотые правила:**
1. **Если Claude может выполнить — Claude выполняет** — Никогда не просите пользователя выполнять CLI-команды, запускать серверы или сборку
2. **Claude настраивает среду проверки** — Запуск dev-серверов, заполнение баз данных, настройка переменных окружения
3. **Пользователь делает только то, что требует человеческого суждения** — Визуальные проверки, оценка UX, «выглядит ли это правильно?»
4. **Секреты от пользователя, автоматизация от Claude** — Запросить API-ключи, затем Claude использует их через CLI
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify (Самый частый — 90%)

**Когда:** Claude завершил автоматизированную работу, человек подтверждает корректность.

**Используется для:**
- Визуальных проверок UI (верстка, стили, адаптивность)
- Интерактивных потоков (прокликать визард, протестировать пользовательские сценарии)
- Функциональной верификации (фича работает как ожидалось)
- Качества воспроизведения аудио/видео
- Плавности анимаций
- Тестирования доступности

**Структура:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[Что Claude автоматизировал и развернул/собрал]</what-built>
  <how-to-verify>
    [Точные шаги для проверки — URL, команды, ожидаемое поведение]
  </how-to-verify>
  <resume-signal>[Как продолжить — «одобрено», «да» или описание проблем]</resume-signal>
</task>
```

**Пример: UI-компонент (показывает ключевой паттерн: Claude запускает сервер ДО контрольной точки)**
```xml
<task type="auto">
  <n>Собрать адаптивный макет дашборда</n>
  <files>src/components/Dashboard.tsx, src/app/dashboard/page.tsx</files>
  <action>Создать дашборд с боковой панелью, шапкой и областью контента. Использовать Tailwind адаптивные классы для мобильных.</action>
  <verify>npm run build завершается успешно, нет ошибок TypeScript</verify>
  <done>Компонент дашборда собирается без ошибок</done>
</task>

<task type="auto">
  <n>Запустить dev-сервер для верификации</n>
  <action>Выполнить `npm run dev` в фоне, дождаться сообщения «ready», зафиксировать порт</action>
  <verify>curl http://localhost:3000 возвращает 200</verify>
  <done>Dev-сервер запущен на http://localhost:3000</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Адаптивный макет дашборда — dev-сервер запущен на http://localhost:3000</what-built>
  <how-to-verify>
    Откройте http://localhost:3000/dashboard и проверьте:
    1. Десктоп (>1024px): Боковая панель слева, контент справа, шапка сверху
    2. Планшет (768px): Боковая панель сворачивается в гамбургер-меню
    3. Мобильный (375px): Одноколоночная верстка, нижняя навигация появляется
    4. Нет сдвигов верстки или горизонтальной прокрутки при любом размере
  </how-to-verify>
  <resume-signal>Напишите «одобрено» или опишите проблемы с версткой</resume-signal>
</task>
```

**Пример: Сборка Xcode**
```xml
<task type="auto">
  <n>Собрать macOS-приложение через Xcode</n>
  <files>App.xcodeproj, Sources/</files>
  <action>Выполнить `xcodebuild -project App.xcodeproj -scheme App build`. Проверить ошибки компиляции в выводе.</action>
  <verify>Вывод сборки содержит «BUILD SUCCEEDED», ошибок нет</verify>
  <done>Приложение успешно собрано</done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Собрано macOS-приложение в DerivedData/Build/Products/Debug/App.app</what-built>
  <how-to-verify>
    Откройте App.app и проверьте:
    - Приложение запускается без вылетов
    - Иконка в строке меню появляется
    - Окно настроек открывается корректно
    - Нет визуальных глитчей или проблем с версткой
  </how-to-verify>
  <resume-signal>Напишите «одобрено» или опишите проблемы</resume-signal>
</task>
```
</type>

<type name="decision">
## checkpoint:decision (9%)

**Когда:** Человек должен сделать выбор, влияющий на направление реализации.

**Используется для:**
- Выбора технологий (какой auth-провайдер, какая база данных)
- Архитектурных решений (монорепо vs отдельные репозитории)
- Дизайн-решений (цветовая схема, подход к верстке)
- Приоритизации функций (какой вариант строить)
- Решений по модели данных (структура схемы)

**Структура:**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>[Что решается]</decision>
  <context>[Почему это решение важно]</context>
  <options>
    <option id="option-a">
      <n>[Название варианта]</n>
      <pros>[Преимущества]</pros>
      <cons>[Компромиссы]</cons>
    </option>
    <option id="option-b">
      <n>[Название варианта]</n>
      <pros>[Преимущества]</pros>
      <cons>[Компромиссы]</cons>
    </option>
  </options>
  <resume-signal>[Как указать выбор]</resume-signal>
</task>
```

**Пример: Выбор Auth-провайдера**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Выбрать провайдер аутентификации</decision>
  <context>
    Приложению нужна аутентификация пользователей. Три надёжных варианта с разными компромиссами.
  </context>
  <options>
    <option id="supabase">
      <n>Supabase Auth</n>
      <pros>Встроен в используемую Supabase DB, щедрый бесплатный тариф, интеграция с row-level security</pros>
      <cons>Менее настраиваемый UI, привязка к экосистеме Supabase</cons>
    </option>
    <option id="clerk">
      <n>Clerk</n>
      <pros>Красивый готовый UI, лучший опыт разработчика, отличная документация</pros>
      <cons>Платный после 10k MAU, зависимость от вендора</cons>
    </option>
    <option id="nextauth">
      <n>NextAuth.js</n>
      <pros>Бесплатный, self-hosted, максимальный контроль, широко используемый</pros>
      <cons>Больше работы по настройке, вы управляете обновлениями безопасности, UI делается самостоятельно</cons>
    </option>
  </options>
  <resume-signal>Выберите: supabase, clerk или nextauth</resume-signal>
</task>
```

**Пример: Выбор базы данных**
```xml
<task type="checkpoint:decision" gate="blocking">
  <decision>Выбрать базу данных для пользовательских данных</decision>
  <context>
    Приложению нужно постоянное хранилище для пользователей, сессий и пользовательского контента.
    Ожидаемый масштаб: 10k пользователей, 1M записей в первый год.
  </context>
  <options>
    <option id="supabase">
      <n>Supabase (Postgres)</n>
      <pros>Полный SQL, щедрый бесплатный тариф, встроенная auth, real-time подписки</pros>
      <cons>Зависимость от вендора для real-time функций, менее гибкий чем чистый Postgres</cons>
    </option>
    <option id="planetscale">
      <n>PlanetScale (MySQL)</n>
      <pros>Serverless масштабирование, workflow с ветвлением, отличный DX</pros>
      <cons>MySQL а не Postgres, нет foreign keys в бесплатном тарифе</cons>
    </option>
    <option id="convex">
      <n>Convex</n>
      <pros>Real-time по умолчанию, нативный TypeScript, автоматическое кеширование</pros>
      <cons>Более новая платформа, другая ментальная модель, менее гибкий SQL</cons>
    </option>
  </options>
  <resume-signal>Выберите: supabase, planetscale или convex</resume-signal>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action (1% — Редкий)

**Когда:** Действие НЕ ИМЕЕТ CLI/API и требует исключительно человеческого взаимодействия, ИЛИ Claude столкнулся с барьером аутентификации при автоматизации.

**Используется ТОЛЬКО для:**
- **Барьеров аутентификации** — Claude попробовал CLI/API, но нужны учётные данные (это НЕ сбой)
- Ссылок подтверждения email (клик по ссылке из письма)
- SMS 2FA кодов (верификация по телефону)
- Ручных одобрений аккаунтов (платформа требует человеческую проверку)
- Потоков 3D Secure для кредитных карт (веб-авторизация платежа)
- Одобрений OAuth-приложений (веб-одобрение)

**НЕ используется для запланированной ручной работы:**
- Развертывание (используйте CLI — барьер auth при необходимости)
- Создание вебхуков/баз данных (используйте API/CLI — барьер auth при необходимости)
- Запуск сборки/тестов (используйте Bash tool)
- Создание файлов (используйте Write tool)

**Структура:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[Что должен сделать человек — Claude уже автоматизировал всё возможное]</action>
  <instructions>
    [Что Claude уже автоматизировал]
    [ОДНА вещь, требующая действия человека]
  </instructions>
  <verification>[Что Claude может проверить после]</verification>
  <resume-signal>[Как продолжить]</resume-signal>
</task>
```

**Пример: Верификация Email**
```xml
<task type="auto">
  <n>Создать аккаунт SendGrid через API</n>
  <action>Использовать SendGrid API для создания субпользовательского аккаунта с предоставленным email. Запросить письмо верификации.</action>
  <verify>API возвращает 201, аккаунт создан</verify>
  <done>Аккаунт создан, письмо верификации отправлено</done>
</task>

<task type="checkpoint:human-action" gate="blocking">
  <action>Завершить верификацию email для аккаунта SendGrid</action>
  <instructions>
    Я создал аккаунт и запросил письмо верификации.
    Проверьте входящие — найдите ссылку верификации SendGrid и кликните по ней.
  </instructions>
  <verification>API-ключ SendGrid работает: curl тест успешен</verification>
  <resume-signal>Напишите «готово» когда email подтверждён</resume-signal>
</task>
```

**Пример: Барьер аутентификации (Динамическая контрольная точка)**
```xml
<task type="auto">
  <n>Развернуть на Vercel</n>
  <files>.vercel/, vercel.json</files>
  <action>Выполнить `vercel --yes` для развертывания</action>
  <verify>vercel ls показывает deployment, curl возвращает 200</verify>
</task>

<!-- Если vercel возвращает «Error: Not authenticated», Claude создаёт контрольную точку на лету -->

<task type="checkpoint:human-action" gate="blocking">
  <action>Аутентифицировать Vercel CLI чтобы я мог продолжить развертывание</action>
  <instructions>
    Я попытался развернуть, но получил ошибку аутентификации.
    Выполните: vercel login
    Это откроет браузер — завершите поток аутентификации.
  </instructions>
  <verification>vercel whoami возвращает email вашего аккаунта</verification>
  <resume-signal>Напишите «готово» когда аутентифицированы</resume-signal>
</task>

<!-- После аутентификации Claude повторяет развертывание -->

<task type="auto">
  <n>Повторить развертывание Vercel</n>
  <action>Выполнить `vercel --yes` (теперь аутентифицирован)</action>
  <verify>vercel ls показывает deployment, curl возвращает 200</verify>
</task>
```

**Ключевое отличие:** Барьеры auth создаются динамически, когда Claude сталкивается с ошибками auth. НЕ планируются заранее — Claude автоматизирует сначала, запрашивает учётные данные только при блокировке.
</type>
</checkpoint_types>

<execution_protocol>

Когда Claude встречает `type="checkpoint:*"`:

1. **Немедленно остановиться** — не переходить к следующей задаче
2. **Чётко отобразить контрольную точку** используя формат ниже
3. **Ждать ответа пользователя** — не выдумывать завершение
4. **Верифицировать если возможно** — проверить файлы, запустить тесты, что указано
5. **Продолжить выполнение** — перейти к следующей задаче только после подтверждения

**Для checkpoint:human-verify:**
```
╔═══════════════════════════════════════════════════════╗
║  КОНТРОЛЬНАЯ ТОЧКА: Требуется верификация              ║
╚═══════════════════════════════════════════════════════╝

Прогресс: 5/8 задач выполнено
Задача: Адаптивный макет дашборда

Собрано: Адаптивный дашборд на /dashboard

Как проверить:
  1. Откройте: http://localhost:3000/dashboard
  2. Десктоп (>1024px): Боковая панель видна, контент заполняет оставшееся пространство
  3. Планшет (768px): Боковая панель сворачивается в иконки
  4. Мобильный (375px): Боковая панель скрыта, появляется гамбургер-меню

────────────────────────────────────────────────────────
→ ВАШЕ ДЕЙСТВИЕ: Напишите «одобрено» или опишите проблемы
────────────────────────────────────────────────────────
```

**Для checkpoint:decision:**
```
╔═══════════════════════════════════════════════════════╗
║  КОНТРОЛЬНАЯ ТОЧКА: Требуется решение                 ║
╚═══════════════════════════════════════════════════════╝

Прогресс: 2/6 задач выполнено
Задача: Выбрать провайдер аутентификации

Решение: Какой auth-провайдер использовать?

Контекст: Нужна аутентификация пользователей. Три варианта с разными компромиссами.

Варианты:
  1. supabase — Встроен в нашу БД, бесплатный тариф
     Плюсы: Интеграция с row-level security, щедрый бесплатный тариф
     Минусы: Менее настраиваемый UI, привязка к экосистеме

  2. clerk — Лучший DX, платный после 10k пользователей
     Плюсы: Красивый готовый UI, отличная документация
     Минусы: Зависимость от вендора, ценообразование при масштабировании

  3. nextauth — Self-hosted, максимальный контроль
     Плюсы: Бесплатный, без привязки к вендору, широко используемый
     Минусы: Больше работы по настройке, самостоятельные обновления безопасности

────────────────────────────────────────────────────────
→ ВАШЕ ДЕЙСТВИЕ: Выберите supabase, clerk или nextauth
────────────────────────────────────────────────────────
```

**Для checkpoint:human-action:**
```
╔═══════════════════════════════════════════════════════╗
║  КОНТРОЛЬНАЯ ТОЧКА: Требуется действие                ║
╚═══════════════════════════════════════════════════════╝

Прогресс: 3/8 задач выполнено
Задача: Развернуть на Vercel

Попытка: vercel --yes
Ошибка: Not authenticated. Please run 'vercel login'

Что вам нужно сделать:
  1. Выполните: vercel login
  2. Завершите аутентификацию в браузере когда он откроется
  3. Вернитесь сюда когда закончите

Я проверю: vercel whoami вернёт ваш аккаунт

────────────────────────────────────────────────────────
→ ВАШЕ ДЕЙСТВИЕ: Напишите «готово» когда аутентифицированы
────────────────────────────────────────────────────────
```
</execution_protocol>

<authentication_gates>

**Барьер auth = Claude попробовал CLI/API, получил ошибку auth.** Не сбой — барьер, требующий ввода от человека для разблокировки.

**Паттерн:** Claude пробует автоматизацию → ошибка auth → создаёт checkpoint:human-action → пользователь аутентифицируется → Claude повторяет → продолжает

**Протокол барьера:**
1. Распознать что это не сбой — отсутствие auth ожидаемо
2. Остановить текущую задачу — не повторять бесконечно
3. Создать checkpoint:human-action динамически
4. Указать точные шаги аутентификации
5. Проверить что аутентификация работает
6. Повторить исходную задачу
7. Продолжить нормально

**Ключевое отличие:**
- Заранее спланированная контрольная точка: «Мне нужно чтобы вы сделали X» (неправильно — Claude должен автоматизировать)
- Барьер auth: «Я пытался автоматизировать X, но нужны учётные данные» (правильно — разблокирует автоматизацию)

</authentication_gates>

<automation_reference>

**Правило:** Если есть CLI/API — Claude делает сам. Никогда не просить человека выполнять автоматизируемую работу.

## Справочник CLI сервисов

| Сервис | CLI/API | Ключевые команды | Барьер Auth |
|---------|---------|--------------|-----------|
| Vercel | `vercel` | `--yes`, `env add`, `--prod`, `ls` | `vercel login` |
| Railway | `railway` | `init`, `up`, `variables set` | `railway login` |
| Fly | `fly` | `launch`, `deploy`, `secrets set` | `fly auth login` |
| Stripe | `stripe` + API | `listen`, `trigger`, API-вызовы | API key в .env |
| Supabase | `supabase` | `init`, `link`, `db push`, `gen types` | `supabase login` |
| Upstash | `upstash` | `redis create`, `redis get` | `upstash auth login` |
| PlanetScale | `pscale` | `database create`, `branch create` | `pscale auth login` |
| GitHub | `gh` | `repo create`, `pr create`, `secret set` | `gh auth login` |
| Node | `npm`/`pnpm` | `install`, `run build`, `test`, `run dev` | Н/Д |
| Xcode | `xcodebuild` | `-project`, `-scheme`, `build`, `test` | Н/Д |
| Convex | `npx convex` | `dev`, `deploy`, `env set`, `env get` | `npx convex login` |

## Автоматизация переменных окружения

**Env-файлы:** Использовать инструменты Write/Edit. Никогда не просить человека создавать .env вручную.

**Переменные окружения на дашборде через CLI:**

| Платформа | CLI-команда | Пример |
|----------|-------------|---------|
| Convex | `npx convex env set` | `npx convex env set OPENAI_API_KEY sk-...` |
| Vercel | `vercel env add` | `vercel env add STRIPE_KEY production` |
| Railway | `railway variables set` | `railway variables set API_KEY=value` |
| Fly | `fly secrets set` | `fly secrets set DATABASE_URL=...` |
| Supabase | `supabase secrets set` | `supabase secrets set MY_SECRET=value` |

**Паттерн сбора секретов:**
```xml
<!-- НЕПРАВИЛЬНО: Просить пользователя добавить env vars через дашборд -->
<task type="checkpoint:human-action">
  <action>Добавить OPENAI_API_KEY в дашборд Convex</action>
  <instructions>Перейдите на dashboard.convex.dev → Settings → Environment Variables → Add</instructions>
</task>

<!-- ПРАВИЛЬНО: Claude запрашивает значение, затем добавляет через CLI -->
<task type="checkpoint:human-action">
  <action>Предоставьте ваш API-ключ OpenAI</action>
  <instructions>
    Мне нужен ваш API-ключ OpenAI для бэкенда Convex.
    Получите его на: https://platform.openai.com/api-keys
    Вставьте ключ (начинается с sk-)
  </instructions>
  <verification>Я добавлю его через `npx convex env set` и проверю</verification>
  <resume-signal>Вставьте ваш API-ключ</resume-signal>
</task>

<task type="auto">
  <n>Настроить ключ OpenAI в Convex</n>
  <action>Выполнить `npx convex env set OPENAI_API_KEY {ключ-от-пользователя}`</action>
  <verify>`npx convex env get OPENAI_API_KEY` возвращает ключ (замаскированный)</verify>
</task>
```

## Автоматизация Dev-серверов

| Фреймворк | Команда запуска | Сигнал готовности | URL по умолчанию |
|-----------|---------------|--------------|-------------|
| Next.js | `npm run dev` | «Ready in» или «started server» | http://localhost:3000 |
| Vite | `npm run dev` | «ready in» | http://localhost:5173 |
| Convex | `npx convex dev` | «Convex functions ready» | Н/Д (только бэкенд) |
| Express | `npm start` | «listening on port» | http://localhost:3000 |
| Django | `python manage.py runserver` | «Starting development server» | http://localhost:8000 |

**Жизненный цикл сервера:**
```bash
# Запуск в фоне, захват PID
npm run dev &
DEV_SERVER_PID=$!

# Ожидание готовности (макс 30 сек)
timeout 30 bash -c 'until curl -s localhost:3000 > /dev/null 2>&1; do sleep 1; done'
```

**Конфликты портов:** Убить зависший процесс (`lsof -ti:3000 | xargs kill`) или использовать альтернативный порт (`--port 3001`).

**Сервер остаётся запущенным** на протяжении контрольных точек. Останавливать только по завершении плана, при переключении на продакшн или когда порт нужен другому сервису.

## Обработка установки CLI

| CLI | Автоустановка? | Команда |
|-----|---------------|---------|
| npm/pnpm/yarn | Нет — спросить пользователя | Пользователь выбирает пакетный менеджер |
| vercel | Да | `npm i -g vercel` |
| gh (GitHub) | Да | `brew install gh` (macOS) или `apt install gh` (Linux) |
| stripe | Да | `npm i -g stripe` |
| supabase | Да | `npm i -g supabase` |
| convex | Нет — использовать npx | `npx convex` (установка не нужна) |
| fly | Да | `brew install flyctl` или curl-установщик |
| railway | Да | `npm i -g @railway/cli` |

**Протокол:** Попробовать команду → «command not found» → автоустановка возможна? → да: установить тихо, повторить → нет: контрольная точка с просьбой к пользователю установить.

## Сбои перед контрольной точкой

| Сбой | Реакция |
|---------|----------|
| Сервер не запускается | Проверить ошибку, исправить, повторить (не переходить к контрольной точке) |
| Порт занят | Убить зависший процесс или использовать альтернативный порт |
| Отсутствует зависимость | Выполнить `npm install`, повторить |
| Ошибка сборки | Сначала исправить ошибку (баг, не проблема контрольной точки) |
| Ошибка auth | Создать контрольную точку барьера auth |
| Таймаут сети | Повторить с отступом, затем контрольная точка если проблема сохраняется |

**Никогда не показывать контрольную точку с неработающей средой верификации.** Если `curl localhost:3000` не работает, не просите пользователя «посетить localhost:3000».

```xml
<!-- НЕПРАВИЛЬНО: Контрольная точка с неработающей средой -->
<task type="checkpoint:human-verify">
  <what-built>Дашборд (сервер не запустился)</what-built>
  <how-to-verify>Откройте http://localhost:3000...</how-to-verify>
</task>

<!-- ПРАВИЛЬНО: Сначала исправить, потом контрольная точка -->
<task type="auto">
  <n>Исправить проблему запуска сервера</n>
  <action>Исследовать ошибку, устранить причину, перезапустить сервер</action>
  <verify>curl http://localhost:3000 возвращает 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Дашборд — сервер запущен на http://localhost:3000</what-built>
  <how-to-verify>Откройте http://localhost:3000/dashboard...</how-to-verify>
</task>
```

## Краткий справочник автоматизации

| Действие | Автоматизируемо? | Claude делает сам? |
|--------|--------------|-----------------|
| Развернуть на Vercel | Да (`vercel`) | ДА |
| Создать Stripe webhook | Да (API) | ДА |
| Записать .env файл | Да (Write tool) | ДА |
| Создать Upstash БД | Да (`upstash`) | ДА |
| Запустить тесты | Да (`npm test`) | ДА |
| Запустить dev-сервер | Да (`npm run dev`) | ДА |
| Добавить env vars в Convex | Да (`npx convex env set`) | ДА |
| Добавить env vars в Vercel | Да (`vercel env add`) | ДА |
| Заполнить базу данных | Да (CLI/API) | ДА |
| Кликнуть ссылку подтверждения email | Нет | НЕТ |
| Ввести кредитную карту с 3DS | Нет | НЕТ |
| Завершить OAuth в браузере | Нет | НЕТ |
| Визуально проверить корректность UI | Нет | НЕТ |
| Протестировать интерактивные потоки | Нет | НЕТ |

</automation_reference>

<writing_guidelines>

**ДЕЛАЙТЕ:**
- Автоматизировать всё через CLI/API перед контрольной точкой
- Быть конкретным: «Откройте https://myapp.vercel.app» а не «проверьте развертывание»
- Нумеровать шаги верификации
- Указывать ожидаемые результаты: «Вы должны увидеть X»
- Давать контекст: зачем существует эта контрольная точка

**НЕ ДЕЛАЙТЕ:**
- Просить человека делать работу которую Claude может автоматизировать ❌
- Предполагать знания: «Настройте стандартные параметры» ❌
- Пропускать шаги: «Настройте базу данных» (слишком размыто) ❌
- Смешивать несколько верификаций в одной контрольной точке ❌

**Размещение:**
- **После завершения автоматизации** — не до того как Claude выполнит работу
- **После создания UI** — до объявления фазы завершённой
- **Перед зависимой работой** — решения до реализации
- **В точках интеграции** — после настройки внешних сервисов

**Плохое размещение:** До автоматизации ❌ | Слишком часто ❌ | Слишком поздно (зависимые задачи уже нуждались в результате) ❌
</writing_guidelines>

<examples>

### Пример 1: Настройка базы данных (Контрольная точка не нужна)

```xml
<task type="auto">
  <n>Создать Upstash Redis базу данных</n>
  <files>.env</files>
  <action>
    1. Выполнить `upstash redis create myapp-cache --region us-east-1`
    2. Захватить URL подключения из вывода
    3. Записать в .env: UPSTASH_REDIS_URL={url}
    4. Проверить подключение тестовой командой
  </action>
  <verify>
    - upstash redis list показывает базу данных
    - .env содержит UPSTASH_REDIS_URL
    - Тестовое подключение успешно
  </verify>
  <done>Redis база данных создана и настроена</done>
</task>

<!-- КОНТРОЛЬНАЯ ТОЧКА НЕ НУЖНА — Claude всё автоматизировал и проверил программно -->
```

### Пример 2: Полный поток Auth (Одна контрольная точка в конце)

```xml
<task type="auto">
  <n>Создать схему пользователя</n>
  <files>src/db/schema.ts</files>
  <action>Определить таблицы User, Session, Account с Drizzle ORM</action>
  <verify>npm run db:generate завершается успешно</verify>
</task>

<task type="auto">
  <n>Создать API-маршруты auth</n>
  <files>src/app/api/auth/[...nextauth]/route.ts</files>
  <action>Настроить NextAuth с провайдером GitHub, стратегия JWT</action>
  <verify>TypeScript компилируется, ошибок нет</verify>
</task>

<task type="auto">
  <n>Создать UI логина</n>
  <files>src/app/login/page.tsx, src/components/LoginButton.tsx</files>
  <action>Создать страницу логина с кнопкой GitHub OAuth</action>
  <verify>npm run build завершается успешно</verify>
</task>

<task type="auto">
  <n>Запустить dev-сервер для тестирования auth</n>
  <action>Выполнить `npm run dev` в фоне, дождаться сигнала готовности</action>
  <verify>curl http://localhost:3000 возвращает 200</verify>
  <done>Dev-сервер запущен на http://localhost:3000</done>
</task>

<!-- ОДНА контрольная точка в конце проверяет весь поток -->
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Полный поток аутентификации — dev-сервер запущен на http://localhost:3000</what-built>
  <how-to-verify>
    1. Откройте: http://localhost:3000/login
    2. Нажмите «Войти через GitHub»
    3. Завершите поток OAuth GitHub
    4. Проверьте: Перенаправление на /dashboard, имя пользователя отображается
    5. Обновите страницу: Сессия сохраняется
    6. Нажмите выход: Сессия очищена
  </how-to-verify>
  <resume-signal>Напишите «одобрено» или опишите проблемы</resume-signal>
</task>
```
</examples>

<anti_patterns>

### ❌ ПЛОХО: Просить пользователя запустить dev-сервер

```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Компонент дашборда</what-built>
  <how-to-verify>
    1. Выполните: npm run dev
    2. Откройте: http://localhost:3000/dashboard
    3. Проверьте корректность верстки
  </how-to-verify>
</task>
```

**Почему плохо:** Claude может выполнить `npm run dev`. Пользователь должен только посещать URL, а не выполнять команды.

### ✅ ХОРОШО: Claude запускает сервер, пользователь посещает

```xml
<task type="auto">
  <n>Запустить dev-сервер</n>
  <action>Выполнить `npm run dev` в фоне</action>
  <verify>curl localhost:3000 возвращает 200</verify>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>Дашборд на http://localhost:3000/dashboard (сервер запущен)</what-built>
  <how-to-verify>
    Откройте http://localhost:3000/dashboard и проверьте:
    1. Верстка соответствует дизайну
    2. Нет ошибок в консоли
  </how-to-verify>
</task>
```

### ❌ ПЛОХО: Просить человека развернуть / ✅ ХОРОШО: Claude автоматизирует

```xml
<!-- ПЛОХО: Просить пользователя развернуть через дашборд -->
<task type="checkpoint:human-action" gate="blocking">
  <action>Развернуть на Vercel</action>
  <instructions>Перейдите на vercel.com/new → Импортировать репо → Нажать Deploy → Скопировать URL</instructions>
</task>

<!-- ХОРОШО: Claude разворачивает, пользователь проверяет -->
<task type="auto">
  <n>Развернуть на Vercel</n>
  <action>Выполнить `vercel --yes`. Захватить URL.</action>
  <verify>vercel ls показывает deployment, curl возвращает 200</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Развернуто на {url}</what-built>
  <how-to-verify>Откройте {url}, проверьте загрузку главной страницы</how-to-verify>
  <resume-signal>Напишите «одобрено»</resume-signal>
</task>
```

### ❌ ПЛОХО: Слишком много контрольных точек / ✅ ХОРОШО: Одна контрольная точка

```xml
<!-- ПЛОХО: Контрольная точка после каждой задачи -->
<task type="auto">Создать схему</task>
<task type="checkpoint:human-verify">Проверить схему</task>
<task type="auto">Создать API-маршрут</task>
<task type="checkpoint:human-verify">Проверить API</task>
<task type="auto">Создать UI-форму</task>
<task type="checkpoint:human-verify">Проверить форму</task>

<!-- ХОРОШО: Одна контрольная точка в конце -->
<task type="auto">Создать схему</task>
<task type="auto">Создать API-маршрут</task>
<task type="auto">Создать UI-форму</task>

<task type="checkpoint:human-verify">
  <what-built>Полный поток auth (схема + API + UI)</what-built>
  <how-to-verify>Протестировать полный цикл: регистрация, логин, доступ к защищённой странице</how-to-verify>
  <resume-signal>Напишите «одобрено»</resume-signal>
</task>
```

### ❌ ПЛОХО: Расплывчатая верификация / ✅ ХОРОШО: Конкретные шаги

```xml
<!-- ПЛОХО -->
<task type="checkpoint:human-verify">
  <what-built>Дашборд</what-built>
  <how-to-verify>Проверьте что работает</how-to-verify>
</task>

<!-- ХОРОШО -->
<task type="checkpoint:human-verify">
  <what-built>Адаптивный дашборд — сервер запущен на http://localhost:3000</what-built>
  <how-to-verify>
    Откройте http://localhost:3000/dashboard и проверьте:
    1. Десктоп (>1024px): Боковая панель видна, область контента заполняет оставшееся пространство
    2. Планшет (768px): Боковая панель сворачивается в иконки
    3. Мобильный (375px): Боковая панель скрыта, гамбургер-меню в шапке
    4. Нет горизонтальной прокрутки при любом размере
  </how-to-verify>
  <resume-signal>Напишите «одобрено» или опишите проблемы с версткой</resume-signal>
</task>
```

### ❌ ПЛОХО: Просить пользователя выполнять CLI-команды

```xml
<task type="checkpoint:human-action">
  <action>Запустить миграции базы данных</action>
  <instructions>Выполните: npx prisma migrate deploy && npx prisma db seed</instructions>
</task>
```

**Почему плохо:** Claude может выполнить эти команды. Пользователь никогда не должен выполнять CLI-команды.

### ❌ ПЛОХО: Просить пользователя копировать значения между сервисами

```xml
<task type="checkpoint:human-action">
  <action>Настроить URL вебхука в Stripe</action>
  <instructions>Скопировать URL развертывания → Stripe Dashboard → Webhooks → Add endpoint → Скопировать секрет → Добавить в .env</instructions>
</task>
```

**Почему плохо:** У Stripe есть API. Claude должен создать вебхук через API и записать в .env напрямую.

</anti_patterns>

<summary>

Контрольные точки формализуют моменты участия человека для верификации и принятия решений, а не для ручной работы.

**Золотое правило:** Если Claude МОЖЕТ автоматизировать — Claude ОБЯЗАН автоматизировать.

**Приоритет контрольных точек:**
1. **checkpoint:human-verify** (90%) — Claude всё автоматизировал, человек подтверждает визуальную/функциональную корректность
2. **checkpoint:decision** (9%) — Человек принимает архитектурные/технологические решения
3. **checkpoint:human-action** (1%) — Действительно неизбежные ручные шаги без API/CLI

**Когда НЕ использовать контрольные точки:**
- Вещи которые Claude может проверить программно (тесты, сборки)
- Файловые операции (Claude может читать файлы)
- Корректность кода (тесты и статический анализ)
- Всё что автоматизируется через CLI/API
</summary>