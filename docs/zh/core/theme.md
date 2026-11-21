# 主题系统（`Theme`）

## 初始化

`Theme` 是 RinUI 用于管理外观的 QML 单例。`RinUIHost` 会在加载 QML 前初始化其后端。自定义引擎的接入方式见 [Runtime 与 Session](/zh/guide/runtime-sessions)。

```qml
import QtQuick
import RinUI

Item {
    Component.onCompleted: Theme.setTheme(Theme.mode.Auto)
}
```

## 主题模式与强调色 {#modes-and-accent-color}

| API | 行为 |
| --- | --- |
| `Theme.setTheme(Theme.mode.Light / Dark / Auto)` | 选择浅色、深色或跟随系统。 |
| `Theme.toggleMode()` | 按实际外观切换深浅色，不在 Auto 等三种模式之间轮换。 |
| `Theme.getTheme()` | 配置模式：`"Light"`、`"Dark"` 或 `"Auto"`。 |
| `Theme.currentTheme.isDark` / `Theme.isDark()` | 实际外观，Auto 模式下也可使用。 |
| `Theme.setThemeColor(color)` | 以十六进制字符串或 QML color 值设置强调色。 |
| `Theme.getThemeColor()` | 当前 QML color，后端未初始化时返回 `"transparent"`。 |

`HostConfig.config_location` 控制主题设置的存储方式。存储配置与 Rust 调用方式见 [配置与 Rust/QML 交互](/zh/guide/rust-host)。

## Windows 背景效果

在窗口加载后调用 `Theme.setBackdropEffect(Theme.effect.Mica / Acrylic / Tabbed / None)`。`Theme.getBackdropEffect()` 返回 `"mica"`、`"acrylic"`、`"tabbed"` 或 `"none"`。

| 效果 | 平台支持 |
| --- | --- |
| Mica | Windows 11。 |
| Acrylic | Windows 10；Windows 11 21H2 请求时回退为 Mica。 |
| Tabbed | Windows 11 22H2+；Windows 11 21H2 请求时回退为 Mica。 |
| None | 关闭 Windows 背景材质；其他平台使用正常背景。 |

Rust 背景效果的调用方式与返回码见 [配置与 Rust/QML 交互](/zh/guide/rust-host#windows-背景与窗口配置)。

参见 [FluentWindow](/zh/Windowing/FluentWindow) 和[项目结构与资源](/zh/guide/project-structure)。
