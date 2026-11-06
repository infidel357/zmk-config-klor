<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/docs/images/klor-font-logo-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="/docs/images/klor-font-logo-bright.svg">
  <img alt="KLOR logo font" src="/docs/images/klor-font-logo-bright.svg">
</picture>

---

```markdown
# KLOR — ZMK config (fork with dongle screen & ZMK Studio)

ZMK configuration for the **KLOR** split keyboard, extended with a
**Seeed XIAO BLE dongle** running a custom **YADS (zmk-dongle-screen)** build:
idle pixel-art face, ZMK Studio support, and correct KLOR layout in Studio.

- Original QMK config: [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR)
- Hardware files & build guide: [GEIGEIGEIST/KLOR-hardware](https://github.com/GEIGEIGEIST/KLOR)
- Upstream dongle screen: [janpfischer/zmk-dongle-screen](https://github.com/janpfischer/zmk-dongle-screen)

---

## ✨ What's different from upstream KLOR

### 1. Dongle with screen (YADS fork)

- Board: `seeeduino_xiao_ble`
- Shield: `klor_dongle dongle_screen`
- Snippet: `studio-rpc-usb-uart`
- Fork of `zmk-dongle-screen`: **`infidel357/zmk-dongle-screen`**

### 2. Idle pixel-art face

After **20 s** of inactivity a yellow pixel-art face appears, stays for
**30 s**, then the screen fades to black.

State machine in `brightness.c`:

```
ACTIVE  ──(20 s idle)──▶  IDLE_IMAGE  ──(30 s)──▶  BLANKED
   ▲                          │
   └────── activity ──────────┘
```

New files in the dongle-screen fork:

- `src/assets/idle_face.c` — 276×236 `LV_IMG_CF_INDEXED_1BIT` (yellow + black)
- `src/widgets/custom_idle_image.c` / `.h` — LVGL widget

Kconfig options:

```kconfig
CONFIG_DONGLE_SCREEN_IDLE_TIMEOUT_S=20
CONFIG_DONGLE_SCREEN_IDLE_IMAGE=y
CONFIG_DONGLE_SCREEN_IDLE_IMAGE_BLANK_DELAY_S=30
```

### 3. ZMK Studio

Enabled on the dongle, **locking disabled** — no `&studio_unlock` needed:

```kconfig
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
```

### 4. KLOR in ZMK Studio

`klor_dongle.overlay` contains a `physical_layout0` with a 44-element `keys`
array, so thumb clusters are drawn **angled** (taken from `vial.json`).

---

## 🚀 How to build

1. **Fork** this repo.
2. **Clone** your fork locally.
3. Edit `config/klor.keymap` (keycodes: [ZMK docs](https://zmk.dev/docs/codes)).
4. **Push** to your fork.
5. On GitHub → **Actions** → wait for the build → download `firmware.zip`.
6. Flash:
   - **Left half** → `klor_left-nice_nano_v2-zmk.uf2`
   - **Right half** → `klor_right-nice_nano_v2-zmk.uf2`
   - **Dongle** → `klor_dongle-seeeduino_xiao_ble-zmk.uf2`
     (reset twice → drag'n'drop; **do a full settings reset on first flash**)
7. Pair halves and host with the dongle; open ZMK Studio — no unlock combo needed.

> ⚠️ The dongle build uses `snippet: studio-rpc-usb-uart` and
> `cmake-args: -DCONFIG_ZMK_STUDIO=y` — see `config/build.yaml`.

---

## 📁 Repo layout (modified parts)

```
config/
├── west.yml                     # dongle-screen → infidel357 fork
├── build.yaml                   # dongle: snippet + studio cmake-args
├── klor_dongle.conf             # idle image, studio, BLE, LVGL pool
├── klor.keymap                  # 5 layers, encoder overrides, tap-dance, combo
└── boards/shields/klor/
    └── klor_dongle.overlay      # physical_layout0 (angled thumbs for Studio)
```

Dongle-screen fork (`infidel357/zmk-dongle-screen`) adds:

```
boards/shields/dongle_screen/
├── src/assets/idle_face.c
├── src/widgets/custom_idle_image.{c,h}
├── src/brightness.c             # ACTIVE → IDLE_IMAGE → BLANKED
├── src/custom_status_screen.c   # widget init
├── CMakeLists.txt
└── Kconfig.defconfig
```

---

## ⌨️ Keymap highlights

- **Layers**: `BASE`, `SYMBOLS`, `NUMBERS`, `ADJUST`, `EXTRA`
- **Left encoder** (all layers): `C_VOL_UP` / `C_VOL_DN`
- **Right encoder on BASE**: `C_NEXT` / `C_PREV` (next track first)
- **Right encoder on other layers**: `&kp LEFT` / `&kp RIGHT` (cursor)
- **Tap-dance on `M`**: 1 tap = `M`, 2 taps = `]` (`Ъ` in RU layout)
- **Combo `Q + E`**: language switch (`Win+Space`)

---

## 🐛 Known issues

- **Left/right halves**: encoder on the *secondary* half doesn't work — ZMK limitation.
- **Pixart PAW3204 trackball**: not implemented (needs a custom driver).
- **Dongle screen artifact**: ST7789V grabs garbage in the last line → image
  reduced to **276×236** and shifted with `lv_obj_align(..., -12)`.
- **Idle timer reset by BLE**: `brightness_wake_screen_on_reconnect()`
  called `record_activity_locked(false)` on every packet → call is commented out.

---

## 🛠️ Rebuilding the dongle from scratch

1. Fork `janpfischer/zmk-dongle-screen` → `infidel357/zmk-dongle-screen`.
2. Apply the dongle-screen changes listed above.
3. In `config/west.yml` point the remote to `infidel357`.
4. Add the Kconfig block to `klor_dongle.conf`.
5. Add `physical_layout0` to `klor_dongle.overlay`.
6. Add `snippet` + `cmake-args` for the dongle in `build.yaml`.
7. Push → GitHub Actions → flash with settings reset.

---

## 📝 License

MIT — same as ZMK and YADS.
```

---

### Что я поменял по сравнению с оригиналом

| Было | Стало |
|---|---|
| «Here you can find the QMK config…» | Явные ссылки + список отличий форка |
| Только 2 файла прошивки (left/right) | Добавлен **dongle** и предупреждение о сбросе настроек |
| Нет упоминания ZMK Studio | Отдельный раздел + `CONFIG_ZMK_STUDIO_LOCKING=n` |
| Нет описания раскладки | Секция **Keymap highlights** |
| Known issues только про энкодер и трекбол | Добавлены артефакт экрана и сброс idle-таймера |
| Нет структуры репо | Дерево `config/` + форк донгла |
| Нет инструкции «с нуля» | Раздел **Rebuilding the dongle** |

Если хочешь — могу ещё:
- добавить **бейджи** (build status, ZMK version, license),
- сделать **английскую + русскую** версию в одном файле (`README.md` / `README.ru.md`),
- вынести картинку idle-мордочки в README как превью (нужен файл `docs/idle_face.png`).
