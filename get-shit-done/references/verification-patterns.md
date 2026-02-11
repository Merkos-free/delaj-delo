# Паттерны верификации

Как проверить что различные типы артефактов являются реальными реализациями, а не заглушками или плейсхолдерами.

<core_principle>
**Существование ≠ Реализация**

Наличие файла не означает что функция работает. Верификация должна проверять:
1. **Существует** — Файл присутствует по ожидаемому пути
2. **Содержательный** — Контент является реальной реализацией, а не плейсхолдером
3. **Связан** — Подключён к остальной системе
4. **Функционирует** — Действительно работает при вызове

Уровни 1-3 можно проверить программно. Уровень 4 часто требует человеческой верификации.
</core_principle>

<stub_detection>

## Универсальные паттерны заглушек

Эти паттерны указывают на плейсхолдерный код вне зависимости от типа файла:

**Заглушки на основе комментариев:**
```bash
# Grep-паттерны для комментариев-заглушек
grep -E "(TODO|FIXME|XXX|HACK|PLACEHOLDER)" "$file"
grep -E "implement|add later|coming soon|will be" "$file" -i
grep -E "// \.\.\.|/\* \.\.\. \*/|# \.\.\." "$file"
```

**Плейсхолдерный текст в выводе:**
```bash
# Паттерны плейсхолдеров в UI
grep -E "placeholder|lorem ipsum|coming soon|under construction" "$file" -i
grep -E "sample|example|test data|dummy" "$file" -i
grep -E "\[.*\]|<.*>|\{.*\}" "$file"  # Оставленные шаблонные скобки
```

**Пустые или тривиальные реализации:**
```bash
# Функции которые ничего не делают
grep -E "return null|return undefined|return \{\}|return \[\]" "$file"
grep -E "pass$|\.\.\.\b|\bnothing\b" "$file"
grep -E "console\.(log|warn|error).*only" "$file"  # Функции только с логированием
```

**Хардкод значений где ожидаются динамические:**
```bash
# Захардкоженные ID, счётчики или контент
grep -E "id.*=.*['\"].*['\"]" "$file"  # Захардкоженные строковые ID
grep -E "count.*=.*\d+|length.*=.*\d+" "$file"  # Захардкоженные счётчики
grep -E "\\\$\d+\.\d{2}|\d+ items" "$file"  # Захардкоженные отображаемые значения
```

</stub_detection>

<react_components>

## React/Next.js компоненты

**Проверка существования:**
```bash
# Файл существует и экспортирует компонент
[ -f "$component_path" ] && grep -E "export (default |)function|export const.*=.*\(" "$component_path"
```

**Проверка содержательности:**
```bash
# Возвращает реальный JSX, не плейсхолдер
grep -E "return.*<" "$component_path" | grep -v "return.*null" | grep -v "placeholder" -i

# Имеет содержательный контент (не просто обёрточный div)
grep -E "<[A-Z][a-zA-Z]+|className=|onClick=|onChange=" "$component_path"

# Использует props или state (не статический)
grep -E "props\.|useState|useEffect|useContext|\{.*\}" "$component_path"
```

**Паттерны заглушек специфичные для React:**
```javascript
// КРАСНЫЕ ФЛАГИ — Это заглушки:
return <div>Component</div>
return <div>Placeholder</div>
return <div>{/* TODO */}</div>
return <p>Coming soon</p>
return null
return <></>

// Тоже заглушки — пустые обработчики:
onClick={() => {}}
onChange={() => console.log('clicked')}
onSubmit={(e) => e.preventDefault()}  // Только предотвращает стандартное поведение, ничего не делает
```

**Проверка связанности:**
```bash
# Компонент импортирует что ему нужно
grep -E "^import.*from" "$component_path"

# Props реально используются (не просто получены)
# Ищем деструктуризацию или использование props.X
grep -E "\{ .* \}.*props|\bprops\.[a-zA-Z]+" "$component_path"

# API-вызовы существуют (для компонентов загружающих данные)
grep -E "fetch\(|axios\.|useSWR|useQuery|getServerSideProps|getStaticProps" "$component_path"
```

**Функциональная верификация (требуется человек):**
- Отображает ли компонент видимый контент?
- Реагируют ли интерактивные элементы на клики?
- Загружаются ли данные и отображаются?
- Правильно ли показываются состояния ошибок?

</react_components>

<api_routes>

## API-маршруты (Next.js App Router / Express / и т.д.)

**Проверка существования:**
```bash
# Файл маршрута существует
[ -f "$route_path" ]

# Экспортирует обработчики HTTP-методов (Next.js App Router)
grep -E "export (async )?(function|const) (GET|POST|PUT|PATCH|DELETE)" "$route_path"

# Или обработчики в стиле Express
grep -E "\.(get|post|put|patch|delete)\(" "$route_path"
```

**Проверка содержательности:**
```bash
# Имеет реальную логику, не просто оператор return
wc -l "$route_path"  # Более 10-15 строк указывает на реальную реализацию

# Взаимодействует с источником данных
grep -E "prisma\.|db\.|mongoose\.|sql|query|find|create|update|delete" "$route_path" -i

# Имеет обработку ошибок
grep -E "try|catch|throw|error|Error" "$route_path"

# Возвращает содержательный ответ
grep -E "Response\.json|res\.json|res\.send|return.*\{" "$route_path" | grep -v "message.*not implemented" -i
```

**Паттерны заглушек специфичные для API-маршрутов:**
```typescript
// КРАСНЫЕ ФЛАГИ — Это заглушки:
export async function POST() {
  return Response.json({ message: "Not implemented" })
}

export async function GET() {
  return Response.json([])  // Пустой массив без запроса к БД
}

export async function PUT() {
  return new Response()  // Пустой ответ
}

// Только логирование в консоль:
export async function POST(req) {
  console.log(await req.json())
  return Response.json({ ok: true })
}
```

**Проверка связанности:**
```bash
# Импортирует клиент базы данных/сервиса
grep -E "^import.*prisma|^import.*db|^import.*client" "$route_path"

# Реально использует тело запроса (для POST/PUT)
grep -E "req\.json\(\)|req\.body|request\.json\(\)" "$route_path"

# Валидирует ввод (не просто доверяет запросу)
grep -E "schema\.parse|validate|zod|yup|joi" "$route_path"
```

**Функциональная верификация (человек или автоматическая):**
- Возвращает ли GET реальные данные из базы?
- Создаёт ли POST действительно запись?
- Имеет ли ответ об ошибке правильный код статуса?
- Применяются ли проверки auth на самом деле?

</api_routes>

<database_schema>

## Схема базы данных (Prisma / Drizzle / SQL)

**Проверка существования:**
```bash
# Файл схемы существует
[ -f "prisma/schema.prisma" ] || [ -f "drizzle/schema.ts" ] || [ -f "src/db/schema.sql" ]

# Модель/таблица определена
grep -E "^model $model_name|CREATE TABLE $table_name|export const $table_name" "$schema_path"
```

**Проверка содержательности:**
```bash
# Имеет ожидаемые поля (не только id)
grep -A 20 "model $model_name" "$schema_path" | grep -E "^\s+\w+\s+\w+"

# Имеет связи если ожидаются
grep -E "@relation|REFERENCES|FOREIGN KEY" "$schema_path"

# Имеет подходящие типы полей (не все String)
grep -A 20 "model $model_name" "$schema_path" | grep -E "Int|DateTime|Boolean|Float|Decimal|Json"
```

**Паттерны заглушек специфичные для схем:**
```prisma
// КРАСНЫЕ ФЛАГИ — Это заглушки:
model User {
  id String @id
  // TODO: add fields
}

model Message {
  id        String @id
  content   String  // Только одно реальное поле
}

// Отсутствуют критичные поля:
model Order {
  id     String @id
  // Нет: userId, items, total, status, createdAt
}
```

**Проверка связанности:**
```bash
# Миграции существуют и применены
ls prisma/migrations/ 2>/dev/null | wc -l  # Должно быть > 0
npx prisma migrate status 2>/dev/null | grep -v "pending"

# Клиент сгенерирован
[ -d "node_modules/.prisma/client" ]
```

**Функциональная верификация:**
```bash
# Можно запросить таблицу (автоматически)
npx prisma db execute --stdin <<< "SELECT COUNT(*) FROM $table_name"
```

</database_schema>

<hooks_utilities>

## Пользовательские хуки и утилиты

**Проверка существования:**
```bash
# Файл существует и экспортирует функцию
[ -f "$hook_path" ] && grep -E "export (default )?(function|const)" "$hook_path"
```

**Проверка содержательности:**
```bash
# Хук использует React-хуки (для пользовательских хуков)
grep -E "useState|useEffect|useCallback|useMemo|useRef|useContext" "$hook_path"

# Имеет содержательное возвращаемое значение
grep -E "return \{|return \[" "$hook_path"

# Более чем тривиальная длина
[ $(wc -l < "$hook_path") -gt 10 ]
```

**Паттерны заглушек специфичные для хуков:**
```typescript
// КРАСНЫЕ ФЛАГИ — Это заглушки:
export function useAuth() {
  return { user: null, login: () => {}, logout: () => {} }
}

export function useCart() {
  const [items, setItems] = useState([])
  return { items, addItem: () => console.log('add'), removeItem: () => {} }
}

// Захардкоженный возврат:
export function useUser() {
  return { name: "Test User", email: "test@example.com" }
}
```

**Проверка связанности:**
```bash
# Хук действительно импортируется где-то
grep -r "import.*$hook_name" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"

# Хук действительно вызывается
grep -r "$hook_name()" src/ --include="*.tsx" --include="*.ts" | grep -v "$hook_path"
```

</hooks_utilities>

<environment_config>

## Переменные окружения и конфигурация

**Проверка существования:**
```bash
# .env файл существует
[ -f ".env" ] || [ -f ".env.local" ]

# Требуемая переменная определена
grep -E "^$VAR_NAME=" .env .env.local 2>/dev/null
```

**Проверка содержательности:**
```bash
# Переменная имеет реальное значение (не плейсхолдер)
grep -E "^$VAR_NAME=.+" .env .env.local 2>/dev/null | grep -v "your-.*-here|xxx|placeholder|TODO" -i

# Значение выглядит корректным для типа:
# - URL должны начинаться с http
# - Ключи должны быть достаточной длины
# - Булевы должны быть true/false
```

**Паттерны заглушек специфичные для env:**
```bash
# КРАСНЫЕ ФЛАГИ — Это заглушки:
DATABASE_URL=your-database-url-here
STRIPE_SECRET_KEY=sk_test_xxx
API_KEY=placeholder
NEXT_PUBLIC_API_URL=http://localhost:3000  # Всё ещё указывает на localhost в проде
```

**Проверка связанности:**
```bash
# Переменная реально используется в коде
grep -r "process\.env\.$VAR_NAME|env\.$VAR_NAME" src/ --include="*.ts" --include="*.tsx"

# Переменная в схеме валидации (если используется zod/и т.д. для env)
grep -E "$VAR_NAME" src/env.ts src/env.mjs 2>/dev/null
```

</environment_config>

<wiring_verification>

## Паттерны верификации связанности

Верификация связанности проверяет что компоненты действительно взаимодействуют. Именно здесь большинство заглушек прячутся.

### Паттерн: Компонент → API

**Проверка:** Вызывает ли компонент действительно API?

```bash
# Найти вызов fetch/axios
grep -E "fetch\(['\"].*$api_path|axios\.(get|post).*$api_path" "$component_path"

# Убедиться что не закомментировано
grep -E "fetch\(|axios\." "$component_path" | grep -v "^.*//.*fetch"

# Проверить что ответ используется
grep -E "await.*fetch|\.then\(|setData|setState" "$component_path"
```

**Красные флаги:**
```typescript
// Fetch существует но ответ игнорируется:
fetch('/api/messages')  // Нет await, нет .then, нет присвоения

// Fetch в комментарии:
// fetch('/api/messages').then(r => r.json()).then(setMessages)

// Fetch на неправильный эндпоинт:
fetch('/api/message')  // Опечатка — должно быть /api/messages
```

### Паттерн: API → База данных

**Проверка:** Делает ли API-маршрут действительно запрос к базе данных?

```bash
# Найти вызов базы данных
grep -E "prisma\.$model|db\.query|Model\.find" "$route_path"

# Убедиться что await используется
grep -E "await.*prisma|await.*db\." "$route_path"

# Проверить что результат возвращается
grep -E "return.*json.*data|res\.json.*result" "$route_path"
```

**Красные флаги:**
```typescript
// Запрос существует но результат не возвращается:
await prisma.message.findMany()
return Response.json({ ok: true })  // Возвращает статику, не результат запроса

// Запрос без await:
const messages = prisma.message.findMany()  // Пропущен await
return Response.json(messages)  // Возвращает Promise, не данные
```

### Паттерн: Форма → Обработчик

**Проверка:** Делает ли отправка формы действительно что-то?

```bash
# Найти обработчик onSubmit
grep -E "onSubmit=\{|handleSubmit" "$component_path"

# Проверить что обработчик содержит логику
grep -A 10 "onSubmit.*=" "$component_path" | grep -E "fetch|axios|mutate|dispatch"

# Убедиться что не только preventDefault
grep -A 5 "onSubmit" "$component_path" | grep -v "only.*preventDefault" -i
```

**Красные флаги:**
```typescript
// Обработчик только предотвращает стандартное поведение:
onSubmit={(e) => e.preventDefault()}

// Обработчик только логирует:
const handleSubmit = (data) => {
  console.log(data)
}

// Обработчик пустой:
onSubmit={() => {}}
```

### Паттерн: Состояние → Отображение

**Проверка:** Отображает ли компонент состояние, а не захардкоженный контент?

```bash
# Найти использование состояния в JSX
grep -E "\{.*messages.*\}|\{.*data.*\}|\{.*items.*\}" "$component_path"

# Проверить map/render состояния
grep -E "\.map\(|\.filter\(|\.reduce\(" "$component_path"

# Проверить динамический контент
grep -E "\{[a-zA-Z_]+\." "$component_path"  # Интерполяция переменных
```

**Красные флаги:**
```tsx
// Захардкожено вместо состояния:
return <div>
  <p>Message 1</p>
  <p>Message 2</p>
</div>

// Состояние существует но не отображается:
const [messages, setMessages] = useState([])
return <div>No messages</div>  // Всегда показывает «нет сообщений»

// Отображается не то состояние:
const [messages, setMessages] = useState([])
return <div>{otherData.map(...)}</div>  // Использует другие данные
```

</wiring_verification>

<verification_checklist>

## Быстрый чек-лист верификации

Для каждого типа артефакта пройдите этот чек-лист:

### Чек-лист компонента
- [ ] Файл существует по ожидаемому пути
- [ ] Экспортирует функциональный/const компонент
- [ ] Возвращает JSX (не null/пустой)
- [ ] Нет плейсхолдерного текста в рендере
- [ ] Использует props или state (не статический)
- [ ] Обработчики событий имеют реальные реализации
- [ ] Импорты разрешаются корректно
- [ ] Используется где-то в приложении

### Чек-лист API-маршрута
- [ ] Файл существует по ожидаемому пути
- [ ] Экспортирует обработчики HTTP-методов
- [ ] Обработчики содержат более 5 строк
- [ ] Запрашивает базу данных или сервис
- [ ] Возвращает содержательный ответ (не пустой/плейсхолдер)
- [ ] Имеет обработку ошибок
- [ ] Валидирует ввод
- [ ] Вызывается из фронтенда

### Чек-лист схемы
- [ ] Модель/таблица определена
- [ ] Имеет все ожидаемые поля
- [ ] Поля имеют подходящие типы
- [ ] Связи определены если необходимо
- [ ] Миграции существуют и применены
- [ ] Клиент сгенерирован

### Чек-лист хука/утилиты
- [ ] Файл существует по ожидаемому пути
- [ ] Экспортирует функцию
- [ ] Имеет содержательную реализацию (не пустые возвраты)
- [ ] Используется где-то в приложении
- [ ] Возвращаемые значения потребляются

### Чек-лист связанности
- [ ] Компонент → API: вызов fetch/axios существует и использует ответ
- [ ] API → База данных: запрос существует и результат возвращается
- [ ] Форма → Обработчик: onSubmit вызывает API/мутацию
- [ ] Состояние → Отображение: переменные состояния появляются в JSX

</verification_checklist>

<automated_verification_script>

## Подход к автоматической верификации

Для суб-агента верификации используйте этот паттерн:

```bash
# 1. Проверка существования
check_exists() {
  [ -f "$1" ] && echo "EXISTS: $1" || echo "MISSING: $1"
}

# 2. Проверка на паттерны заглушек
check_stubs() {
  local file="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder|not implemented" "$file" 2>/dev/null || echo 0)
  [ "$stubs" -gt 0 ] && echo "STUB_PATTERNS: $stubs in $file"
}

# 3. Проверка связанности (компонент вызывает API)
check_wiring() {
  local component="$1"
  local api_path="$2"
  grep -q "$api_path" "$component" && echo "WIRED: $component → $api_path" || echo "NOT_WIRED: $component → $api_path"
}

# 4. Проверка содержательности (более N строк, содержит ожидаемые паттерны)
check_substantive() {
  local file="$1"
  local min_lines="$2"
  local pattern="$3"
  local lines=$(wc -l < "$file" 2>/dev/null || echo 0)
  local has_pattern=$(grep -c -E "$pattern" "$file" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && [ "$has_pattern" -gt 0 ] && echo "SUBSTANTIVE: $file" || echo "THIN: $file ($lines строк, $has_pattern совпадений)"
}
```

Запускайте эти проверки для каждого обязательного артефакта. Агрегируйте результаты в VERIFICATION.md.

</automated_verification_script>

<human_verification_triggers>

## Когда требовать человеческую верификацию

Некоторые вещи нельзя проверить программно. Помечайте их для тестирования человеком:

**Всегда человек:**
- Внешний вид (выглядит ли правильно?)
- Завершение пользовательского потока (можно ли реально сделать действие?)
- Поведение в реальном времени (WebSocket, SSE)
- Интеграция с внешними сервисами (Stripe, отправка email)
- Ясность сообщений об ошибках (понятно ли сообщение?)
- Ощущение производительности (кажется ли быстрым?)

**Человек при неопределённости:**
- Сложная связанность которую grep не может отследить
- Динамическое поведение зависящее от состояния
- Граничные случаи и состояния ошибок
- Мобильная адаптивность
- Доступность

**Формат запроса человеческой верификации:**
```markdown
## Требуется человеческая верификация

### 1. Отправка сообщения в чат
**Тест:** Введите сообщение и нажмите Отправить
**Ожидание:** Сообщение появляется в списке, поле ввода очищается
**Проверка:** Сохраняется ли сообщение после обновления страницы?

### 2. Обработка ошибок
**Тест:** Отключите сеть, попробуйте отправить
**Ожидание:** Появляется сообщение об ошибке, сообщение не потеряно
**Проверка:** Можно ли повторить после восстановления соединения?
```

</human_verification_triggers>

<checkpoint_automation_reference>

## Автоматизация перед контрольными точками

Для паттернов контрольных точек с приоритетом автоматизации, управления жизненным циклом серверов, обработки установки CLI и протоколов восстановления после ошибок, см.:

**@~/.claude/get-shit-done/references/checkpoints.md** → секция `<automation_reference>`

Ключевые принципы:
- Claude настраивает среду верификации ДО представления контрольных точек
- Пользователи никогда не выполняют CLI-команды (только посещают URL)
- Жизненный цикл серверов: запустить перед контрольной точкой, обработать конфликты портов, оставить запущенным на время
- Установка CLI: автоустановка где безопасно, контрольная точка для выбора пользователя в остальных случаях
- Обработка ошибок: исправить неработающую среду перед контрольной точкой, никогда не показывать контрольную точку с неудавшейся настройкой

</checkpoint_automation_reference>