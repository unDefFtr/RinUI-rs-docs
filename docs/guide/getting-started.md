# Getting Started

Use `rinui-host` from crates.io to build your first RinUI-rs application. For the desktop demo, see [RinUI-rs Gallery](#gallery-downloads).

## Build requirements

Install Rust (MSRV **1.88**), a C++17 toolchain, and a **Qt 6** development installation with Qt Quick, Quick Controls 2, **Qt5Compat** and **QtShaderTools**. Release builds and CI use Qt **6.12.0**.

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


## Create a minimal Rust application

Add [rinui-host](https://crates.io/crates/rinui-host/0.1.0-alpha.2) as the only dependency. The crate includes the native bridges and pinned RinUI QML. You can create the app in any directory without cloning a repository or initializing a submodule.

```text
rinui-starter/
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

Run from `rinui-starter/`:

```bash
cargo run
```

The button switches light/dark appearance. The host creates `QGuiApplication`, configures Qt Quick Controls with the `Basic` style, registers RinUI resources and `ThemeManager`, then runs the Qt event loop. Keep the host on the GUI main thread until the event loop returns.

The `CARGO_MANIFEST_DIR` path is convenient for source development. A distributed application must package its own QML and assets: [Project Structure and Resources](project-structure) explains filesystem and QRC deployment.

## Run the RinUI-rs Gallery {#gallery-downloads}

Download the Gallery to explore the controls and interactive examples. The packages include Qt 6.12.0 and the required QML modules. [v0.1.0-alpha.2 release](https://github.com/unDefFtr/rinui-rs/releases/tag/v0.1.0-alpha.2)

| Platform | Download | How to run |
| --- | --- | --- |
| Windows x64 | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64.zip) | Fully extract the unsigned ZIP and run `rinui-gallery.exe`. Install the [x64 Visual C++ Redistributable](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist). |
| macOS Apple Silicon | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip) | Extract and open `RinUIGallery.app`. The app is ad-hoc signed and has no notarization, so Gatekeeper may block it. |
| Linux x64 | [tar.gz](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-linux-x64.tar.gz) | Extract the archive and run `./run-gallery.sh` from the extracted directory. Requires system glibc, the C++ runtime, XCB/X11 and graphics libraries. Startup was verified on Ubuntu 24.04 with offscreen and Xvfb/XCB. |

Save the checksum file alongside the archive. For the macOS ZIP, run `grep -F 'RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip' SHA256SUMS.txt | shasum -a 256 -c`. Download the archive again if verification fails. [SHA256SUMS.txt](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/SHA256SUMS.txt)

Use the Windows ZIP for everyday use. The release also includes an [unsigned MSIX](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64-unsigned.msix) whose packaging and unpacking were verified. If no package matches your platform, see [Build Gallery from source](#build-gallery-from-source).

<GalleryCarousel section="guide" :show-copy="false" />

### Build Gallery from source {#build-gallery-from-source}

Set up the build environment above. Clone the release tag with its submodule and run the Gallery from the repository root.

```bash
git clone --branch v0.1.0-alpha.2 --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

For an existing checkout, run `git submodule update --init --recursive`. `--dev` reads filesystem QML at startup. Restart the Gallery to pick up edits. See [resource modes](project-structure#resource-modes) for explicit paths.

macOS uses QML window controls by default. Add `--upstream-mac-frame` to opt into the native titlebar. Use `cargo run -p rinui-gallery -- --help` for other Gallery options.

## Next steps

- [Configuration and Rust/QML Integration](rust-host): configuration, theme, translation and Rust/QML interaction.
- [Runtime and Sessions](runtime-sessions): multiple engines and existing Qt applications.
- [FluentWindow](/Windowing/FluentWindow): window and navigation controls.
