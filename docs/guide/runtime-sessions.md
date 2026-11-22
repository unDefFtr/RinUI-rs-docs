# Runtime and Sessions

Use `RinUIHost` for a standalone application. When several consumers need QML under one Qt application, use a single `RinUIRuntime` and multiple `RinUISession`s. Do not create one standalone host per window.

## Two sessions under one runtime

Keep the `Cargo.toml` and `qml/main.qml` from [Getting Started](getting-started), replace `src/main.rs` with the following, then run `cargo run`. It opens two windows. Close both to leave the event loop.

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

## Engine sharing and lifetime

| Preset | QML engine | Context property conflict policy |
| --- | --- | --- |
| `SessionConfig::shared()` | Runtime's shared engine | `Reject` |
| `SessionConfig::private()` | Dedicated engine | `Overwrite` |
| `RinUIHost` default session | Runtime's shared engine | `Overwrite` |

Shared sessions share QML singletons, context properties, import paths and component caches. Private engines isolate those engine-local objects; they still share process-level theme, configuration and translation coordination. Set a different collision policy explicitly with `with_conflict_policy(ContextPropertyConflict::...)` when needed. Do not reuse reserved host context property names such as `ThemeManager`.

Dropping a session releases that session's roots/windows and invalidates its `SessionHandle` clones; sibling sessions keep running. Shared-engine context properties belong to the engine, not an isolated per-session namespace. A private engine is destroyed with its session. Keep the runtime and the external Qt application, if any, alive until all consumers are released.

`RinUIRuntime`, sessions, root/window handles and `HostHandle` belong on the Qt GUI main thread and are not transferable to worker threads. Dispatch work results back to that thread using your application's Qt integration.

## Attach to an existing Qt application

A process may have only one active GUI application/runtime. Use `attach_existing(&QGuiApplication, RuntimeConfig)` to borrow the existing application rather than creating another. The application must already be Qt's active instance on the GUI thread. A plain `QCoreApplication` is insufficient. The following illustrates a short integration scope; production code must retain the runtime and session for the UI's lifetime:

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

For an existing Qt Widgets `QApplication`, use an appropriate Qt upcast or the advanced `unsafe attach_existing_raw` API. A raw pointer must identify the live active GUI application, be used on its GUI thread, and outlive the runtime, sessions and handles. Refer to the repository's external application tests rather than guessing pointer casts.

The upstream Qt Widgets integration test uses the `qt-widgets-test` feature, which is disabled by default.

The owner drives the event loop. Calling `runtime.exec()` on a borrowed application returns `BorrowedApplicationExecNotAllowed`; dropping the runtime does not destroy the external application. `apply_identity_to_borrowed_app` defaults to `false`. Match the existing Qt Quick Controls style; requesting a conflicting initialized style returns `IncompatibleQuickControlsStyle`.

When teardown is deferred, let the external event loop deliver Qt’s deferred deletion events after the current callback returns. Wait for teardown to complete before attaching another runtime.

## Engine configuration and multiple windows

Before loading QML, use `session.configure_engine(...)` for import/plugin paths, base URL, offline storage and file selectors. The configurator is scoped and must not escape its closure. Additional windows can be loaded through `SessionHandle::load_secondary_window`; keep returned root IDs to release them with `release_root`.

Use `sync_windows()` to discover dynamically created RinUI windows and `register_window_from_property(root_id, name)` when explicit attribution is needed. Root/window IDs belong to their owning session; using one from a sibling returns an ownership error. Prefer session APIs that return `Result` when handling errors is essential. Do not retain theme guards across QML callbacks or event-loop entry.

See [Configuration and Rust/QML Integration](rust-host) and [Project Structure and Resources](project-structure).

[RinUIRuntime](https://docs.rs/rinui-host/0.1.0-alpha.2/rinui_host/struct.RinUIRuntime.html) · [RinUISession](https://docs.rs/rinui-host/0.1.0-alpha.2/rinui_host/struct.RinUISession.html) · [External QApplication test](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/crates/rinui-host/tests/external_qapplication.rs)
