order: 2

## Logging

Quickshell writes logs to the terminal by default. Verbosity is controlled with `-v` flags:

- `-v` enables INFO level internal logs
- `-vv` enables DEBUG level internal logs

Additional options:

- `--no-color` disables colored output (also respects the `NO_COLOR` environment variable)
- `--log-times` prepends a timestamp to each log message
- `--log-rules <rules>` applies Qt logging rules in the `QT_LOGGING_RULES` format, allowing fine-grained filtering by logging category

Example — enabling debug output only for the QML scanner:

```sh
qs --log-rules "quickshell.qmlscanner=true" -vv
```

## QML Debugger

Quickshell supports attaching an external QML debugger over TCP.

### Starting with the debugger enabled

```sh
qs --debug <port>
```

Opens the given port for an incoming debugger connection. The shell starts and runs normally while waiting for a connection.

To block execution until a debugger connects, add `--waitfordebug`:

```sh
qs --debug <port> --waitfordebug
```

This is useful when you need to catch issues that occur at startup before any interaction is possible.

### Connecting Qt Creator's QML Profiler

1. Open Qt Creator and go to **Analyze → QML Profiler (Attach to Waiting Application)**.
2. Enter the port you passed to `--debug`.
3. Click **Connect**.

The profiler will show a timeline of QML engine events, including component creation, binding evaluations, signal handlers, and JavaScript execution. This is the primary tool for identifying what is slow or firing unexpectedly.

If you used `--waitfordebug`, Quickshell will begin executing only after the profiler connects, so the full startup sequence is captured.

## Performance

### Reduce memory usage with Loader and LazyLoader

The most effective way to reduce memory usage is to avoid keeping objects alive when they are not needed. Use `Loader` and `LazyLoader` to create components on demand and destroy them when they are no longer required.

- Use `Loader` for components that inherit from `Item` (visual elements).
- Use `LazyLoader` for everything else (non-visual QML objects).

```qml
// Instead of always having this in the tree:
MyHeavyPopup {}

// Create it only when needed:
Loader {
    active: popupVisible
    sourceComponent: MyHeavyPopup {}
}
```

Setting `active: false` destroys the loaded component and frees its memory. Setting it back to `true` recreates it.

### Avoid excessive repaints

Bindings that change frequently can trigger repaints across multiple windows. Keep animated or rapidly-updating properties isolated to the smallest possible subtree, and avoid binding them to properties that influence layout or visibility of unrelated components.

### DropExpensiveFonts pragma

If font loading is contributing to startup time or memory usage, the [`DropExpensiveFonts`](pragmas.md#dropexpensivefonts) pragma excludes WOFF and WOFF2 web fonts from the Fontconfig font list. See the Pragmas page for details.