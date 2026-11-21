# Configuration and Rust/QML Integration

This guide builds on the application from [Getting Started](getting-started) to demonstrate configuration, themes, translation and Rust/QML interaction.

## Host configuration

`RinUIHost` is the standalone entry point. `HostConfig` configures these concerns before QML loads:

| Field | Behavior / default |
| --- | --- |
| `identity` | `AppIdentity`; use your own organization and application name to isolate configuration. Default: `RinUI` / `Application`. |
| `resource_mode` | Default `Auto`; use `Embedded` for reproducible bundled RinUI resources. |
| `config_location` | Default `ConfigLocation::Standard`; alternatives: `CustomDir(PathBuf)` and `InMemory`. |
| `locale` | `None` detects the system locale; `Some("en_US".into())` starts explicitly. |
| `mac_window` | Default QML titlebar; `with_upstream_mac_frame()` opts into native macOS framing. |
| `qt_quick_controls_style` | Default `"Basic"`; set before Quick Controls initialization. |

The host persists `rin_ui.json` through `ConfigStore`. `Standard` uses Qt's application local data location; `CustomDir(dir)` writes to `<dir>/RinUI/config/rin_ui.json`; it does not append the application identity, so choose a separate directory per application. `InMemory` performs no configuration disk I/O. `config_store().borrow().path()` returns the actual optional filename. Configuration supports migration of legacy schema and recovery of corrupt files with a backup. Business data should use a separate store.

## Values, methods, theme and translation

Replace the minimal application's two files below, keeping its `Cargo.toml`, then run `cargo run`. Register context properties before loading QML. The example reads/writes a root property, invokes a typed QML method, sets the theme and switches language.

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

Without application `.qm` files, the example still runs: the application catalog is optional and `qsTr()` keeps its source text. To translate it, create a Qt TS catalog with context `main` and source `Hello Rust`, compile it with Qt Linguist's `lrelease`, and place `zh_CN.qm` in `languages/`. For QRC catalogs, use `TranslationCatalog::QrcPrefix` and bundle each `.qm` yourself.

`set_language()` retranslates live QML bindings. `active_locale` / `effective_locale` represent the canonical application locale; `rinui_locale` and `app_catalogs` report the actual loaded catalogs and fallbacks. A missing RinUI catalog can fall back to `en_US` while the application retains its requested locale. Available RinUI translations do not imply that application strings are translated.

In QML, use `Theme.setTheme(Theme.mode.Auto)`, `Theme.setThemeColor("#605ed2")` and `Theme.currentTheme.isDark`. In Rust, use the scoped `with_theme_manager_mut` access above; `toggle_theme("Auto")` selects Auto mode rather than cycling modes. Release theme guards before invoking QML or entering the event loop; overlapping/reentrant access can return `HostError::ReentrantCall`.

`set_root_property` and `invoke_root_method` return errors for stale/wrong-session roots, missing properties/methods or incompatible types. Keep the `LoadedRoot.root_id` returned by `load`; do not invent numeric root IDs.

## Expose a Rust QObject

For invokable Rust behavior, add a CXX-Qt backend to the integration application. Add the dependencies below to their existing sections (do not duplicate section headers). Use matching CXX-Qt **0.10.0** versions. The native factory returns the generated QObject in a `UniquePtr`.

```toml [Cargo.toml]
cxx = "1.0"
cxx-qt = "0.10.0"

[build-dependencies]
cxx-qt-build = "0.10.0"
```

Add these files; if you already have a QRC build script, combine `.qrc("resources.qrc")` with the same builder instead of replacing resource registration.

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

Add `mod backend;` at the top of `src/main.rs`. In `run()`, call `cxx_qt::init_crate!(rinui_starter);` before creating the host, then add the following **before `host.load(...)`**:

```rust
host.expose_qobject("Backend", backend::ffi::make_backend())?;
```

In the QML `Column`, add `Text { text: Backend.greeting() }`, then run `cargo run`. Keep the package name: the generated C++ include path and `init_crate!` identifier depend on it.

`expose_qobject` transfers ownership to engine-managed storage; do not destroy the object separately. The engine is destroyed before its exposed objects. `HostHandle` is a cloneable GUI-thread handle for backends; it becomes disposed when its owning session ends and does not extend the host lifetime. Context properties such as `Backend` are application-defined, not automatically provided by `rinui-host`.

## Windows backdrops and window configuration

Load a window before applying an effect. Check platform capabilities before requesting Windows-only behavior. The Rust `ThemeManager::apply_backdrop_effect(QString)` returns:

| Code | Meaning |
| --- | --- |
| `0` | Applied successfully. |
| `-1` | Unsupported or invalid effect. |
| `-2` | Non-Windows platform, no registered window, or unavailable/reentrant call. |

Windows 10 supports Acrylic; Windows 11 supports Mica; Tabbed requires Windows 11 22H2. On Windows 11 21H2, Acrylic/Tabbed requests fall back to Mica. The current host does not expose Acrylic support on Windows 11 22H2+. Use `host.runtime().capabilities()` to inspect support. The QML `Theme.setBackdropEffect()` wrapper does not forward the Rust status code.

For icons, use `host.set_icon(path, Some(loaded.root_id))` after loading; use an application QRC path when distributing. See [FluentWindow](/Windowing/FluentWindow) and [Runtime and Sessions](runtime-sessions).

[rinui-host public API](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/src/lib.rs) · [Gallery backend](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/examples/gallery/src/backend.rs)
