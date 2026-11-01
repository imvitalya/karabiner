# Karabiner

Моя конфигурация [Karabiner-Elements](https://karabiner-elements.pqrs.org/): переключение русской и английской раскладок, двойное нажатие Shift, управление вкладками Zen Browser и переназначения клавиш.

Снимок рабочего профиля от 2 ноября 2026 года. Полная конфигурация находится в [`karabiner.json`](karabiner.json), отдельные правила для импорта — в [`assets/complex_modifications`](assets/complex_modifications).

## Правила

| Действие | Результат | Файл |
| --- | --- | --- |
| Двойной тап левого Ctrl | Русская — ПК ↔ ABC | [double-control-input-source.json](assets/complex_modifications/double-control-input-source.json) |
| Caps Lock на встроенной клавиатуре MacBook | Работает как левый Ctrl; двойной тап отправляет Ctrl+Shift+Space | [caps-lock-control-shortcut.json](assets/complex_modifications/caps-lock-control-shortcut.json) |
| Двойной тап правого Ctrl | Отправляет Ctrl+Shift+Space | [double-right-control-shortcut.json](assets/complex_modifications/double-right-control-shortcut.json) |
| Двойное нажатие левого Shift | Отправляет Cmd+Shift+Space | [double-shift-shortcut.json](assets/complex_modifications/double-shift-shortcut.json) |
| Ctrl+J / Ctrl+K в Zen Browser | Отправляет Ctrl+Tab / Ctrl+Shift+Tab для переключения вкладок | [zen-browser-tabs.json](assets/complex_modifications/zen-browser-tabs.json) |

### Переключение языка

Двойной Ctrl напрямую выбирает нужный источник ввода через [`select_input_source`](https://karabiner-elements.pqrs.org/docs/json/complex-modifications-manipulator-definition/to/select-input-source/):

- `com.apple.keylayout.RussianWin` — Русская — ПК;
- `com.apple.keylayout.ABC` — ABC.

Между началами двух нажатий должно пройти меньше 400 мс, каждое нажатие должно быть короче 300 мс. Раскладка меняется при отпускании второго Ctrl. Сочетания Ctrl с другими клавишами отменяют распознавание двойного тапа.

Ранее правило отправляло Ctrl+Shift+Space, назначенное в macOS на предыдущий источник ввода. При проверке это сочетание иногда оставляло прежнюю раскладку. Прямое переключение устранило пропуски при ручной проверке. Системное сочетание для переключения языка этому правилу не требуется.

### Caps Lock и правый Ctrl

Сохранено локальное поведение: Caps Lock на встроенной клавиатуре MacBook работает как левый Ctrl, а двойной тап отправляет Ctrl+Shift+Space. На внешних клавиатурах это правило не действует. Двойной тап правого Ctrl тоже отправляет Ctrl+Shift+Space; одиночное нажатие сохраняет обычный правый Ctrl.

Интервал между нажатиями — 300 мс. Для переключения языка этими двумя правилами сочетание Ctrl+Shift+Space должно быть назначено в macOS на предыдущий источник ввода.

Каждое правило обрабатывает свою клавишу. Дубли обработчиков левого и правого Ctrl из прежнего объединённого правила Caps Lock удалены, чтобы левый Ctrl использовал прямое переключение раскладки.

### Двойной Shift

Интервал между нажатиями — 300 мс. Правило отправляет Cmd+Shift+Space; действие на это сочетание назначается отдельно в macOS или нужном приложении.

### Переназначения в полном профиле

В полном `karabiner.json` дополнительно сохранены:

- для устройства с `vendor_id: 14000`, `product_id: 12292`: левый Command → левый Option, левый Option → левый Command, левый Ctrl → Fn;
- тип виртуальной клавиатуры ANSI и выбранный профиль `Default profile`.

Эти настройки устройств входят в полный профиль. Отдельные файлы правил содержат только Complex Modifications.

## Установка

Нужны macOS и Karabiner-Elements 16.3.0 или новее. Для переключения языка добавь **ABC** и **Русская — ПК** в «Системные настройки → Клавиатура → Ввод текста → Изменить».

```sh
git clone https://github.com/imvitalya/karabiner.git
cd karabiner
```

### Импорт отдельных правил

Скопируй файлы в каталог правил Karabiner:

```sh
karabiner_config_dir="${XDG_CONFIG_HOME:-$HOME/.config}/karabiner"
mkdir -p "$karabiner_config_dir/assets/complex_modifications"
cp assets/complex_modifications/*.json "$karabiner_config_dir/assets/complex_modifications/"
```

В Karabiner-Elements открой **Complex Modifications → Add predefined rule** и включи нужные правила. Перед включением новых отключи прежнее правило двойного Ctrl и старое объединённое правило Caps Lock с обработчиками Ctrl.

Если в **Simple Modifications** уже настроено Caps Lock → левый Ctrl, удали это переназначение перед включением отдельного правила Caps Lock. Иначе простое переназначение сработает раньше и правило двойного тапа Caps Lock не получит нажатия.

### Восстановление полного профиля

Этот вариант заменяет весь текущий `karabiner.json`, включая профили и настройки устройств. Сначала сохраняется резервная копия существующего файла:

```sh
karabiner_config_dir="${XDG_CONFIG_HOME:-$HOME/.config}/karabiner"
mkdir -p "$karabiner_config_dir/backups"
if [ -f "$karabiner_config_dir/karabiner.json" ]; then
    cp -p "$karabiner_config_dir/karabiner.json" \
        "$karabiner_config_dir/backups/karabiner-$(date +%Y%m%d-%H%M%S).json"
fi
cp karabiner.json "$karabiner_config_dir/karabiner.json"
```

Karabiner автоматически перечитает конфигурацию. После установки полного профиля отдельно включать экспортированные правила не нужно.

Для отката скопируй нужный файл из `backups` обратно в `karabiner.json`.

## Проверка конфигурации

```sh
"/Library/Application Support/org.pqrs/Karabiner-Elements/bin/karabiner_cli" \
    --lint-complex-modifications 'assets/complex_modifications/*.json'
```

## Обновление снимка

Изменения в настройках Karabiner сохраняются в его рабочем каталоге и автоматически в Git не попадают. При обновлении репозитория нужно заново скопировать актуальный `karabiner.json` и обновить соответствующие файлы в `assets/complex_modifications`, затем выполнить проверку выше.

Автоматические резервные копии и журналы исключены из Git.
