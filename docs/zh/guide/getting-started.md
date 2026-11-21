# 快速开始

## 运行 rinui-rs Gallery

运行 rinui-rs Gallery，浏览控件并体验交互示例。

<GalleryCarousel section="guide" :show-copy="false" />

## 构建环境

安装 Rust（MSRV **1.85**）、C++17 工具链，以及包含 Qt Quick、Quick Controls 2、**Qt5Compat** 和 **QtShaderTools** 的 **Qt 6**（CI 验证版本：**6.12.0**）。

- **Windows：** MSVC Build Tools 2022、Rust MSVC 工具链和架构匹配的 Qt。在开发者终端执行 Cargo；按 Qt 要求在 `CXXFLAGS` 中加入 `/Zc:__cplusplus /permissive-`。
- **macOS：** Xcode Command Line Tools。Apple Silicon + Xcode 26 搭配 Qt 低于 6.11.1 时，在 `CXXFLAGS` 中追加 `-include arm_acle.h`。
- **Linux：** GCC/Clang、Qt 所需的 X11/Wayland 开发依赖，以及 `lld`、`gold` 或 `mold` 链接器；为 Rust 工具链配置所选链接器。

若 Qt 6 的 `qmake` 不在 `PATH` 中，设置 `QMAKE` 指向实际路径：

::: code-group

```bash [macOS / Linux]
export QMAKE="/path/to/Qt/bin/qmake"
"$QMAKE" --version
```

```powershell [Windows]
$env:QMAKE = "C:\Qt\6.12.0\msvc2022_64\bin\qmake.exe"
& $env:QMAKE --version
```

:::

克隆仓库及子模块，然后在仓库根目录运行：

```bash
git clone --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

已有仓库请执行 `git submodule update --init --recursive`。`--dev` 在启动时读取文件系统 QML，不提供自动热重载；自定义路径见[资源模式](project-structure#resource-modes)。

macOS 默认使用 QML 自绘窗口控件，追加 `--upstream-mac-frame` 可启用原生标题栏。其他 Gallery 参数可通过 `cargo run -p rinui-gallery -- --help` 查看。

## 创建最小 Rust 应用

本教程通过源码路径依赖接入。将应用放在完整的 `rinui-rs` 仓库旁边，仓库需包含原生源码和已初始化的子模块。

```text
projects/
├── rinui-rs/                 # complete checkout, including upstream/Rin-UI
└── rinui-starter/
    ├── Cargo.toml
    ├── src/main.rs
    └── qml/main.qml
```

创建以下三个文件：

::: code-group

```toml [Cargo.toml]
[package]
name = "rinui-starter"
version = "0.1.0"
edition = "2021"
rust-version = "1.85"

[dependencies]
rinui-host = { path = "../rinui-rs/crates/rinui-host" }
```

```rust [src/main.rs]
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

```qml [qml/main.qml]
import QtQuick
import RinUI

FluentWindow {
    visible: true
    width: 640
    height: 480
    title: qsTr("rinui-rs")

    Button {
        anchors.centerIn: parent
        text: qsTr("Toggle theme")
        onClicked: Theme.toggleMode()
    }
}
```

:::

在 `rinui-starter/` 中运行：

```bash
cargo run
```

点击按钮可切换深浅色外观。宿主负责创建 `QGuiApplication`、将 Qt Quick Controls 配置为 `Basic` 样式、注册 RinUI 资源和 `ThemeManager`，随后进入 Qt 事件循环。宿主应在 GUI 主线程上存活至事件循环返回。

`CARGO_MANIFEST_DIR` 路径适用于源码开发；发布时需要额外打包应用自身的 QML 和资源，见[项目结构与资源](project-structure)。

## 继续阅读

- [配置与 Rust/QML 交互](rust-host)：配置、主题、翻译和 Rust/QML 交互。
- [Runtime 与 Session](runtime-sessions)：多引擎与已有 Qt 应用接入。
- [FluentWindow](/zh/Windowing/FluentWindow)：窗口与导航控件。
