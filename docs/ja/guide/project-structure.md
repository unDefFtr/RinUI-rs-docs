# プロジェクト構成とリソース

Rust の起動処理とビジネスロジックは `src/`、ページと画像は `qml/` に置きます。RinUI コントロールは QML の `import RinUI` で利用します。出発点となる完全なアプリは[はじめに](getting-started)を参照してください。

```text
rinui-starter/
├── Cargo.toml
├── build.rs              # 任意：アプリの QRC / CXX-Qt バックエンド
├── resources.qrc         # 任意：アプリのリソース
├── src/
│   ├── main.rs
│   └── backend.rs        # 任意：アプリの QObject
└── qml/
    ├── main.qml
    ├── pages/Home.qml
    └── assets/images/logo.png
```

## リソースモード {#resource-modes}

`HostConfig::default()` は `ResourceMode::Auto` を使用します。空でない `RINUI_DEV_PATH` があればそのパスを選び、それ以外は内蔵リソースを使います。無効な環境変数パスはエラーになり、暗黙にはフォールバックしません。

| モード | RinUI リソースの場所 |
| --- | --- |
| `Embedded` | コンパイル済み QRC。インポート接頭辞は `qrc:/rinui`、アセットは `:/rinui/RinUI`。 |
| `Filesystem(path)` | `RinUI/qmldir` を含む checkout。`RinUI/` ディレクトリ自体も指定可能。 |
| `Auto` | 空でない `RINUI_DEV_PATH`、それ以外は `Embedded`。 |

ファイルシステムモードでは、起動時に RinUI リソースを整えた一時コピーを作成し、元のファイルを保持します。ファイルの自動監視やホットリロードは行いません。UI の使用中はホストまたは Session を保持してください。関連するエンジンとリソースビューがすべて解放されると、一時コピーが削除されます。`0.1.0-alpha.2` では、Windows でアイコンフォントが一時ファイルを開いたままになり、最後の Session を閉じても削除できない問題を修正しました。

**ビルド時。** `rinui-host` はクレート内の `native/` からネイティブブリッジをビルドし、既定で `vendor/RinUI` の QML を内蔵します。内蔵する QML の取得元を変更するには `RINUI_UPSTREAM_DIR` を設定します。`RinUI/qmldir` を含むディレクトリか、`qmldir` を直接含む `RinUI/` ディレクトリを指定できます。指定先に `qmldir` が見つからない場合、ビルドは失敗します。

**実行時：** `RINUI_DEV_PATH` は `Auto` の選択に影響します。`Embedded` にコンパイル済みの内容は変更しません。配布には `Embedded` を明示することを推奨します。

**Gallery：** ルートから `cargo run -p rinui-gallery -- --dev /absolute/path/to/Rin-UI` でファイルシステムを選択します。`examples/gallery.qml` も必要です。RinUI だけのディレクトリなら `--examples-dir /absolute/path/to/examples` または `RINUI_EXAMPLES_DIR` を指定します。これらは Gallery 専用引数です。

## アプリのファイルと QRC

RinUI 内蔵リソースにアプリ自身の QML、画像、翻訳は**自動では含まれません**。開発中は絶対パスと `QmlSource::File` を使います。QML の相対 URL は宣言元ファイルを基準に解決されます。例えば `qml/main.qml` では `source: "assets/images/logo.png"` と書けます。

配布用のアプリリソースは別途同梱します。最小アプリに以下の設定とファイルを追加してください。この例は `main.qml` だけを内蔵します。参照するページ、画像、翻訳も相対パスを保つエイリアスで追加します。

既存の `[dependencies]` に次の依存関係を追加し、新しいビルド依存関係の節を作ります：

```toml [Cargo.toml]
cxx-qt = "0.10.0"

[build-dependencies]
cxx-qt-build = "0.10.0"
```

```xml [resources.qrc]
<RCC>
    <qresource prefix="/app">
        <file alias="main.qml">qml/main.qml</file>
    </qresource>
</RCC>
```

```rust [build.rs]
fn main() {
    println!("cargo::rerun-if-changed=resources.qrc");
    println!("cargo::rerun-if-changed=qml/main.qml");
    cxx_qt_build::CxxQtBuilder::new()
        .qrc("resources.qrc")
        .build();
}
```

`run()` でホスト作成前にアプリリソースを登録し、ファイル読み込みを QRC URL に置き換えます：

```rust
cxx_qt::init_crate!(rinui_starter);
// ... create host ...
host.load(&QmlSource::Url("qrc:/app/main.qml".to_owned()))?;
```

不要になった `PathBuf` の import を削除し、`cargo run` を実行します。ホストは自身の RinUI リソースを初期化し、上記 `init_crate!` はアプリのリソースを初期化します。`QmlSource::Url` には `qrc:/app/main.qml`、Qt のファイル名 API には `:/app/...` を使います。

予約済みの `qrc:/rinui` 名前空間は避けてください。独自 QML モジュールのディレクトリは読み込み前に `host.add_import_path(...)` または `configure_engine(...)` で登録します。

## パッケージ化

`cargo build --release -p rinui-gallery` で Gallery をビルドします。`rinui-rs` のルートから実行する以下のスクリプトを配布の参考にできます：

```bash [macOS / Linux]
bash scripts/package_macos.sh
bash scripts/package_linux.sh
```

```powershell [Windows]
.\scripts\package_windows.ps1
# MSIX: see packaging/windows/README.md and scripts/package_msix.ps1
```

これらは Gallery 用で、Qt ライブラリ、プラグイン、QML ランタイムモジュール、ライセンスを同梱します。独自アプリではバイナリとリソースに合わせて調整してください。QRC を内蔵しても Qt の動的ライブラリは必要です。開発 checkout や開発用 Qt パスに依存しない環境で確認してください。

## トラブルシューティングと HiDPI

- コンパイル時に `RinUI qmldir not found` が出る場合は、`RINUI_UPSTREAM_DIR` を解除して同梱リソースを使うか、有効なディレクトリを指定してください。Gallery のビルドで上流のサンプルが見つからない場合は `git submodule update --init --recursive` でサブモジュールを初期化します。
- `module "RinUI" is not installed`：`host.import_paths()`、リソースモード、`RINUI_DEV_PATH` を確認します。
- QML 読み込み失敗：`HostError::QmlLoadFailed` と `host.qml_warnings()` でファイル、import、リソースのエイリアスを調べます。
- Qt プラットフォームプラグインや Qt5Compat がない：Qt プラグインと QML モジュールを配布物に含めます。
- Qt Quick の寸法と `font.pixelSize` は論理ピクセルです。`Utils.dpiScale` を掛けないでください。`Utils.dp()` は非推奨の恒等関数です。

[テーマ](/ja/core/theme)、[ユーティリティ](/ja/core/utils)、[設定と Rust/QML 連携](rust-host)も参照してください。

[rinui-host のリソース実装](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/crates/rinui-host/src/resources.rs) · [Gallery CLI](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/examples/gallery/src/main.rs)
