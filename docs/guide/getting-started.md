# Getting Started

## Run the Rust Gallery

Run the Gallery to explore RinUI controls and interactive examples.

<GalleryCarousel section="guide" :show-copy="false" />

## Build requirements

Install Rust (MSRV **1.85**), a C++17 toolchain, and **Qt 6** (CI-tested with **6.12.0**) with Qt Quick, Quick Controls 2, **Qt5Compat** and **QtShaderTools**.

- **Windows:** MSVC Build Tools 2022 and the Rust MSVC toolchain, with matching Qt architecture. Run Cargo in a developer shell. Include `/Zc:__cplusplus /permissive-` in `CXXFLAGS` when required by Qt.
- **macOS:** Xcode Command Line Tools. On Apple Silicon with Xcode 26 and Qt older than 6.11.1, append `-include arm_acle.h` to `CXXFLAGS`.
- **Linux:** GCC/Clang, Qt's X11/Wayland development dependencies, and `lld`, `gold` or `mold` for linking. Configure the selected linker for your Rust toolchain.

Point `QMAKE` to your Qt 6 executable if it is not on `PATH` (replace the example path):

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

Clone with the submodule, then run from the repository root:

```bash
git clone --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

For an existing checkout, run `git submodule update --init --recursive`. `--dev` reads filesystem QML at startup; it does not enable automatic hot reload. See [resource modes](project-structure#resource-modes) for explicit paths.

macOS uses QML window controls by default. Add `--upstream-mac-frame` to opt into the native titlebar. Use `cargo run -p rinui-gallery -- --help` for other Gallery options.

## Create a minimal Rust application

This tutorial uses a source path dependency. Keep the application next to the complete `rinui-rs` repository, including its native sources and initialized submodule.

```text
projects/
├── rinui-rs/                 # complete checkout, including upstream/Rin-UI
└── rinui-starter/
    ├── Cargo.toml
    ├── src/main.rs
    └── qml/main.qml
```

Create these three files:

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
    title: qsTr("Rust + RinUI")

    Button {
        anchors.centerIn: parent
        text: qsTr("Toggle theme")
        onClicked: Theme.toggleMode()
    }
}
```

:::

Run from `rinui-starter/`:

```bash
cargo run
```

The button switches light/dark appearance. The host creates `QGuiApplication`, configures Qt Quick Controls with the `Basic` style, registers RinUI resources and `ThemeManager`, then runs the Qt event loop. Keep the host on the GUI main thread until the event loop returns.

The `CARGO_MANIFEST_DIR` path is convenient for source development. A distributed application must package its own QML and assets: [Project Structure and Resources](project-structure) explains filesystem and QRC deployment.

## Next steps

- [Rust Host integration](rust-host): configuration, theme, translation and Rust/QML interaction.
- [Runtime and Sessions](runtime-sessions): multiple engines and existing Qt applications.
- [FluentWindow](/Windowing/FluentWindow): window and navigation controls.
