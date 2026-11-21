# はじめに

## rinui-rs Gallery を実行する

rinui-rs Gallery を実行して、コントロールとインタラクティブなサンプルを確認できます。

<GalleryCarousel section="guide" :show-copy="false" />

## ビルド環境

Rust（MSRV **1.85**）、C++17 ツールチェーン、Qt Quick、Quick Controls 2、**Qt5Compat**、**QtShaderTools** を含む **Qt 6**（CI 検証バージョン：**6.12.0**）を用意します。

- **Windows:** MSVC Build Tools 2022、Rust MSVC ツールチェーン、同じアーキテクチャの Qt を使用します。開発者シェルから Cargo を実行し、Qt の要件に応じて `CXXFLAGS` に `/Zc:__cplusplus /permissive-` を追加します。
- **macOS:** Xcode Command Line Tools が必要です。Apple Silicon + Xcode 26 と Qt 6.11.1 未満の組み合わせでは、`CXXFLAGS` に `-include arm_acle.h` を追加します。
- **Linux:** GCC/Clang、Qt の X11/Wayland 開発依存関係、`lld`、`gold` または `mold` が必要です。Rust ツールチェーンにリンカーを設定してください。

Qt 6 の `qmake` が `PATH` にない場合は、実際のパスを `QMAKE` に指定します：

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

サブモジュールを含めてクローンし、リポジトリのルートから実行します：

```bash
git clone --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

既存の checkout では `git submodule update --init --recursive` を実行します。`--dev` は起動時にファイルシステムから QML を読み込みます。自動ホットリロードは行いません。パスの指定は[リソースモード](project-structure#resource-modes)を参照してください。

macOS は既定で QML のウィンドウボタンを使用します。ネイティブタイトルバーを使う場合は `--upstream-mac-frame` を追加します。他の引数は `cargo run -p rinui-gallery -- --help` で確認できます。

## 最小 Rust アプリを作る

このチュートリアルではソースへの path 依存関係を使います。ネイティブソースと初期化済みサブモジュールを含む完全な `rinui-rs` リポジトリの隣にアプリを置いてください。

```text
projects/
├── rinui-rs/                 # complete checkout, including upstream/Rin-UI
└── rinui-starter/
    ├── Cargo.toml
    ├── src/main.rs
    └── qml/main.qml
```

以下の 3 ファイルを作成します：

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

`rinui-starter/` から実行します：

```bash
cargo run
```

ボタンでライト／ダーク表示を切り替えられます。ホストは `QGuiApplication` を作成し、Qt Quick Controls を `Basic` スタイルに設定し、RinUI リソースと `ThemeManager` を登録して Qt イベントループを実行します。イベントループが終了するまで、ホストを GUI メインスレッドで保持してください。

`CARGO_MANIFEST_DIR` のパスはソース開発用です。配布時にはアプリ自身の QML とアセットも同梱します。[プロジェクト構成とリソース](project-structure)を参照してください。

## 次のステップ

- [設定と Rust/QML 連携](rust-host)：設定、テーマ、翻訳、Rust/QML 連携。
- [Runtime と Session](runtime-sessions)：複数エンジンと既存 Qt アプリへの接続。
- [FluentWindow](/ja/Windowing/FluentWindow)：ウィンドウとナビゲーション。
