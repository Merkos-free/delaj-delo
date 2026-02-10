<purpose>
Переключить профиль модели, используемый агентами GSD. Контролирует, какую модель Claude использует каждый агент, балансируя между качеством и расходом токенов.
</purpose>

<required_reading>
Прочитайте все файлы, указанные в execution_context вызывающего промпта, перед началом работы.
</required_reading>

<process>

<step name="validate">
Проверьте аргумент:

```
if $ARGUMENTS.profile not in ["quality", "balanced", "budget"]:
  Ошибка: Недопустимый профиль "$ARGUMENTS.profile"
  Допустимые профили: quality, balanced, budget
  ВЫХОД
```
</step>

<step name="ensure_and_load_config">
Убедитесь, что конфиг существует, и загрузите текущее состояние:

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js config-ensure-section
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
```

Это создаёт `.planning/config.json` с значениями по умолчанию, если файл отсутствует, и загружает текущую конфигурацию.
</step>

<step name="update_config">
Прочитайте текущий конфиг из state load или напрямую:

Обновите поле `model_profile`:
```json
{
  "model_profile": "$ARGUMENTS.profile"
}
```

Запишите обновлённый конфиг обратно в `.planning/config.json`.
</step>

<step name="confirm">
Покажите подтверждение с таблицей моделей для выбранного профиля:

```
✓ Профиль модели установлен: $ARGUMENTS.profile

Агенты теперь будут использовать:

[Покажите таблицу из MODEL_PROFILES в gsd-tools.js для выбранного профиля]

Пример:
| Агент | Модель |
|-------|--------|
| gsd-planner | opus |
| gsd-executor | sonnet |
| gsd-verifier | haiku |
| ... | ... |

Следующие запущенные агенты будут использовать новый профиль.
```

Сопоставление названий профилей:
- quality: использовать столбец "quality" из MODEL_PROFILES
- balanced: использовать столбец "balanced" из MODEL_PROFILES
- budget: использовать столбец "budget" из MODEL_PROFILES
</step>

</process>

<success_criteria>
- [ ] Аргумент проверен
- [ ] Файл конфигурации обеспечен
- [ ] Конфиг обновлён новым model_profile
- [ ] Показано подтверждение с таблицей моделей
</success_criteria>
