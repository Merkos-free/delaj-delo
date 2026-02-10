# Парсинг аргументов фазы

Разбор и нормализация аргументов фазы для команд, работающих с фазами.

## Извлечение

Из `$ARGUMENTS`:
- Извлечь номер фазы (первый числовой аргумент)
- Извлечь флаги (с префиксом `--`)
- Оставшийся текст — описание (для команд insert/add)

## Использование gsd-tools

Команда `find-phase` выполняет нормализацию и валидацию за один шаг:

```bash
PHASE_INFO=$(node ~/.claude/get-shit-done/bin/gsd-tools.js find-phase "${PHASE}")
```

Возвращает JSON с полями:
- `found`: true/false
- `directory`: Полный путь к каталогу фазы
- `phase_number`: Нормализованный номер (напр., "06", "06.1")
- `phase_name`: Название (напр., "foundation")
- `plans`: Массив файлов PLAN.md
- `summaries`: Массив файлов SUMMARY.md

## Ручная нормализация (устаревший способ)

Целочисленные фазы дополняются нулями до 2 цифр. Десятичные суффиксы сохраняются.

```bash
# Нормализация номера фазы
if [[ "$PHASE" =~ ^[0-9]+$ ]]; then
  # Целое: 8 → 08
  PHASE=$(printf "%02d" "$PHASE")
elif [[ "$PHASE" =~ ^([0-9]+)\.([0-9]+)$ ]]; then
  # Десятичное: 2.1 → 02.1
  PHASE=$(printf "%02d.%s" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}")
fi
```

## Валидация

Используйте `roadmap get-phase` для проверки существования фазы:

```bash
PHASE_CHECK=$(node ~/.claude/get-shit-done/bin/gsd-tools.js roadmap get-phase "${PHASE}")
if [ "$(echo "$PHASE_CHECK" | jq -r '.found')" = "false" ]; then
  echo "ОШИБКА: Фаза ${PHASE} не найдена в дорожной карте"
  exit 1
fi
```

## Поиск каталога

Используйте `find-phase` для поиска каталога:

```bash
PHASE_DIR=$(node ~/.claude/get-shit-done/bin/gsd-tools.js find-phase "${PHASE}" --raw)
```
