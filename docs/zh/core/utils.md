# 辅助工具（`Utils`）

`RinUIHost` 初始化 Rust 主题后端，向 `Utils` 提供持久化颜色。配置方式见 [配置与 Rust/QML 交互](/zh/guide/rust-host)。默认配置强调色为 `#605ed2`，字体与图标仍通过 QML 资源提供。

## 概述

`Utils` 是用于管理全局资源、字体和动画时长的 QML 单例。修改其属性可自定义应用的外观和行为。

您可以像这样使用辅助工具`Utils`：
```qml
import QtQuick
import RinUI

Text {
    font.family: Utils.fontFamily
    font.pixelSize: 14
    text: "rinui-rs"
}
```

## 外观与主题
| 属性           | 	说明                                                                                                                                   |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------|
| primaryColor | 	主题主色，默认读取后端的配置文件默认值 `#605ed2` <br/> 建议通过 [`Theme.setThemeColor(color)`](/zh/core/theme.md#modes-and-accent-color) 修改主题色        |
| colors       | 	当前主题的颜色表，供所有控件引用                                                                                                  |
| appearance   | 	当前主题的外观设定（如圆角、阴影、透明度）                                                                                                                |
| typography   | 	目前仅存储字体的大小信息                                                                                                                         |

## 字体与图标
| 属性                      | 	说明                                                   |
|-------------------------|-------------------------------------------------------|
| `fontFamily`	           | 默认应用字体，默认为`（Qt.application.font.family）   `           |
| `iconFontFamily`        | 	字体图标的字体名（通常由 `FontIconLoader` 提供）                    |
| `fontIconSource`        | 	字体图标`.ttf`文件路径                                       |
| `fontIconIndexSource`   | 	字体图标索引`.js`文件路径                                      |
| `fontIconIndex`	        | 图标索引对象，可直接通过 Utils.fontIconIndex['icon-name'] 获取字符    |
| `dpiScale`              | 当前屏幕的只读 `devicePixelRatio`，仅用于诊断                     |

## 动画时长

| 属性                            | 	说明              |
|-------------------------------|------------------|
| `animationSpeed`              | 标准动画速度（250ms） |
| `animationSpeedExpander`      | 展开器动画速度（333ms） |
| `animationSpeedFaster`        | 快速动画速度（167ms） |
| `animationSpeedMiddle`        | 中等偏慢速度（667ms） |
| `appearanceSpeed`             | 界面切换速度（187ms） |
| `progressBarAnimationSpeed`   | 进度条动画时间（1550ms） |

## 尺寸与 DPI

`Utils.dp(value)` 已弃用。为了兼容旧代码它仍然存在，但现在是恒等函数：传入多少就返回多少，并会在首次调用时输出弃用警告。

Qt Quick 的 `width`、`height`、`font.pixelSize` 等尺寸本身使用逻辑像素，不需要再乘以设备像素比。直接写 QML 尺寸即可：

```qml
Text {
    width: 320
    height: 48
    font.pixelSize: 14
    text: "rinui-rs"
}
```

不要写成 `Utils.dp(320)`，也不要手动乘 `Utils.dpiScale`，否则在 HiDPI 屏幕上会出现重复缩放。
