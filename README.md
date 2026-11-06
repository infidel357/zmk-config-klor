<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/docs/images/klor-font-logo-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="/docs/images/klor-font-logo-bright.svg">
  <img alt="KLOR logo font" src="/docs/images/klor-font-logo-bright.svg">
</picture>


***

🇬🇧 [English](README.en.md) | 🇷🇺 Русский

# KLOR — конфиг ZMK (форк с экраном на донгле и ZMK Studio)

Конфигурация ZMK для сплит-клавиатуры **KLOR**, расширенная **донглом на Seeed XIAO BLE** с кастомной сборкой **YADS (zmk-dongle-screen)**: idle-картинка, поддержка ZMK Studio и корректное отображение раскладки KLOR в Studio.

- Оригинальный QMK-конфиг: [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR)
- Файлы железа и гайд по сборке: [GEIGEIGEIST/KLOR-hardware](https://github.com/GEIGEIGEIST/KLOR-hardware)
- Оригинальный dongle-screen: [janpfischer/zmk-dongle-screen](https://github.com/janpfischer/zmk-dongle-screen)

---

## ✨ Чем отличается от оригинального KLOR

### 1. Донгл с экраном (форк YADS)

- Плата: `seeeduino_xiao_ble`
- Щит: `klor_dongle dongle_screen`
- Сниппет: `studio-rpc-usb-uart`
- Форк `zmk-dongle-screen`: **[infidel357/zmk-dongle-screen](https://github.com/infidel357/zmk-dongle-screen)**

### 2. Idle-картинка

Через **20 секунд** бездействия появляется жёлтая пиксельная мордочка, висит **30 секунд**, затем экран плавно гаснет.

Конечный автомат в `brightness.c`:

```text
ACTIVE  ──(20 с простоя)──▶  IDLE_IMAGE  ──(30 с)──▶  BLANKED
   ▲                            │
   └──────── активность ────────┘
```

Новые файлы в форке dongle-screen:

- `src/assets/idle_face.c` — 276×236 `LV_IMG_CF_INDEXED_1BIT` (жёлтый + чёрный)
- `src/widgets/custom_idle_image.c` / `.h` — виджет LVGL

Опции Kconfig:

```kconfig
CONFIG_DONGLE_SCREEN_IDLE_TIMEOUT_S=20
CONFIG_DONGLE_SCREEN_IDLE_IMAGE=y
CONFIG_DONGLE_SCREEN_IDLE_IMAGE_BLANK_DELAY_S=30
```

### 3. ZMK Studio

Включена на донгле, **блокировка отключена** — комбо `&studio_unlock` не нужно:

```kconfig
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
```

### 4. KLOR в ZMK Studio

В `klor_dongle.overlay` добавлен `physical_layout0` с массивом `keys` из 44 элементов — тамбы рисуются **с наклоном** (взято из `vial.json`).

---

## 🚀 Как собрать и прошить

### Шаг 1. Форк и клонирование

1. Нажми **Fork** на странице этого репозитория (создай свою копию).
2. Клонируй форк локально:

```bash
git clone https://github.com/<твой-ник>/zmk-config-klor.git
cd zmk-config-klor
```

### Шаг 2. Правка раскладки (по желанию)

Отредактируй `config/klor.keymap` — коды клавиш смотри в [документации ZMK](https://zmk.dev/docs/codes).

### Шаг 3. Сборка через GitHub Actions

1. Закоммить и запушь изменения:

```bash
git add .
git commit -m "my keymap"
git push
```

2. На странице своего форка открой вкладку **Actions**.
3. Дождись окончания сборки (зелёная галочка).
4. Скачай артефакт **`firmware.zip`** (кнопка внизу страницы сборки).

> 💡 Если Actions не запустились автоматически — зайди в **Actions → Build → Run workflow**.

### Шаг 4. Прошивка

В архиве `firmware.zip` три файла:

| Файл | Куда прошивать |
|---|---|
| `klor_left-nice_nano_v2-zmk.uf2` | левая половина |
| `klor_right-nice_nano_v2-zmk.uf2` | правая половина |
| `klor_dongle-seeeduino_xiao_ble-zmk.uf2` | донгл (XIAO BLE) |

**Порядок прошивки:**

1. **Левая половина**: подключи к ПК, нажми **Reset** дважды — появится USB-накопитель. Перетащи `klor_left-...uf2` на него.
2. **Правая половина**: то же самое с `klor_right-...uf2`.
3. **Донгл**: подключи XIAO BLE, нажми **Reset** дважды, перетащи `klor_dongle-...uf2`.

> ⚠️ **Важно для донгла:** при первой прошивке обязательно сделай **сброс настроек** (settings reset).

### Шаг 5. Сопряжение

1. Включи донгл — на экране появится статус.
2. Включи левую и правую половины — они автоматически сопрягутся с донглом.
3. Подключи донгл к ПК по USB — клавиатура определится как HID-устройство.
4. Открой **ZMK Studio** ([studio.zmk.dev](https://studio.zmk.dev)) — раскладка редактируется **без** `&studio_unlock`.

### Шаг 6. Проверка

- ✅ Через 20 с простоя появляется мордочка, висит 30 с, экран гаснет.
- ✅ Левый энкодер — громкость, правый — трек/каретка по слоям.
- ✅ Tap-dance на `M`, комбо `Q+E` переключает язык.
- ✅ ZMK Studio открывается и показывает KLOR с наклонными тамбами.

---

## ⌨️ Раскладка

- **Слои**: `BASE`, `SYMBOLS`, `NUMBERS`, `ADJUST`, `EXTRA`
- **Левый энкодер** (все слои): `C_VOL_UP` / `C_VOL_DN`
- **Правый энкодер на BASE**: `C_NEXT` / `C_PREV` (сначала трек вперёд)
- **Правый энкодер на остальных слоях**: `&kp LEFT` / `&kp RIGHT` (каретка)
- **Tap-dance на `M`**: 1 нажатие = `M`, 2 = `]` (`Ъ` в русской раскладке)
- **Комбо `Q + E`**: переключение языка (`Win+Space`)

---

## 🐛 Известные проблемы

- **Трекбол Pixart PAW3204** не реализован (нужен кастомный драйвер).
- **Артефакт на экране донгла**: ST7789V захватывает мусор в последней строке → картинка уменьшена до **276×236** и сдвинута `lv_obj_align(..., -12)`.
- **Сброс idle-таймера по BLE**: `brightness_wake_screen_on_reconnect()` вызывал `record_activity_locked(false)` на каждом пакете → вызов закомментирован.

---

## 📝 Лицензия

MIT — как ZMK и YADS.
