# Runtime と Session

独立アプリには `RinUIHost` を使用します。複数の利用側が一つの Qt アプリを共有する場合、一つの `RinUIRuntime` と複数の `RinUISession` を使います。ウィンドウごとに独立ホストを作成しないでください。

## 一つの Runtime に二つの Session

[はじめに](getting-started)の `Cargo.toml` と `qml/main.qml` を維持し、`src/main.rs` を以下に置き換えて `cargo run` を実行します。二つのウィンドウが開き、両方を閉じるとイベントループが終了します。

```rust [src/main.rs]
use rinui_host::{
    AppIdentity, QmlSource, ResourceMode, RinUIRuntime, RuntimeConfig, SessionConfig,
};
use std::{path::PathBuf, process::ExitCode};

fn run() -> Result<i32, Box<dyn std::error::Error>> {
    let mut runtime = RinUIRuntime::new(RuntimeConfig {
        identity: AppIdentity::new("RinUIDocs", "Sessions"),
        resource_mode: ResourceMode::Embedded,
        ..RuntimeConfig::default()
    })?;
    let mut first = runtime.create_session(SessionConfig::shared())?;
    let mut second = runtime.create_session(SessionConfig::private())?;
    let qml = PathBuf::from(env!("CARGO_MANIFEST_DIR")).join("qml/main.qml");
    first.load(&QmlSource::File(qml.clone()))?;
    second.load(&QmlSource::File(qml))?;
    Ok(runtime.exec()?)
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

## エンジン共有と寿命

| プリセット | QML エンジン | コンテキストプロパティの衝突方針 |
| --- | --- | --- |
| `SessionConfig::shared()` | Runtime の共有エンジン | `Reject` |
| `SessionConfig::private()` | 専用エンジン | `Overwrite` |
| `RinUIHost` の既定 Session | Runtime の共有エンジン | `Overwrite` |

共有 Session は QML シングルトン、コンテキストプロパティ、import パス、キャッシュを共有します。専用エンジンはそれらを分離しますが、プロセス全体のテーマ、設定、翻訳の調整は共有します。方針を変える場合は `with_conflict_policy(ContextPropertyConflict::...)` を明示します。`ThemeManager` などホスト予約名を再利用しないでください。

Session の破棄はそのルート／ウィンドウを解放し、`SessionHandle` のコピーを無効化します。他の Session は継続します。共有エンジンのコンテキストプロパティはエンジンに属し、Session ごとの名前空間ではありません。専用エンジンはその Session とともに破棄されます。全利用側の解放まで Runtime と外部 Qt アプリ（使用時）を保持してください。

`RinUIRuntime`、Session、ルート／ウィンドウハンドル、`HostHandle` は Qt GUI メインスレッド専用で、ワーカースレッドへ移動できません。バックグラウンド処理の結果はアプリの Qt 統合経由で GUI スレッドへ戻してください。

## 既存 Qt アプリに接続する

一つのプロセスには一つのアクティブな GUI アプリ／Runtime を使用します。`attach_existing(&QGuiApplication, RuntimeConfig)` で既存アプリを借用します。GUI スレッド上で Qt の現在のインスタンスである必要があり、単なる `QCoreApplication` では不十分です。以下は短いスコープの例です。実際には UI が必要な期間、Runtime と Session を保持してください。

```rust
use rinui_host::cxx_qt_lib::QGuiApplication;
use rinui_host::{RinUIRuntime, RuntimeConfig, SessionConfig};

fn integrate(app: &QGuiApplication) -> Result<(), Box<dyn std::error::Error>> {
    let runtime = RinUIRuntime::attach_existing(app, RuntimeConfig::default())?;
    let mut session = runtime.create_session(SessionConfig::private())?;
    // Configure and load QML here. Keep runtime/session alive while the UI is needed.
    session.set_context_property_string("CustomGreeting", "Existing Qt application")?;
    // The owner drives its event loop; do not call runtime.exec() here.
    Ok(()) // This illustrative scope releases the session immediately.
}
```

Qt Widgets の `QApplication` には適切な Qt upcast、または上級用 `unsafe attach_existing_raw` API を使います。raw ポインターは現在存続中の GUI アプリを指し、GUI スレッドで使用し、Runtime、Session、ハンドルより長く存続する必要があります。推測したポインターキャストではなく、外部アプリのテストを参照してください。

上流の Qt Widgets 統合テストでは、既定で無効になっている `qt-widgets-test` feature を使います。

イベントループは元の所有側が実行します。借用アプリで `runtime.exec()` を呼ぶと `BorrowedApplicationExecNotAllowed` になります。Runtime を破棄しても外部アプリは破棄されません。`apply_identity_to_borrowed_app` の既定値は `false` です。既存の Qt Quick Controls スタイルに合わせてください。初期化済みスタイルと衝突すると `IncompatibleQuickControlsStyle` になります。

破棄が遅延された場合、現在のコールバックから戻った後に、外部イベントループで Qt の遅延削除イベントを処理します。破棄が完了してから次の Runtime を接続してください。

## エンジン設定と複数ウィンドウ

QML 読み込み前に `session.configure_engine(...)` で import／plugin パス、base URL、オフラインストレージ、ファイルセレクターを設定します。設定器はクロージャー外へ持ち出せません。追加ウィンドウは `SessionHandle::load_secondary_window` で読み込み、返されたルート ID を保存して `release_root` で解放します。

`sync_windows()` で動的な RinUI ウィンドウを検出できます。明示的な帰属には `register_window_from_property(root_id, name)` を使います。ルート／ウィンドウ ID は所有 Session に属し、別 Session での使用はエラーです。エラー処理が必要なら `Result` を返す Session API を優先し、QML コールバックやイベントループ開始をまたいでテーマ guard を保持しないでください。

[設定と Rust/QML 連携](rust-host)、[プロジェクト構成とリソース](project-structure)も参照してください。

[RinUIRuntime](https://docs.rs/rinui-host/0.1.0-alpha.2/rinui_host/struct.RinUIRuntime.html) · [RinUISession](https://docs.rs/rinui-host/0.1.0-alpha.2/rinui_host/struct.RinUISession.html) · [外部 QApplication のテスト](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/crates/rinui-host/tests/external_qapplication.rs)
