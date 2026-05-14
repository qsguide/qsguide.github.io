order: 1
title: Simple Bar
sidebar_title: Simple Bar
summary: <a class="button secondary"><small>AI-Assisted</small></a> <small>Some information may be incomplete or inconsistent</small>

This tutorial walks you through building a functional status bar from scratch, beginning with a minimal hello-world window and ending with a multi-monitor bar that displays workspaces, CPU usage, memory, and a live clock.

## Project Structure

Quickshell gets configuration from `~/.config/quickshell/`. The entry point is always `shell.qml`. Any `.qml` file whose name begins with an uppercase letter is automatically exposed as an importable component within that directory.

    :::
    ~/.config/quickshell/mybar/
    ├── shell.qml          # Entry point
    ├── Bar.qml            # Bar component
    ├── ClockWidget.qml    # Clock widget
    └── Theme.qml.json     # Singleton colour/font config


Launch a specific configuration with:

    :::bash
    qs -c mybar
    # or, for a raw file during prototyping:
    qs -p ~/.config/quickshell/mybar/shell.qml


!!! success ""
    <p style="font-size: 1.6em">+lucide:info+ Live Reloading </p></br>

    Quickshell supports hot-reloading. When you save a file Quickshell will attempt to reload the configuration in-place, preserving state where possible.


## Part 1 — Hello World

Every Quickshell config begins with imports. The `Quickshell` module supplies core types; `QtQuick` supplies fundamental UI primitives such as `Text`, `Rectangle`, and `Item`.


/// tab | shell.qml

    :::qml
    import Quickshell
    import QtQuick

    FloatingWindow {
        visible: true
        implicitWidth: 200
        implicitHeight: 100
        color: "#333"

        Text {
            anchors.centerIn: parent
            text: "Hello, Quickshell!"
            color: "#ffaa22"
            font.pixelSize: 18
        }
    }
///

`FloatingWindow` is a standard desktop window, it does not dock to any edge and does not reserve screen space. `anchors.centerIn: parent` is QML's anchoring system, it positions the `Text` in the center of its parent container.


## Part 2 — A Docked Panel

To create a proper bar that docks to a screen edge, replace `FloatingWindow` with `PanelWindow`.

/// tab | shell.qml

    :::qml
    import Quickshell
    import QtQuick

    PanelWindow {
        visible: true
        implicitHeight: 32
        color: "#333"
        anchors {
            top: true
            left: true
            right: true
        }

        Text {
            anchors.centerIn: parent
            text: "My bar"
            color: "#ffaa22"
            font.pixelSize: 18
        }
    }
///

`anchors.top/left/right: true` instructs the compositor to pin the window to the top edge and span the full screen width. Setting `implicitHeight` defines the bar's height. Unlike a floating window, a `PanelWindow` actively reserves space — other windows will not overlap it.

!!! note "Exclusive Zone"
    `PanelWindow` sets an exclusive zone automatically based on the anchored edges. If you want a bar that floats over content without reserving space, set `exclusionMode: ExclusionMode.Ignore`.


## Part 3 — Layouts and Sections

Real bars have distinct sections — typically a left group, a centered group, and a right group. QML provides `RowLayout` from `QtQuick.Layouts` for horizontal arrangement, and an `Item` with `Layout.fillWidth: true` acts as a flexible spacer.

/// tab | shell.qml

    :::qml
    import Quickshell
    import QtQuick
    import QtQuick.Layouts

    PanelWindow {
        id: root
        anchors { top: true; left: true; right: true }
        implicitHeight: 32
        color: "#1e1e2e"

        RowLayout {
            anchors.fill: parent
            anchors.leftMargin: 12
            anchors.rightMargin: 12
            spacing: 8

            // --- Left section ---
            Text {
                text: "Left"
                color: "#cdd6f4"
                font.pixelSize: 14
            }

            // --- Centre spacer ---
            Item { Layout.fillWidth: true }

            // --- Right section ---
            Text {
                text: "Right"
                color: "#cdd6f4"
                font.pixelSize: 14
            }
        }
    }
///

For a true three-column layout with a centred middle group, use three `Item` children where the outer two fill equally:

    :::qml
    RowLayout {
        anchors.fill: parent

        Item {
            Layout.fillWidth: true
            Layout.fillHeight: true
            // Left widgets go here
        }

        Item {
            Layout.fillHeight: true
            implicitWidth: centreRow.implicitWidth
            // Centre widgets go here
            RowLayout { id: centreRow; anchors.centerIn: parent }
        }

        Item {
            Layout.fillWidth: true
            Layout.fillHeight: true
            // Right widgets go here
        }
    }

## Part 4 — Theming with JSON Singletons

Rather than scattering color literals and font names throughout the code, define a singleton theme. Files named `ComponentName.qml.json` are automatically synthesised into QML singletons.

/// tab | Theme.qml.json

    :::json
    {
        "bg":       "#1e1e2e",
        "fg":       "#cdd6f4",
        "muted":    "#585b70",
        "accent":   "#89b4fa",
        "green":    "#a6e3a1",
        "yellow":   "#f9e2af",
        "red":      "#f38ba8",
        "radius":   8,
        "barHeight": 32,
        "fontSize": 14,
        "fontFamily": "JetBrainsMono Nerd Font"
    }
///

/// tab | qml

    :::qml
    color: Theme.bg
    font.pixelSize: Theme.fontSize
///


## Part 5 — Multi-Monitor Support with Variants

By default, a `PanelWindow` appears on the first connected monitor only. To span all connected monitors, wrap the window in a `Variants` component driven by `Quickshell.screens`.

/// tab | shell.qml

    :::qml
    import Quickshell
    import QtQuick

    ShellRoot {
        Variants {
            model: Quickshell.screens

            PanelWindow {
                property var modelData
                screen: modelData

                anchors { top: true; left: true; right: true }
                implicitHeight: Theme.barHeight
                color: Theme.bg

                // bar contents...
            }
        }
    }
///

`Variants` instantiates one copy of its child for each entry in the model. Each instance receives a `modelData` property bound to the corresponding `ShellScreen`. Assigning that to `screen` pins the window to the correct output.


## Part 6 — Running Processes

`Quickshell.Io` provides the `Process` type for running external commands and reading their output. The `stdout` property accepts a parser; `SplitParser` calls `onRead` once per line, while `StdioCollector` buffers the entire output and fires `onStreamFinished`.

### Clock via `date`

    :::qml
    import Quickshell.Io

    // Declare at the top of your PanelWindow or Scope:
    property string currentTime: "--:--"

    Process {
        id: dateProc
        command: ["date", "+%H:%M"]
        stdout: StdioCollector {
            onStreamFinished: currentTime = this.text.trim()
        }
    }

    Timer {
        interval: 1000
        running: true
        repeat: true
        onTriggered: dateProc.running = true
    }

Or make a Singleton like this using the built-in `SystemClock`:

    :::qml
    pragma Singleton

    import Quickshell
    import QtQuick
    import qs.config

    Singleton {
        id: root
        readonly property string date: clock.date

        SystemClock {
            id: clock
            precision: SystemClock.Seconds
        }
    }

### CPU Usage

Reading `/proc/stat` yields raw CPU tick counts. Two successive samples allow computing actual utilisation:

```qml
property int cpuUsage: 0
property var _cpuIdle: 0
property var _cpuTotal: 0

Process {
    id: cpuProc
    command: ["sh", "-c", "head -1 /proc/stat"]
    stdout: SplitParser {
        onRead: data => {
            if (!data) return
            var p = data.trim().split(/\s+/)
            var idle  = parseInt(p[4]) + parseInt(p[5])
            var total = p.slice(1, 8).reduce((a, b) => a + parseInt(b), 0)
            if (_cpuTotal > 0) {
                cpuUsage = Math.round(
                    100 * (1 - (idle - _cpuIdle) / (total - _cpuTotal))
                )
            }
            _cpuTotal = total
            _cpuIdle  = idle
        }
    }
    Component.onCompleted: running = true
}

Timer {
    interval: 2000; running: true; repeat: true
    onTriggered: cpuProc.running = true
}
```

### Memory Usage

```qml
property int memUsage: 0

Process {
    id: memProc
    command: ["sh", "-c", "free | awk '/Mem:/ {print $3/$2 * 100}'"]
    stdout: StdioCollector {
        onStreamFinished: memUsage = Math.round(parseFloat(this.text))
    }
    Component.onCompleted: running = true
}
```

---

## Part 7 — Hyprland Workspaces

`Quickshell.Hyprland` exposes live IPC data from the compositor. `Hyprland.workspaces` is a reactive model; the bar updates automatically as workspaces are created, destroyed, or focused.

```qml
import Quickshell.Hyprland
import QtQuick.Layouts

// Inside your RowLayout:
Repeater {
    model: 9

    Rectangle {
        property int wsId: index + 1
        property var ws: Hyprland.workspaces.values.find(w => w.id === wsId)
        property bool active: Hyprland.focusedWorkspace?.id === wsId

        width: 24; height: 24
        radius: 4
        color: active ? Theme.accent : (ws ? "#313244" : "transparent")

        Text {
            anchors.centerIn: parent
            text: parent.wsId
            color: parent.active ? Theme.bg : (parent.ws ? Theme.fg : Theme.muted)
            font.pixelSize: 12; font.bold: true
        }

        MouseArea {
            anchors.fill: parent
            onClicked: Hyprland.dispatch("workspace " + parent.wsId)
        }
    }
}
```

`Hyprland.dispatch()` sends arbitrary Hyprland dispatcher commands over IPC, making workspace switching, window focus, and layout changes straightforward.

---

## Part 8 — Extracting Widgets into Separate Files

As the bar grows, inline code becomes unwieldy. Move each widget to its own `.qml` file — Quickshell's scanner exposes them automatically.

```qml
// ClockWidget.qml
import QtQuick

Text {
    id: root
    color: Theme.fg
    font.pixelSize: Theme.fontSize
    font.family: Theme.fontFamily
    text: Qt.formatDateTime(new Date(), "ddd dd MMM  HH:mm")

    Timer {
        interval: 1000; running: true; repeat: true
        onTriggered: root.text = Qt.formatDateTime(new Date(), "ddd dd MMM  HH:mm")
    }
}
```

```qml
// CpuWidget.qml
import QtQuick
import Quickshell.Io

Text {
    id: root
    property int usage: 0
    property var _idle: 0
    property var _total: 0

    color: usage > 80 ? Theme.red : Theme.yellow
    font.pixelSize: Theme.fontSize
    font.family: Theme.fontFamily
    text: " " + usage + "%"

    Process {
        id: proc
        command: ["sh", "-c", "head -1 /proc/stat"]
        stdout: SplitParser {
            onRead: data => {
                if (!data) return
                var p = data.trim().split(/\s+/)
                var idle  = parseInt(p[4]) + parseInt(p[5])
                var total = p.slice(1, 8).reduce((a, b) => a + parseInt(b), 0)
                if (root._total > 0)
                    root.usage = Math.round(100 * (1 - (idle - root._idle) / (total - root._total)))
                root._total = total
                root._idle  = idle
            }
        }
        Component.onCompleted: running = true
    }

    Timer { interval: 2000; running: true; repeat: true; onTriggered: proc.running = true }
}
```

In `Bar.qml`, reference these as first-class types:

```qml
// Bar.qml
import Quickshell
import Quickshell.Wayland
import Quickshell.Hyprland
import QtQuick
import QtQuick.Layouts

PanelWindow {
    id: root
    anchors { top: true; left: true; right: true }
    implicitHeight: Theme.barHeight
    color: Theme.bg

    RowLayout {
        anchors.fill: parent
        anchors.leftMargin: 12
        anchors.rightMargin: 12
        spacing: 10

        // Workspaces (inline for brevity)
        Repeater {
            model: 9
            Text {
                property bool active: Hyprland.focusedWorkspace?.id === (index + 1)
                text: index + 1
                color: active ? Theme.accent : Theme.muted
                font.pixelSize: Theme.fontSize; font.bold: true
                MouseArea {
                    anchors.fill: parent
                    onClicked: Hyprland.dispatch("workspace " + (index + 1))
                }
            }
        }

        Item { Layout.fillWidth: true }

        CpuWidget {}

        Rectangle { width: 1; height: 16; color: Theme.muted }

        ClockWidget {}
    }
}
```

`shell.qml` then simply instantiates the bar per-screen:

```qml
// shell.qml
import Quickshell
import Quickshell.Wayland
import QtQuick

ShellRoot {
    Variants {
        model: Quickshell.screens
        Bar { property var modelData; screen: modelData }
    }
}
```

---

## Part 9 — Pragmas

Pragmas are scanner-level directives that configure Quickshell before the QML engine initialises. They must appear before any `import` statement.

```qml
//@ pragma NativeTextRendering
//@ pragma ShellId mybar
//@ pragma Env QT_SCALE_FACTOR = 1

import Quickshell
```

Relevant pragmas for bar development:

| Pragma | Effect |
|---|---|
| `NativeTextRendering` | Enables native font hinting — improves legibility at small sizes |
| `ShellId <id>` | Fixes the run/state/cache directory name instead of deriving it from a hash |
| `Env VAR = VALUE` | Sets an environment variable before the engine starts |
| `IgnoreSystemSettings` | Prevents Qt from reading system fonts or colour settings |
| `DropExpensiveFonts` | Skips WOFF/WOFF2 web fonts — reduces startup time on font-heavy systems |

---

## Complete Minimal Bar

The following self-contained file produces a working top bar with a live clock and no external dependencies beyond Quickshell itself.

```qml
//@ pragma NativeTextRendering
//@ pragma ShellId minimal-bar

import Quickshell
import Quickshell.Wayland
import QtQuick
import QtQuick.Layouts

ShellRoot {
    Variants {
        model: Quickshell.screens

        PanelWindow {
            property var modelData
            screen: modelData

            anchors { top: true; left: true; right: true }
            implicitHeight: 30
            color: "#1e1e2e"

            RowLayout {
                anchors.fill: parent
                anchors.leftMargin: 12
                anchors.rightMargin: 12
                spacing: 8

                Text {
                    text: "󱄅 myshell"
                    color: "#89b4fa"
                    font.pixelSize: 13
                    font.bold: true
                }

                Item { Layout.fillWidth: true }

                Text {
                    id: clock
                    color: "#cdd6f4"
                    font.pixelSize: 13
                    text: Qt.formatDateTime(new Date(), "ddd dd MMM   HH:mm")
                    Timer {
                        interval: 1000; running: true; repeat: true
                        onTriggered: clock.text = Qt.formatDateTime(
                            new Date(), "ddd dd MMM   HH:mm"
                        )
                    }
                }
            }
        }
    }
}
```

---

## Quick Reference

| Type | Module | Purpose |
|---|---|---|
| `PanelWindow` | `Quickshell` | Layer-shell docked window |
| `FloatingWindow` | `Quickshell` | Standard floating window |
| `ShellRoot` | `Quickshell` | Top-level container (preferred over bare `Scope`) |
| `Variants` | `Quickshell` | Instantiates a component per model entry |
| `Process` | `Quickshell.Io` | Runs external commands |
| `SplitParser` | `Quickshell.Io` | Parses stdout line-by-line |
| `StdioCollector` | `Quickshell.Io` | Buffers full stdout |
| `Hyprland` | `Quickshell.Hyprland` | Hyprland IPC singleton |
| `RowLayout` | `QtQuick.Layouts` | Horizontal layout |
| `Timer` | `QtQuick` | Interval-based trigger |
| `MouseArea` | `QtQuick` | Input event handler |

---

## Further Reading

- [Quickshell Official Documentation](https://quickshell.org/docs/master/guide/)
- [QML Language Overview](https://quickshell.org/docs/master/guide/qml-language/)
- [Item Size and Position](https://quickshell.org/docs/master/guide/size-position/)
- [Quickshell Type Reference](https://quickshell.org/docs/master/types/)
- [QsGuide](https://qsguide.github.io) — Community-maintained guides and examples