# Runtime 与 Session

独立应用使用 `RinUIHost` 即可。多个使用方需要共用一个 Qt 应用时，使用一个 `RinUIRuntime` 配合多个 `RinUISession`；不要为每个窗口创建独立宿主。

## 在一个 Runtime 下创建两个 Session

保留[快速开始](getting-started)的 `Cargo.toml` 与 `qml/main.qml`，将 `src/main.rs` 替换如下，然后执行 `cargo run`。此例打开两个窗口，全部关闭后退出事件循环。

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

## 引擎共享与生命周期

| 预设 | QML 引擎 | 上下文属性冲突策略 |
| --- | --- | --- |
| `SessionConfig::shared()` | Runtime 的共享引擎 | `Reject` |
| `SessionConfig::private()` | 独立引擎 | `Overwrite` |
| `RinUIHost` 默认 Session | Runtime 的共享引擎 | `Overwrite` |

共享 Session 共用 QML 单例、上下文属性、导入路径和组件缓存。独立引擎隔离这些引擎局部对象，但仍共享进程级主题、配置和翻译协调。需要变更冲突策略时显式调用 `with_conflict_policy(ContextPropertyConflict::...)`。不要复用 `ThemeManager` 等宿主保留的上下文属性名。

释放 Session 会销毁它所属的根对象／窗口，并使其 `SessionHandle` 副本失效，其他 Session 继续运行。共享引擎的上下文属性属于引擎，并不是 Session 独立的命名空间。独立引擎随其 Session 销毁；Runtime 及外部 Qt 应用（如有）需保持存活直到使用方全部释放。

`RinUIRuntime`、Session、根对象／窗口句柄和 `HostHandle` 只能用于 Qt GUI 主线程，不能移动到工作线程。通过应用的 Qt 集成方式把后台计算结果调度回 GUI 线程。

## 接入已有 Qt 应用

一个进程只能有一个活动 GUI 应用／Runtime。使用 `attach_existing(&QGuiApplication, RuntimeConfig)` 借用已有应用，不要另建应用。它必须是 GUI 主线程上 Qt 当前的应用实例，纯 `QCoreApplication` 不足以加载 GUI。以下仅演示一个短暂的接入作用域，实际代码需在界面使用期间保留 Runtime 和 Session：

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

接入 Qt Widgets 的 `QApplication` 时，可使用合适的 Qt 向上转型，或进阶的 `unsafe attach_existing_raw` API。原始指针必须指向当前存活的 GUI 应用，在 GUI 线程使用，且其生命周期覆盖 Runtime、Session 和句柄。请参考仓库外部应用测试，不要猜测指针转换。

事件循环由原应用驱动。借用模式调用 `runtime.exec()` 会返回 `BorrowedApplicationExecNotAllowed`，释放 Runtime 不会销毁外部应用。`apply_identity_to_borrowed_app` 默认 `false`。Qt Quick Controls 样式应与已有应用一致，冲突时返回 `IncompatibleQuickControlsStyle`。

## 引擎配置与多窗口

在 QML 加载前通过 `session.configure_engine(...)` 配置导入／插件路径、base URL、离线存储与文件选择器。配置器仅在闭包内有效，不能逃逸。可通过 `SessionHandle::load_secondary_window` 加载附加窗口，保存返回的根 ID，以便使用 `release_root` 释放。

`sync_windows()` 可发现动态创建的 RinUI 窗口；需要显式归属时使用 `register_window_from_property(root_id, name)`。根对象／窗口 ID 属于创建它的 Session，跨 Session 使用会返回归属错误。需要严格处理错误时优先使用返回 `Result` 的 Session API；不要跨 QML 回调或事件循环入口持有主题 guard。

参见 [配置与 Rust/QML 交互](rust-host)和[项目结构与资源](project-structure)。

[Runtime/session implementation](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/src/runtime.rs) · [External QApplication test](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/tests/external_qapplication.rs)
