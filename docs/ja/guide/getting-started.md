# はじめに

crates.io の `rinui-host` を使って最初の RinUI-rs アプリを作ります。デスクトップデモを試すには [RinUI-rs Gallery](#gallery-downloads) を参照してください。

## ビルド環境

Rust（MSRV **1.88**）、C++17 ツールチェーン、Qt Quick、Quick Controls 2、**Qt5Compat**、**QtShaderTools** を含む **Qt 6** の開発環境を用意します。リリースビルドと CI では Qt **6.12.0** を使用します。

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


## 最小 Rust アプリを作る

依存関係には [rinui-host](https://crates.io/crates/rinui-host/0.1.0-alpha.2) だけを追加します。このクレートにはネイティブブリッジとバージョンを固定した RinUI QML が含まれます。リポジトリのクローンやサブモジュールの初期化なしに、任意のディレクトリでアプリを作れます。

```text
rinui-starter/
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

`rinui-starter/` から実行します：

```bash
cargo run
```

ボタンでライト／ダーク表示を切り替えられます。ホストは `QGuiApplication` を作成し、Qt Quick Controls を `Basic` スタイルに設定し、RinUI リソースと `ThemeManager` を登録して Qt イベントループを実行します。イベントループが終了するまで、ホストを GUI メインスレッドで保持してください。

`CARGO_MANIFEST_DIR` のパスはソース開発用です。配布時にはアプリ自身の QML とアセットも同梱します。[プロジェクト構成とリソース](project-structure)を参照してください。

## RinUI-rs Gallery を実行する {#gallery-downloads}

Gallery をダウンロードして、コントロールとインタラクティブなサンプルを試せます。パッケージには Qt 6.12.0 と必要な QML モジュールが含まれます。[v0.1.0-alpha.2 のリリースページ](https://github.com/unDefFtr/rinui-rs/releases/tag/v0.1.0-alpha.2)

| プラットフォーム | ダウンロード | 実行方法 |
| --- | --- | --- |
| Windows x64 | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64.zip) | 署名なしの ZIP をすべて展開して `rinui-gallery.exe` を起動します。[x64 Visual C++ 再頒布可能パッケージ](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)が必要です。 |
| macOS Apple Silicon | [ZIP](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip) | 展開して `RinUIGallery.app` を開きます。アドホック署名のみで公証されていないため、Gatekeeper が起動をブロックする場合があります。 |
| Linux x64 | [tar.gz](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-linux-x64.tar.gz) | アーカイブを展開し、展開先で `./run-gallery.sh` を実行します。システムの glibc、C++ ランタイム、XCB/X11、グラフィックスライブラリが必要です。Ubuntu 24.04 の offscreen と Xvfb/XCB で起動を確認済みです。 |

チェックサムファイルをアーカイブと同じディレクトリに保存します。macOS の ZIP を検証するには、`grep -F 'RinUIGallery-v0.1.0-alpha.2-macos-arm64.zip' SHA256SUMS.txt | shasum -a 256 -c` を実行します。検証に失敗したらアーカイブを再ダウンロードしてください。[SHA256SUMS.txt](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/SHA256SUMS.txt)

Windows で通常利用する場合は ZIP を使います。リリースページにはパッケージの作成と展開を検証した[署名なしの MSIX](https://github.com/unDefFtr/RinUI-rs/releases/download/v0.1.0-alpha.2/RinUIGallery-v0.1.0-alpha.2-windows-x64-unsigned.msix)もあります。環境に合ったパッケージがない場合は、[Gallery をソースからビルド](#build-gallery-from-source)を参照してください。

<GalleryCarousel section="guide" :show-copy="false" />

### Gallery をソースからビルド {#build-gallery-from-source}

上記のビルド環境を用意します。リリースタグをサブモジュールごとクローンし、リポジトリのルートから Gallery を実行します。

```bash
git clone --branch v0.1.0-alpha.2 --recurse-submodules https://github.com/unDefFtr/rinui-rs
cd rinui-rs
cargo run -p rinui-gallery

cargo run -p rinui-gallery -- --dev
cargo run -p rinui-gallery -- --locale zh_CN
```

既存のチェックアウトでは `git submodule update --init --recursive` を実行します。`--dev` は起動時にファイルシステムから QML を読み込みます。変更を反映するには Gallery を再起動してください。パスの指定は[リソースモード](project-structure#resource-modes)を参照してください。

macOS は既定で QML のウィンドウボタンを使用します。ネイティブタイトルバーを使う場合は `--upstream-mac-frame` を追加します。他の引数は `cargo run -p rinui-gallery -- --help` で確認できます。

## 次のステップ

- [設定と Rust/QML 連携](rust-host)：設定、テーマ、翻訳、Rust/QML 連携。
- [Runtime と Session](runtime-sessions)：複数エンジンと既存 Qt アプリへの接続。
- [FluentWindow](/ja/Windowing/FluentWindow)：ウィンドウとナビゲーション。
