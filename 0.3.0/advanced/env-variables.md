title: Env Variables
order: 2


## Environment Variables

| Environment variable | Description |
|---|---|
| [`QS_ICON_THEME`](#qs_icon_theme) | `Changes the default Qt icon theme` |
| [`QS_APP_ID`](#qs_app_id) | `Sets the desktop application ID` |
| [`QS_DROP_EXPENSIVE_FONTS`](#qs_drop_expensive_fonts) | `Disables WOFF/WOFF2 font loading through a temporary Fontconfig override` |
| [`QS_DISABLE_CRASH_HANDLER`](#qs_disable_crash_handler) | `Disables the crash handler` |
| [`QT_QUICK_CONTROLS_STYLE`](#qt_quick_controls_style) | `Overrides the Qt Quick Controls style` |
| [`QT_STYLE_OVERRIDE`](#qt_style_override) | `Overrides the Qt widget style` |
| [`FONTCONFIG_FILE`](#fontconfig_file) | `Overrides the active Fontconfig configuration file` |
| [`XDG_DATA_DIRS`](#xdg_data_dirs) | `Used to populate icon fallback search paths from <dir>/pixmaps` |
| [`QSG_USE_SIMPLE_ANIMATION_DRIVER`](#qsg_use_simple_animation_driver) | `Enables Qt's simple animation driver (currently not forced by Quickshell)` |

## Descriptions

### `QS_ICON_THEME`

Sets the default Qt icon theme used by Quickshell.

Equivalent to:

```bash
//@ pragma IconTheme <name>
```

Example:

```bash
export QS_ICON_THEME=Papirus-Dark
```

If the `IconTheme` pragma is present, it overrides this environment variable.

---

### `QS_APP_ID`

Sets the desktop application ID passed to:

```cpp
QGuiApplication::setDesktopFileName(...)
```

Example:

```bash
export QS_APP_ID=org.example.myshell
```

Equivalent to:

```qml
//@ pragma AppId org.example.myshell
```

If the `AppId` pragma is present, it overrides this environment variable.

---

### `QS_DROP_EXPENSIVE_FONTS`

Disables loading WOFF and WOFF2 fonts through a generated temporary Fontconfig override.

Useful for improving startup performance and reducing memory usage on systems with very large font collections.

Example:

```bash
export QS_DROP_EXPENSIVE_FONTS=1
```

Equivalent to:

```qml
//@ pragma DropExpensiveFonts
```

When enabled, Quickshell creates a temporary Fontconfig file and sets `FONTCONFIG_FILE` internally to point to it.

---

### `QS_DISABLE_CRASH_HANDLER`

Disables Quickshell's crash handler subsystem.

Example:

```bash
export QS_DISABLE_CRASH_HANDLER=1
```

When enabled, Quickshell skips crash handler initialisation entirely.

---

### `QT_QUICK_CONTROLS_STYLE`

Controls the Qt Quick Controls style used by Qt applications.

By default, Quickshell forces:

```bash
QT_QUICK_CONTROLS_STYLE=Fusion
```

unless the following pragma is present:

```qml
//@ pragma RespectSystemStyle
```

Example:

```bash
export QT_QUICK_CONTROLS_STYLE=Material
```

---

### `QT_STYLE_OVERRIDE`

Overrides the Qt widget style.

By default, Quickshell unsets this variable to ensure style consistency.

This behaviour is disabled by:

```qml
//@ pragma RespectSystemStyle
```

Example:

```bash
export QT_STYLE_OVERRIDE=kvantum
```

---

### `FONTCONFIG_FILE`

Overrides the Fontconfig configuration file used by font discovery.

When `QS_DROP_EXPENSIVE_FONTS=1` or `//@ pragma DropExpensiveFonts` is enabled, Quickshell generates a temporary Fontconfig file and sets this variable automatically.

Example:

```bash
export FONTCONFIG_FILE=/etc/fonts/fonts.conf
```

---

### `XDG_DATA_DIRS`

Defines additional shared data lookup directories.

Quickshell scans these directories for:

```text
<prefix>/pixmaps
```

and adds them to Qt's fallback icon search paths.

If unset, Quickshell falls back to:

```text
/usr/local/share
/usr/share
```

Example:

```bash
export XDG_DATA_DIRS=/usr/local/share:/usr/share
```

---

### `QSG_USE_SIMPLE_ANIMATION_DRIVER`

Enables Qt's simple animation driver.

This can improve animation smoothness in some cases, but may also significantly increase GPU usage and repaint frequency.

Quickshell currently does not force-enable this variable.

Example:

```bash
export QSG_USE_SIMPLE_ANIMATION_DRIVER=1
```