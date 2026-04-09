title: Pragmas
order: 1

# Pragmas

Pragmas are special directives embedded as comments at the top of a Quickshell config file. They are read by the launcher before the QML engine initialises and allow you to control low-level runtime behaviour — application identity, environment variables, directory overrides, rendering options, and more.

A pragma line takes the form:

```qml
//@ pragma <PragmaName> [value]
```

Pragma scanning stops at the first `import` statement, so all pragmas must appear before any imports.


## Reference

### `UseQApplication`

```qml
//@ pragma UseQApplication
```

Instantiates a full `QApplication` instead of the default `QGuiApplication`. Required if you use widgets or any Qt module that depends on `QApplication`.

### `NativeTextRendering`

```qml
//@ pragma NativeTextRendering
```

Switches the Qt Quick text render type to `NativeTextRendering`. May improve font rendering fidelity on some setups, particularly with subpixel hinting, at the cost of reduced scalability.

### `IgnoreSystemSettings`

```qml
//@ pragma IgnoreSystemSettings
```

Disables `QGuiApplication::setDesktopSettingsAware`. Quickshell will not read platform-level desktop settings (fonts, colours, etc.) from the system.

### `RespectSystemStyle`

```qml
//@ pragma RespectSystemStyle
```

By default, Quickshell unsets `QT_STYLE_OVERRIDE` and forces `QT_QUICK_CONTROLS_STYLE=Fusion` for consistency. This pragma disables that override, allowing the system Qt style to apply.

### `DropExpensiveFonts`

```qml
//@ pragma DropExpensiveFonts
```

Writes a temporary Fontconfig override that excludes WOFF and WOFF2 web fonts from the font list. Useful for reducing startup time and memory usage on systems with large font collections. Can also be triggered via the environment variable `QS_DROP_EXPENSIVE_FONTS=1`.

### `IconTheme <name>`

```qml
//@ pragma IconTheme hicolor
```

Sets the Qt icon theme via `QIcon::setThemeName`. Overrides the `QS_ICON_THEME` environment variable if both are present.

### `AppId <id>`

```qml
//@ pragma AppId org.example.myshell
```

Sets the desktop application ID passed to `QGuiApplication::setDesktopFileName`. Defaults to `org.quickshell` if not specified, or to the `QS_APP_ID` environment variable.

### `ShellId <id>`

```qml
//@ pragma ShellId my-shell
```

Overrides the shell identifier used for run/state/cache directory namespacing. By default the shell ID is derived from an MD5 hash of the config file path.

### `DataDir <path>`

```qml
//@ pragma DataDir ~/.local/share/myshell
```

Overrides the base data directory used by `QsPaths`. Accepts an absolute path.

### `StateDir <path>`

```qml
//@ pragma StateDir ~/.local/state/myshell
```

Overrides the base state directory used by `QsPaths`.

### `CacheDir <path>`

```qml
//@ pragma CacheDir ~/.cache/myshell
```

Overrides the base cache directory used by `QsPaths`.

### `Env <VAR> = <VALUE>`

```qml
//@ pragma Env MY_VAR = hello
//@ pragma Env WAYLAND_DISPLAY = wayland-1
```

Sets an environment variable before the QML engine starts. Multiple `Env` pragmas are allowed. The variable name and value are trimmed of surrounding whitespace. The separator must be `=`.

## Environment variable interactions

Several pragmas have corresponding environment variables that serve as fallbacks:

| Pragma | Environment variable |
|---|---|
| `IconTheme` | `QS_ICON_THEME` |
| `AppId` | `QS_APP_ID` |
| `DropExpensiveFonts` | `QS_DROP_EXPENSIVE_FONTS=1` |

When both a pragma and an environment variable are set, the pragma takes precedence (except for `IconTheme`, where the pragma overrides the env var entirely via `QIcon::setThemeName`).