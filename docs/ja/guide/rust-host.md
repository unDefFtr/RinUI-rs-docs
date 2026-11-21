# 設定と Rust/QML 連携

[はじめに](getting-started)のアプリを基に、設定、テーマ、翻訳、Rust/QML 連携を紹介します。

## ホストの設定

`RinUIHost` は独立アプリの入口です。`HostConfig` は QML 読み込み前に以下を設定します。

| フィールド | 動作／既定値 |
| --- | --- |
| `identity` | `AppIdentity`。設定を分離するため独自の組織名とアプリ名を指定。既定値は `RinUI` / `Application`。 |
| `resource_mode` | 既定値 `Auto`。再現可能な内蔵リソースには `Embedded` を使用。 |
| `config_location` | 既定値 `ConfigLocation::Standard`。`CustomDir(PathBuf)` と `InMemory` も利用可能。 |
| `locale` | `None` はシステム言語を検出。`Some("en_US".into())` で明示指定。 |
| `mac_window` | 既定は QML タイトルバー。`with_upstream_mac_frame()` で macOS ネイティブフレームを選択。 |
| `qt_quick_controls_style` | 既定値 `"Basic"`。Quick Controls の初期化前に指定。 |

ホストは `ConfigStore` で `rin_ui.json` を保存します。`Standard` は Qt のアプリローカルデータ領域を使い、`CustomDir(dir)` は `<dir>/RinUI/config/rin_ui.json` に書き込み、アプリ名は追加しません。アプリごとに別のディレクトリを指定してください。`InMemory` は設定ファイルを読み書きしません。`config_store().borrow().path()` で実際のファイル名（省略可能）を確認できます。旧 schema の移行と、破損ファイルのバックアップ／復旧をサポートします。業務データには別のストアを使ってください。

## 値、メソッド、テーマ、翻訳

最小アプリの以下 2 ファイルを置き換え、`Cargo.toml` は維持して `cargo run` を実行します。コンテキストプロパティを QML 読み込み前に登録します。ルートプロパティの読み書き、型付き QML メソッドの呼び出し、テーマ設定と言語切り替えを示します。

::: code-group

```rust [src/main.rs]
use rinui_host::cxx_qt_lib::{QString, QVariant};
use rinui_host::{
    AppIdentity, HostConfig, QmlSource, ResourceMode, RinUIHost, TranslationCatalog,
};
use std::{path::PathBuf, process::ExitCode};

fn run() -> Result<i32, Box<dyn std::error::Error>> {
    let app_dir = PathBuf::from(env!("CARGO_MANIFEST_DIR"));
    let mut host = RinUIHost::new(HostConfig {
        identity: AppIdentity::new("RinUIDocs", "Integration"),
        resource_mode: ResourceMode::Embedded,
        locale: Some("en_US".to_owned()),
        ..HostConfig::default()
    })?;
    host.configure_engine(|engine| {
        engine.add_import_path(&app_dir.join("qml"));
    })?;
    host.set_context_property_string("CustomGreeting", "Hello from Rust")?;
    host.register_translation_catalog(TranslationCatalog::Directory {
        id: "app".to_owned(),
        dir: app_dir.join("languages"),
    })?;
    let loaded = host.load(&QmlSource::File(app_dir.join("qml/main.qml")))?;
    host.set_root_property(
        loaded.root_id,
        "title",
        &QVariant::from(&QString::from("RinUI-rs integration")),
    )?;
    let title = host.get_root_property(loaded.root_id, "title")?;
    let sum = host.invoke_root_method(
        loaded.root_id,
        "addNumbers",
        &[QVariant::from(&20i32), QVariant::from(&22i32)],
    )?;
    println!("title: {:?}, sum: {:?}", title.value::<QString>(), sum.value::<i32>());
    host.with_theme_manager_mut(|mut theme| {
        theme.as_mut().toggle_theme(QString::from("Dark"));
        theme.as_mut().set_theme_color(QString::from("#605ed2"));
    })?;
    let translation = host.set_language("zh_CN")?;
    println!("active: {}, RinUI catalog: {}",
        translation.active_locale, translation.rinui_locale);
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
    title: "Integration"
    function addNumbers(a: int, b: int): int { return a + b }

    Column {
        anchors.centerIn: parent
        spacing: 16
        Text { text: CustomGreeting }
        Text { text: qsTr("Hello Rust") }
        Button {
            text: qsTr("Toggle theme")
            onClicked: Theme.toggleMode()
        }
    }
}
```

:::

アプリの `.qm` がなくても実行できます。アプリカタログは任意で、`qsTr()` は原文を表示します。翻訳するには context `main`、原文 `Hello Rust` の Qt TS カタログを作成し、Qt Linguist の `lrelease` でコンパイルして `languages/zh_CN.qm` に置きます。QRC カタログには `TranslationCatalog::QrcPrefix` を使い、各 `.qm` を自分で同梱します。

`set_language()` は存続中の QML バインディングを再翻訳します。`active_locale` / `effective_locale` は正規化されたアプリ言語、`rinui_locale` と `app_catalogs` は実際のカタログとフォールバックを表します。RinUI が `en_US` にフォールバックしてもアプリ言語は維持できます。RinUI の翻訳だけでアプリの文字列まで翻訳されるわけではありません。

QML では `Theme.setTheme(Theme.mode.Auto)`、`Theme.setThemeColor("#605ed2")`、`Theme.currentTheme.isDark` を使います。Rust では例の `with_theme_manager_mut` でアクセスを限定します。`toggle_theme("Auto")` は Auto モードを選択します。QML 呼び出しやイベントループの前にテーマ guard を解放してください。重複／再入アクセスは `HostError::ReentrantCall` になる場合があります。

`set_root_property` と `invoke_root_method` は無効なルート、別 Session のルート、不明なプロパティ／メソッド、型不一致にエラーを返します。`load` が返す `LoadedRoot.root_id` を使用し、数値 ID を独自に作らないでください。

## Rust QObject を公開する

QML から Rust の処理を呼ぶには、上記アプリに CXX-Qt バックエンドを追加します。以下の依存関係は対応する既存の節へ追加し、節ヘッダーを重複させないでください。CXX-Qt **0.10.0** に統一します。ネイティブファクトリは生成済み QObject を `UniquePtr` で返します。

```toml [Cargo.toml]
cxx = "1.0"
cxx-qt = "0.10.0"

[build-dependencies]
cxx-qt-build = "0.10.0"
```

以下のファイルを追加します。QRC の build.rs がある場合、同じ builder に `.qrc("resources.qrc")` を組み合わせて登録を維持してください。

::: code-group

```rust [build.rs]
fn main() {
    println!("cargo::rerun-if-changed=native/starter_factory.h");
    println!("cargo::rerun-if-changed=native/starter_factory.cpp");
    cxx_qt_build::CxxQtBuilder::new()
        .include_dir("native")
        .file("src/backend.rs")
        .cpp_file("native/starter_factory.cpp")
        .build();
}
```

```rust [src/backend.rs]
#[cxx_qt::bridge(namespace = "starter")]
pub mod ffi {
    #[namespace = ""]
    unsafe extern "C++" {
        include!("cxx-qt-lib/qstring.h");
        type QString = rinui_host::cxx_qt_lib::QString;
        include!("starter_factory.h");
        fn make_backend() -> UniquePtr<Backend>;
    }
    extern "RustQt" {
        #[qobject]
        type Backend = super::BackendRust;
        #[qinvokable]
        fn greeting(self: &Backend) -> QString;
    }
}

#[derive(Default)]
pub struct BackendRust;

impl ffi::Backend {
    pub fn greeting(&self) -> rinui_host::cxx_qt_lib::QString {
        rinui_host::cxx_qt_lib::QString::from("Hello from a Rust QObject")
    }
}
```

```cpp [native/starter_factory.h]
#pragma once
#include <memory>
namespace starter { class Backend; }
std::unique_ptr<starter::Backend> make_backend();
```

```cpp [native/starter_factory.cpp]
#include "starter_factory.h"
#include "rinui-starter/src/backend.cxxqt.h"

std::unique_ptr<starter::Backend> make_backend()
{
    return std::make_unique<starter::Backend>();
}
```

:::

`src/main.rs` の先頭に `mod backend;`、`run()` のホスト作成前に `cxx_qt::init_crate!(rinui_starter);` を追加し、**`host.load(...)` の前**に以下を追加します：

```rust
host.expose_qobject("Backend", backend::ffi::make_backend())?;
```

QML の `Column` に `Text { text: Backend.greeting() }` を追加して `cargo run` を実行します。生成された C++ include パスと `init_crate!` 識別子が依存するため、パッケージ名を維持してください。

`expose_qobject` はエンジン管理ストレージへ所有権を移します。別途破棄しないでください。エンジンは公開オブジェクトより先に破棄されます。`HostHandle` はバックエンド向けの GUI スレッド専用ハンドルです。所有 Session の終了時に無効になり、ホストの寿命を延長しません。`Backend` などはアプリが定義するプロパティであり、自動では提供されません。

## Windows の背景とウィンドウ設定

ウィンドウを読み込んでから背景を適用します。Windows 専用効果はプラットフォーム能力を確認してから要求してください。Rust の `ThemeManager::apply_backdrop_effect(QString)` は以下を返します。

| コード | 意味 |
| --- | --- |
| `0` | 適用成功。 |
| `-1` | 未対応または無効な効果。 |
| `-2` | Windows 以外、登録ウィンドウなし、または利用不可／再入。 |

Windows 10 は Acrylic、Windows 11 は Mica、Windows 11 22H2 以降は Tabbed に対応します。Windows 11 21H2 の Acrylic／Tabbed 要求は Mica にフォールバックします。現在のホストは Windows 11 22H2 以降の Acrylic をサポートとして公開しません。`host.runtime().capabilities()` で確認できます。QML の `Theme.setBackdropEffect()` は Rust の状態コードを返しません。

アイコンは読み込み後に `host.set_icon(path, Some(loaded.root_id))` で設定します。配布にはアプリの QRC パスを使ってください。[FluentWindow](/ja/Windowing/FluentWindow) と [Runtime と Session](runtime-sessions)も参照してください。

[rinui-host public API](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/src/lib.rs) · [Gallery backend](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/examples/gallery/src/backend.rs)
