---
layout: home

hero:
  name: "rinui-rs"
  text: "使用 Rust 与 QML 构建 Fluent 桌面应用"
  tagline: "结合 Rust 与 Qt Quick，使用开箱即用的控件、主题和导航系统构建桌面应用。"
  image:
    light:
      src: /assets/images/brand/rinui-vertical-light.png
      width: 128
      height: 128
    dark:
      src: /assets/images/brand/rinui-vertical-dark.png
      width: 128
      height: 128
    alt: RinUI
  actions:
    - theme: brand
      text: 快速开始
      link: /zh/guide/getting-started
    - theme: alt
      text: 浏览组件
      link: /zh/BasicInput/Button

features:
  - icon:
      src: /assets/images/home/icons/navigationview.png
      alt: NavigationView
      width: 48
      height: 48
    title: 完整的桌面应用结构
    details: 使用 NavigationView、FluentPage 与标题栏组合出完整的导航、页面与窗口体验。
    link: /zh/Navigation/NavigationView
    linkText: NavigationView
  - icon:
      src: /assets/images/home/icons/colorpicker.png
      alt: ColorPicker
      width: 48
      height: 48
    title: 全局一致的主题
    details: 浅色、深色与自动模式共享同一套颜色、强调色和动效语言。
    link: /zh/core/theme
    linkText: 主题系统
  - icon:
      src: /assets/images/home/icons/button.png
      alt: Button
      width: 48
      height: 48
    title: 熟悉的 QML 控件
    details: 保留熟悉的控件名称，同时提供完整的 Fluent 状态、基础输入、集合与对话框。
    link: /zh/BasicInput/Button
    linkText: 浏览控件
---

## 为什么选择 rinui-rs

rinui-rs 为 Rust 桌面应用提供 Fluent 风格的控件、主题和导航系统。使用 QML 编写界面，通过 Rust 管理应用启动、配置与平台集成。

- **完整的窗口结构**：`FluentWindow`、`NavigationView` 与 `FluentPage` 开箱即可组合使用。
- **原生桌面体验**：Windows 支持 Mica、Acrylic、自定义标题栏和标准窗口控制。
- **统一的设计系统**：控件、主题、图标与翻译资源在应用中协同工作。

<GalleryCarousel section="home" />

## 适合哪些项目

rinui-rs 更适合需要桌面窗口和完整交互、又希望保留 QML 开发效率的项目：

- 使用 Rust 与 Qt Quick 构建的桌面工具和内容管理器。
- 希望采用 Fluent 视觉语言，但不想从零编写窗口框架与控件状态的内部应用。
- 需要导航、设置页、对话框、表格、日期选择和主题切换等常用能力的原型或持续迭代项目。

如果你只需要网页组件，或者项目必须完全依赖 Qt Widgets，rinui-rs 并不是对应场景的替代品。

## 从熟悉的 QML 开始


<div align="center">
  <img src="/assets/images/docs/FluentWindow/example.png" alt="使用 FluentWindow 与 SamplePage 的示例界面">
</div>

```qml
import QtQuick
import RinUI

FluentWindow {
    visible: true
    width: 960
    height: 640

    navigationItems: [
        {
            title: "首页",
            icon: "ic_fluent_home_20_regular",
            page: Qt.resolvedUrl("pages/Home.qml")
        }
    ]
}
```

从 Rust 加载 QML，完整 Cargo 配置见[快速开始](/zh/guide/getting-started)：

```rust
use rinui_host::{AppIdentity, HostConfig, QmlSource, ResourceMode, RinUIHost};
use std::{path::PathBuf, process::ExitCode};

fn run() -> Result<i32, Box<dyn std::error::Error>> {
    let mut host = RinUIHost::new(HostConfig {
        identity: AppIdentity::new("RinUIDocs", "Starter"),
        resource_mode: ResourceMode::Embedded,
        ..HostConfig::default()
    })?;
    let qml = PathBuf::from(env!("CARGO_MANIFEST_DIR")).join("qml/main.qml");
    host.load(&QmlSource::File(qml))?;
    Ok(host.exec())
}

fn main() -> ExitCode {
    match run() {
        Ok(code) => ExitCode::from((code & 0xff) as u8),
        Err(error) => {
            eprintln!("error: {error}");
            ExitCode::FAILURE
        }
    }
}
```

进一步阅读：

- [从零运行 rinui-rs](/zh/guide/getting-started)
- [窗口与页面结构](/zh/Windowing/FluentWindow)
- [NavigationView 导航容器](/zh/Navigation/NavigationView)
- [主题系统与窗口背景](/zh/core/theme)

## 致谢

rinui-rs 由 [unDefFtr](https://github.com/unDefFtr/rinui-rs) 维护。QML 控件及沿用的文档、图片来自 RinLit 与上游 RinUI 贡献者；截图用于展示上游 QML 组件。

[![Contributors](http://contrib.nn.ci/api?repo=rinlit-233-shiroko/Rin-UI)](https://github.com/RinLit-233-shiroko/Rin-UI/graphs/contributors)
