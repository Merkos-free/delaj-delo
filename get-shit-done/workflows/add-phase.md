<purpose>
Добавить новую целочисленную фазу в конец текущего этапа в дорожной карте. Автоматически вычисляет следующий номер фазы, создаёт каталог фазы и обновляет структуру дорожной карты.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="parse_arguments">
Разберите аргументы команды:
- Все аргументы становятся описанием фазы
- Пример: `/gsd:add-phase Добавить аутентификацию` → description = "Добавить аутентификацию"
- Пример: `/gsd:add-phase Исправить критические проблемы производительности` → description = "Исправить критические проблемы производительности"

Если аргументы не указаны:

```
ОШИБКА: Требуется описание фазы
Использование: /gsd:add-phase <описание>
Пример: /gsd:add-phase Добавить систему аутентификации
```

Выход.
</step>

<step name="init_context">
Загрузите контекст операции с фазой:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init phase-op "0")
```

Проверьте `roadmap_exists` из JSON инициализации. Если false:
```
ОШИБКА: Дорожная карта не найдена (.planning/ROADMAP.md)
Запустите /gsd:new-project для инициализации.
```
Выход.
</step>

<step name="add_phase">
**Делегируйте добавление фазы в gsd-tools:**

```bash
RESULT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase add "${description}")
```

CLI выполняет:
- Поиск наибольшего существующего целочисленного номера фазы
- Вычисление следующего номера фазы (max + 1)
- Генерацию slug из описания
- Создание каталога фазы (`.planning/phases/{NN}-{slug}/`)
- Вставку записи фазы в ROADMAP.md с секциями Goal, Depends on и Plans

Извлеките из результата: `phase_number`, `padded`, `name`, `slug`, `directory`.
</step>

<step name="update_project_state">
Обновите STATE.md для отражения новой фазы:

1. Прочитайте `.planning/STATE.md`
2. В секции "## Accumulated Context" → "### Roadmap Evolution" добавьте запись:
   ```
   - Phase {N} added: {description}
   ```

Если секция "Roadmap Evolution" не существует, создайте её.
</step>

<step name="completion">
Представьте итоговый отчёт:

```
Фаза {N} добавлена в текущий этап:
- Описание: {description}
- Каталог: .planning/phases/{phase-num}-{slug}/
- Статус: Ещё не запланирована

Дорожная карта обновлена: .planning/ROADMAP.md

---

## ▶ Далее

**Фаза {N}: {description}**

`/gsd:plan-phase {N}`

<sub>`/clear` сначала → чистое контекстное окно</sub>

---

**Также доступно:**
- `/gsd:add-phase <описание>` — добавить ещё одну фазу
- Просмотр дорожной карты

---
```
</step>

</process>

<success_criteria>
- [ ] `gsd-tools phase add` выполнена успешно
- [ ] Каталог фазы создан
- [ ] Дорожная карта обновлена записью о новой фазе
- [ ] STATE.md обновлён заметкой об эволюции дорожной карты
- [ ] Пользователь проинформирован о следующих шагах
</success_criteria>
