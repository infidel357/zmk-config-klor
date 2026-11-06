Вот английская версия вашего README. Я сохранил структуру, логотип с поддержкой темной/светлой темы и все ваши технические детали.

Скопируйте этот текст в файл **`README.md`** (если он должен быть основным) или **`README.en.md`** (если он дополнительный).

***

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/docs/images/klor-font-logo-dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="/docs/images/klor-font-logo-bright.svg">
  <img alt="KLOR logo font" src="/docs/images/klor-font-logo-bright.svg">
</picture>

***

[🇷🇺 Русский](README.ru.md) | 🇬🇧 English

# KLOR — ZMK config (fork with dongle screen & ZMK Studio)

ZMK configuration for the **KLOR** split keyboard, extended with a **Seeed XIAO BLE dongle** running a custom **YADS (zmk-dongle-screen)** build: idle image, ZMK Studio support, and correct KLOR layout rendering in Studio.

- Original QMK config: [GEIGEIGEIST/KLOR](https://github.com/GEIGEIGEIST/KLOR)
- Hardware files & build guide: [GEIGEIGEIST/KLOR-hardware](https://github.com/GEIGEIGEIST/KLOR-hardware)
- Upstream dongle-screen: [janpfischer/zmk-dongle-screen](https://github.com/janpfischer/zmk-dongle-screen)

---

## ✨ What's different from upstream KLOR

### 1. Dongle with screen (YADS fork)

- Board: `seeeduino_xiao_ble`
- Shield: `klor_dongle dongle_screen`
- Snippet: `studio-rpc-usb-uart`
- Fork of `zmk-dongle-screen`: **[infidel357/zmk-dongle-screen](https://github.com/infidel357/zmk-dongle-screen)**

### 2. Idle image

After **20 seconds** of inactivity, a yellow pixel-art face appears, stays for **30 seconds**, then the screen fades to black.

State machine in `brightness.c`:

```text
ACTIVE  ──(20s idle)──▶  IDLE_IMAGE  ──(30s)──▶  BLANKED
   ▲                        │
   └─────── activity ───────┘
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

Enabled on the dongle, **locking disabled** — no `&studio_unlock` combo needed:

```kconfig
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
```

### 4. KLOR in ZMK Studio

`klor_dongle.overlay` includes a `physical_layout0` with a 44-element `keys` array — thumb clusters are rendered **angled** (taken from `vial.json`).

---

## 🚀 How to build and flash

### Step 1. Fork and clone

1. Click **Fork** on this repository page (create your own copy).
2. Clone the fork locally:

```bash
git clone https://github.com/<your-username>/zmk-config-klor.git
cd zmk-config-klor
```

### Step 2. Edit keymap (optional)

Edit `config/klor.keymap` — find keycodes in the [ZMK documentation](https://zmk.dev/docs/codes).

### Step 3. Build via GitHub Actions

1. Commit and push your changes:

```bash
git add .
git commit -m "my keymap"
git push
```

2. Open the **Actions** tab on your fork's page.
3. Wait for the build to finish (green checkmark).
4. Download the **`firmware.zip`** artifact (button at the bottom of the build page).

> 💡 If Actions didn't start automatically — go to **Actions → Build → Run workflow**.

### Step 4. Flashing

The `firmware.zip` archive contains three files:

| File | Where to flash |
|---|---|
| `klor_left-nice_nano_v2-zmk.uf2` | left half |
| `klor_right-nice_nano_v2-zmk.uf2` | right half |
| `klor_dongle-seeeduino_xiao_ble-zmk.uf2` | dongle (XIAO BLE) |

**Flashing order:**

1. **Left half**: connect to PC, press **Reset** twice — a USB drive will appear. Drag `klor_left-...uf2` onto it.
2. **Right half**: same with `klor_right-...uf2`.
3. **Dongle**: connect XIAO BLE, press **Reset** twice, drag `klor_dongle-...uf2`.

> ⚠️ **Important for the dongle:** on the first flash, be sure to perform a **settings reset**.

### Step 5. Pairing

1. Turn on the dongle — status will appear on the screen.
2. Turn on the left and right halves — they will automatically pair with the dongle.
3. Connect the dongle to the PC via USB — the keyboard will be recognized as a HID device.
4. Open **ZMK Studio** ([studio.zmk.dev](https://studio.zmk.dev)) — the layout is editable **without** `&studio_unlock`.

### Step 6. Verification

- ✅ After 20s idle the face appears, stays 30s, screen turns off.
- ✅ Left encoder — volume, right encoder — track/cursor per layer.
- ✅ Tap-dance on `M`, combo `Q+E` switches language.
- ✅ ZMK Studio opens and shows KLOR with angled thumbs.

---

## ⌨️ Keymap

- **Layers**: `BASE`, `SYMBOLS`, `NUMBERS`, `ADJUST`, `EXTRA`
- **Left encoder** (all layers): `C_VOL_UP` / `C_VOL_DN`
- **Right encoder on BASE**: `C_NEXT` / `C_PREV` (track forward first)
- **Right encoder on other layers**: `&kp LEFT` / `&kp RIGHT` (cursor)
- **Tap-dance on `M`**: 1 tap = `M`, 2 taps = `]` (`Ъ` in RU layout)
- **Combo `Q + E`**: language switch (`Win+Space`)

---

## 🐛 Known issues

- **Pixart PAW3204 trackball** not implemented (needs a custom driver).
- **Dongle screen artifact**: ST7789V grabs garbage in the last line → image reduced to **276×236** and shifted with `lv_obj_align(..., -12)`.
- **Idle timer reset by BLE**: `brightness_wake_screen_on_reconnect()` called `record_activity_locked(false)` on every packet → call is commented out.

---

## 📝 License

MIT — same as ZMK and YADS.
