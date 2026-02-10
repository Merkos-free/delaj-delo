<purpose>
Удалить неначатую будущую фазу из дорожной карты проекта, удалить её каталог, перенумеровать все последующие фазы для поддержания чистой линейной последовательности и закоммитить изменение. Git-коммит служит историческим журналом удаления.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="parse_arguments">
Разберите аргументы команды:
- Аргумент — номер фазы для удаления (целое или десятичное число)
- Пример: `/gsd:remove-phase 17` → phase = 17
- Пример: `/gsd:remove-phase 16.1` → phase = 16.1

Если аргумент не указан:

```
ОШИБКА: Требуется номер фазы
Использование: /gsd:remove-phase <номер-фазы>
Пример: /gsd:remove-phase 17
```

Выход.
</step>

<step name="init_context">
Загрузите контекст операции с фазой:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init phase-op "${target}")
```

Извлеките: `phase_found`, `phase_dir`, `phase_number`, `commit_docs`, `roadmap_exists`.

Также прочитайте содержимое STATE.md и ROADMAP.md для определения текущей позиции.
</step>

<step name="validate_future_phase">
Убедитесь, что фаза является будущей (не начата):

1. Сравните целевую фазу с текущей фазой из STATE.md
2. Целевая должна быть > текущего номера фазы

Если target <= текущей фазы:

```
ОШИБКА: Невозможно удалить Фазу {target}

Удалять можно только будущие фазы:
- Текущая фаза: {current}
- Фаза {target} является текущей или завершённой

Для отказа от текущей работы используйте /gsd:pause-work.
```

Выход.
</step>

<step name="confirm_removal">
Представьте сводку удаления и подтвердите:

```
Удаление Фазы {target}: {Name}

Будет выполнено:
- Удалён: .planning/phases/{target}-{slug}/
- Перенумерованы все последующие фазы
- Обновлены: ROADMAP.md, STATE.md

Продолжить? (y/n)
```

Ожидайте подтверждения.
</step>

<step name="execute_removal">
**Делегируйте всю операцию удаления в gsd-tools:**

```bash
RESULT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase remove "${target}")
```

Если у фазы есть выполненные планы (файлы SUMMARY.md), gsd-tools вернёт ошибку. Используйте `--force` только после подтверждения пользователя:

```bash
RESULT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js phase remove "${target}" --force)
```

CLI выполняет:
- Удаление каталога фазы
- Перенумерацию всех последующих каталогов (в обратном порядке для избежания конфликтов)
- Переименование всех файлов внутри перенумерованных каталогов (PLAN.md, SUMMARY.md и т.д.)
- Обновление ROADMAP.md (удаление секции, перенумерация всех ссылок на фазы, обновление зависимостей)
- Обновление STATE.md (уменьшение счётчика фаз)

Извлеките из результата: `removed`, `directory_deleted`, `renamed_directories`, `renamed_files`, `roadmap_updated`, `state_updated`.
</step>

<step name="commit">
Проиндексируйте и закоммитьте удаление:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "chore: remove phase {target} ({original-phase-name})" --files .planning/
```

Сообщение коммита сохраняет историческую запись о том, что было удалено.
</step>

<step name="completion">
Представьте итоговый отчёт:

```
Фаза {target} ({original-name}) удалена.

Изменения:
- Удалён: .planning/phases/{target}-{slug}/
- Перенумеровано: {N} каталогов и {M} файлов
- Обновлены: ROADMAP.md, STATE.md
- Закоммичено: chore: remove phase {target} ({original-name})

---

## Что дальше

Что вы хотите сделать:
- `/gsd:progress` — посмотреть обновлённый статус дорожной карты
- Продолжить текущую фазу
- Просмотреть дорожную карту

---
```
</step>

</process>

<anti_patterns>

- Не удаляйте завершённые фазы (имеющие файлы SUMMARY.md) без --force
- Не удаляйте текущие или прошлые фазы
- Не перенумеровывайте вручную — используйте `gsd-tools phase remove`, который обрабатывает всю перенумерацию
- Не добавляйте заметки об удалённой фазе в STATE.md — git-коммит является записью
- Не модифицируйте каталоги завершённых фаз
</anti_patterns>

<success_criteria>
Удаление фазы завершено, когда:

- [ ] Целевая фаза подтверждена как будущая/неначатая
- [ ] `gsd-tools phase remove` выполнена успешно
- [ ] Изменения закоммичены с описательным сообщением
- [ ] Пользователь проинформирован об изменениях
</success_criteria>
