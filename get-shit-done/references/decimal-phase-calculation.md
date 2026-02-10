# Вычисление десятичной фазы

Вычисление следующего десятичного номера фазы для срочных вставок.

## Использование gsd-tools

```bash
# Получить следующую десятичную фазу после фазы 6
node ~/.claude/get-shit-done/bin/gsd-tools.js phase next-decimal 6
```

Вывод:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.1",
  "existing": []
}
```

С существующими десятичными:
```json
{
  "found": true,
  "base_phase": "06",
  "next": "06.3",
  "existing": ["06.1", "06.2"]
}
```

## Извлечение значений

```bash
DECIMAL_INFO=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase next-decimal "${AFTER_PHASE}")
DECIMAL_PHASE=$(echo "$DECIMAL_INFO" | jq -r '.next')
BASE_PHASE=$(echo "$DECIMAL_INFO" | jq -r '.base_phase')
```

Или с флагом --raw:
```bash
DECIMAL_PHASE=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase next-decimal "${AFTER_PHASE}" --raw)
# Возвращает просто: 06.1
```

## Примеры

| Существующие фазы | Следующая фаза |
|-------------------|----------------|
| только 06 | 06.1 |
| 06, 06.1 | 06.2 |
| 06, 06.1, 06.2 | 06.3 |
| 06, 06.1, 06.3 (пробел) | 06.4 |

## Именование каталогов

Каталоги десятичных фаз используют полный десятичный номер:

```bash
SLUG=$(node ~/.claude/get-shit-done/bin/gsd-tools.js generate-slug "$DESCRIPTION" --raw)
PHASE_DIR=".planning/phases/${DECIMAL_PHASE}-${SLUG}"
mkdir -p "$PHASE_DIR"
```

Пример: `.planning/phases/06.1-fix-critical-auth-bug/`