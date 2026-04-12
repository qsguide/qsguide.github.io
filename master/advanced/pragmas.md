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

### `Singleton`
 
```qml
pragma Singleton
```
 
Marks a QML component as a singleton. Unlike the other pragmas on this page, this uses the standard QML `pragma` keyword rather than the `//@ pragma` comment syntax. Singleton components have a single shared instance accessible across the entire shell.

!!! note ""
    <p style="font-size: 1.3em">+lucide:badge-info+ Note </p></br>
    `.qml.json` files are automatically treated as singletons — see [JSON Singletons](#json-singletons) below.

### `Internal`

```qml
//@ pragma Internal
```

Marks a QML component as internal to its module, preventing it from being imported by files outside of that module. Useful for encapsulating implementation details that are not part of a module's public API.

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

## Preprocessor directives
 
In addition to pragmas, the scanner supports a simple conditional compilation system using comment directives. These are evaluated before the QML engine sees the file — masked-out blocks are replaced with commented lines so line numbers are preserved.
 
### `//@ if <expression>`
 
```qml
//@ if <expression>
```
 
Evaluates `<expression>` as JavaScript. If the result is falsy, all lines up to the matching `//@ endif` are masked (replaced with `// MASKED: ...` comments). Expressions are evaluated against a `PreprocEnv` object that exposes environment-specific values.
 
Directives can be nested. An inner `//@ if` that passes cannot unmask lines already masked by an outer `//@ if` that failed.
 
```qml
//@ if isEnvSet("WAYLAND_DISPLAY")
import Quickshell.Wayland
//@ endif
```
 
The following functions are available inside `//@ if` expressions:
 
#### `env(variable: string): string`
 
Returns the value of an environment variable, or an empty string if it is not set.
 
```qml
//@ if env("XDG_SESSION_TYPE") === "wayland"
```
 
#### `isEnvSet(variable: string): bool`
 
Returns `true` if the given environment variable is set, regardless of its value.
 
```qml
//@ if isEnvSet("HYPRLAND_INSTANCE_SIGNATURE")
```
 
#### `hasVersion(major: int, minor: int, features?: string[]): bool`
 
Returns `true` if the running Quickshell build meets the given version requirement. An optional list of feature strings can be passed to additionally require that specific build-time features are enabled.
 
```qml
//@ if hasVersion(0, 2)
//@ if hasVersion(0, 2, ["wayland", "hyprland"])
```
 
### `//@ endif`
 
```qml
//@ endif
```
 
Closes the most recent `//@ if` block. Every `//@ if` must have a corresponding `//@ endif`; an unclosed block or an unmatched `//@ endif` is reported as a scan error.
 

## JSON Singletons
 
Files named `ComponentName.qml.json` are automatically synthesised into QML singleton components at scan time. No `pragma Singleton` or `//@ pragma` directives are required.
 
The JSON object is converted to a `QtObject` with typed `readonly property` declarations. Type mapping is as follows:
 
| JSON type | QML type |
|---|---|
| Object | `QtObject` (nested) |
| Array | `var` |
| String matching `#RGB`, `#RRGGBB`, or `#RRGGBBAA` | `color` |
| Other string | `string` |
| Number (whole) | `int` |
| Number (fractional) | `real` |
| Boolean | `bool` |
| Null | `var` (null) |
 
Example — `Theme.qml.json`:
 
```json
{
  "accent": "#4a90d9",
  "radius": 8,
  "fontFamily": "Inter"
}
```
 
This is equivalent to:
 
```qml
pragma Singleton
import QtQuick as Q
 
Q.QtObject {
  readonly property color accent: "#4a90d9";
  readonly property int radius: 8;
  readonly property string fontFamily: "Inter";
}
```