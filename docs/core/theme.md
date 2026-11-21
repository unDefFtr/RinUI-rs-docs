# Theme

## Initialization

`Theme` is RinUI's QML singleton for appearance settings. `RinUIHost` initializes its backend before loading QML. For a custom engine, see [Runtime and Sessions](/guide/runtime-sessions).

```qml
import QtQuick
import RinUI

Item {
    Component.onCompleted: Theme.setTheme(Theme.mode.Auto)
}
```

## Modes and accent color {#modes-and-accent-color}

| API | Behavior |
| --- | --- |
| `Theme.setTheme(Theme.mode.Light / Dark / Auto)` | Select light, dark or system-following appearance. |
| `Theme.toggleMode()` | Toggle effective light/dark appearance; does not cycle through Auto. |
| `Theme.getTheme()` | Configured mode: `"Light"`, `"Dark"` or `"Auto"`. |
| `Theme.currentTheme.isDark` / `Theme.isDark()` | Current effective appearance, including when Auto is selected. |
| `Theme.setThemeColor(color)` | Set the accent with a hex string or QML color value. |
| `Theme.getThemeColor()` | Current QML color; `"transparent"` before backend initialization. |

`HostConfig.config_location` controls how theme settings are stored. For storage configuration and Rust calls, see [Rust Host integration](/guide/rust-host).

## Windows backdrop effects

Use `Theme.setBackdropEffect(Theme.effect.Mica / Acrylic / Tabbed / None)` after a window is loaded. `Theme.getBackdropEffect()` returns `"mica"`, `"acrylic"`, `"tabbed"` or `"none"`.

| Effect | Platform support |
| --- | --- |
| Mica | Windows 11. |
| Acrylic | Windows 10; requests on Windows 11 21H2 fall back to Mica. |
| Tabbed | Windows 11 22H2+; requests on Windows 11 21H2 fall back to Mica. |
| None | Disables Windows background material. Other platforms use their normal background. |

For Rust backdrop calls and return codes, see [Rust Host integration](/guide/rust-host#windows-backdrops-and-window-configuration).

See [FluentWindow](/Windowing/FluentWindow) and [Project Structure and Resources](/guide/project-structure).
