# 快速开始

使用 crates.io 上的 `rinui-host` 创建第一个 RinUI-rs 应用。若要体验桌面演示，请参阅 [RinUI-rs Gallery](#gallery-downloads)。

## 构建环境

安装 Rust（MSRV **1.88**）、C++17 工具链，以及包含 Qt Quick、Quick Controls 2、**Qt5Compat** 和 **QtShaderTools** 的 **Qt 6** 开发环境。发布构建和 CI 使用 Qt **6.12.0**。

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


## 创建最小 Rust 应用

应用只需添加 [rinui-host](https://crates.io/crates/rinui-host/0.1.0-alpha.2) 作为依赖。crate 自带原生桥接代码和固定版本的 RinUI QML。可以在任意目录创建应用，无需克隆仓库或初始化子模块。

```text
rinui-starter/
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
rust-version = "1.88"

[dependencies]
rinui-host = "0.1.0-alpha.2"
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
    title: qsTr("RinUI-rs")

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

## 运行 RinUI-rs Gallery {#gallery-downloads}

下载 Gallery 即可浏览控件和交互示例。预编译包内含 Qt 6.12.0 和所需的 QML 模块。[v0.1.0-alpha.2 发布页](https://github.com/unDefFtr/rinui-rs/releases/tag/v0.1.0-alpha.2)

| 平台 | 下载 | 运行方式 |
| --- | --- | --- |
| Windows x64 | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64.zip) | 完整解压未签名的 ZIP，再运行 `rinui-gallery.exe`。需安装 [x64 Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)。 |
| macOS Apple Silicon | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip) | 解压后打开 `RinUIGallery.app`。应用使用临时签名（ad-hoc），未经公证，Gatekeeper 可能阻止打开。 |
| Linux x64 | [tar.gz](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-linux-x64.tar.gz) | 解压后在解压目录运行 `./run-gallery.sh`。依赖系统 glibc、C++ 运行库、XCB/X11 和图形库。已在 Ubuntu 24.04 的 offscreen 和 Xvfb/XCB 环境验证启动。 |

将校验文件与压缩包放在同一目录。以 macOS 为例，运行 `grep -F 'RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip' SHA256SUMS.txt | shasum -a 256 -c`。校验失败时请重新下载压缩包。[SHA256SUMS.txt](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/SHA256SUMS.txt)

Windows 日常使用推荐 ZIP。发布页还提供经过打包和解包验证的[未签名 MSIX](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64-unsigned.msix)。如果没有适合当前平台的预编译包，请参阅[从源码构建 Gallery](#build-gallery-from-source)。

<GalleryCarousel section="guide" :show-copy="false" />

### 从源码构建 Gallery {#build-gallery-from-source}

按上文准备构建环境。克隆发布标签及子模块，然后在仓库根目录运行。

```bash
git clone --branch v0.1.0-alpha.2 --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

已有仓库请执行 `git submodule update --init --recursive`。`--dev` 在启动时读取文件系统 QML。修改后请重启 Gallery。自定义路径见[资源模式](project-structure#resource-modes)。

macOS 默认使用 QML 自绘窗口控件，追加 `--upstream-mac-frame` 可启用原生标题栏。其他 Gallery 参数可通过 `cargo run -p rinui-gallery -- --help` 查看。

## 继续阅读

- [配置与 Rust/QML 交互](rust-host)：配置、主题、翻译和 Rust/QML 交互。
- [Runtime 与 Session](runtime-sessions)：多引擎与已有 Qt 应用接入。
- [FluentWindow](/zh/Windowing/FluentWindow)：窗口与导航控件。
