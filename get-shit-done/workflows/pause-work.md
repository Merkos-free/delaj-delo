<purpose>
Создать файл передачи `.continue-here.md` для сохранения полного состояния работы между сессиями. Обеспечивает бесшовное возобновление с полным восстановлением контекста.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="detect">
Найдите текущий каталог фазы по последним изменённым файлам:

```bash
# Найти самый последний каталог фазы с работой
ls -lt .planning/phases/*/PLAN.md 2>/dev/null | head -1 | grep -oP 'phases/\K[^/]+'
```

Если активная фаза не обнаружена, спросите пользователя, на какой фазе он приостанавливает работу.
</step>

<step name="gather">
**Соберите полное состояние для передачи:**

1. **Текущая позиция**: Какая фаза, какой план, какая задача
2. **Выполненная работа**: Что было сделано в этой сессии
3. **Оставшаяся работа**: Что осталось в текущем плане/фазе
4. **Принятые решения**: Ключевые решения и обоснования
5. **Блокеры/проблемы**: Что застряло
6. **Ментальный контекст**: Подход, следующие шаги, общее направление
7. **Изменённые файлы**: Что изменено, но не закоммичено

При необходимости задайте пользователю уточняющие вопросы.
</step>

<step name="write">
**Запишите передачу в `.planning/phases/XX-name/.continue-here.md`:**

```markdown
---
phase: XX-name
task: 3
total_tasks: 7
status: in_progress
last_updated: [timestamp from current-timestamp]
---

<current_state>
[Где именно мы находимся? Непосредственный контекст]
</current_state>

<completed_work>

- Задача 1: [название] - Выполнена
- Задача 2: [название] - Выполнена
- Задача 3: [название] - В процессе, [что сделано]
</completed_work>

<remaining_work>

- Задача 3: [что осталось]
- Задача 4: Не начата
- Задача 5: Не начата
</remaining_work>

<decisions_made>

- Решили использовать [X], потому что [причина]
- Выбрали [подход] вместо [альтернативы], потому что [причина]
</decisions_made>

<blockers>
- [Блокер 1]: [статус/обходное решение]
</blockers>

<context>
[Ментальное состояние, о чём вы думали, план]
</context>

<next_action>
Начните с: [конкретное первое действие при возобновлении]
</next_action>
```

Будьте достаточно конкретны, чтобы свежий Claude мог сразу понять.

Используйте `current-timestamp` для поля last_updated. Можно использовать init todos (который предоставляет timestamps) или вызвать напрямую:
```bash
timestamp=$(node ~/.claude/get-shit-done/bin/gsd-tools.js current-timestamp full --raw)
```
</step>

<step name="commit">
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "wip: [phase-name] paused at task [X]/[Y]" --files .planning/phases/*/.continue-here.md
```
</step>

<step name="confirm">
```
✓ Файл передачи создан: .planning/phases/[XX-name]/.continue-here.md

Текущее состояние:

- Фаза: [XX-name]
- Задача: [X] из [Y]
- Статус: [in_progress/blocked]
- Закоммичено как WIP

Для возобновления: /gsd:resume-work

```
</step>

</process>

<success_criteria>
- [ ] .continue-here.md создан в правильном каталоге фазы
- [ ] Все секции заполнены конкретным содержимым
- [ ] Закоммичено как WIP
- [ ] Пользователь знает расположение и как возобновить
</success_criteria>
