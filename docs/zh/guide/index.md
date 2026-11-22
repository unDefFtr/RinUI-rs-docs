# 什么是 RinUI-rs？

RinUI-rs 是 [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI) 的 Rust Host，支持以 Rust 与 Qt Quick（QML）构建 Fluent 风格桌面应用。

应用界面继续使用 QML，Rust 负责启动、配置、翻译、引擎设置与原生平台集成。独立应用使用 `RinUIHost`；多个使用方或已有 Qt GUI 应用可使用 `RinUIRuntime` 配合 Session。

```qml
import QtQuick
import RinUI
```

## 提供的能力

- 熟悉的 QML 控件：输入、导航、集合、对话框和状态信息。
- 深浅色与自动主题、强调色和 Windows 背景效果。
- 内嵌 RinUI QRC 资源与文件系统开发资源模式。
- Windows、macOS、Linux 平台集成。
- Rust/QML 上下文属性、QObject 后端、带类型的根对象交互与翻译目录。

## 从哪里开始

- [快速开始](getting-started)介绍如何从 crates.io 接入 RinUI.rs，以及如何下载或构建 Gallery。
- [项目结构与资源](project-structure)：导入、资源模式、QRC 与打包。
- [配置与 Rust/QML 交互](rust-host)：配置、主题、翻译和 Rust/QML 交互。
- [Runtime 与 Session](runtime-sessions)：共享／独立引擎与已有 Qt 应用。
- [组件文档](/zh/components/)：QML 属性、信号与使用示例。

## 致谢

QML 控件、主题及沿用的文档与图片来自 RinLit 与贡献者维护的 [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI)。RinUI-rs 由 [unDefFtr](https://github.com/unDefFtr/rinui-rs) 维护。同时感谢 Qt Quick、CXX-Qt、Fluent Design 与 Fluent UI System Icons。保留 MIT 许可证及原作者归属。
