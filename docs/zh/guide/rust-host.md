# Rust Host 接入

本文基于[快速开始](getting-started)的工程，介绍配置、主题、翻译和 Rust/QML 交互。

## 宿主配置

`RinUIHost` 是独立应用入口，`HostConfig` 在加载 QML 前配置以下内容：

| 字段 | 行为／默认值 |
| --- | --- |
| `identity` | `AppIdentity`；设置自己的组织与应用名以隔离配置，默认 `RinUI` / `Application`。 |
| `resource_mode` | 默认 `Auto`；需要稳定的内嵌 RinUI 资源时使用 `Embedded`。 |
| `config_location` | 默认 `ConfigLocation::Standard`，也可选 `CustomDir(PathBuf)` 或 `InMemory`。 |
| `locale` | `None` 检测系统语言，`Some("en_US".into())` 显式指定启动语言。 |
| `mac_window` | 默认 QML 自绘标题栏；`with_upstream_mac_frame()` 启用 macOS 原生窗口框架。 |
| `qt_quick_controls_style` | 默认 `"Basic"`，需在 Quick Controls 初始化前确定。 |

宿主通过 `ConfigStore` 保存 `rin_ui.json`。`Standard` 使用 Qt 应用本地数据目录；`CustomDir(dir)` 写入 `<dir>/RinUI/config/rin_ui.json`，不会再追加应用身份，因此应为每个应用选独立目录；`InMemory` 不读写配置磁盘文件。`config_store().borrow().path()` 可获取实际文件名（可能为 `None`）。配置支持旧 schema 迁移，以及损坏文件的备份与恢复。业务数据应另用独立存储。

## 数值、方法、主题与翻译

替换最小工程中的以下两个文件，保留 `Cargo.toml`，然后执行 `cargo run`。上下文属性在加载 QML 前注册；此例演示根属性读写、带类型的 QML 方法调用、主题设置和语言切换。

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
        &QVariant::from(&QString::from("Rust Host integration")),
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

没有应用 `.qm` 文件时，示例仍可运行：应用翻译目录是可选的，`qsTr()` 保留源文字。若要翻译，创建 context 为 `main`、源文字为 `Hello Rust` 的 Qt TS 翻译文件，使用 Qt Linguist 的 `lrelease` 编译，将 `zh_CN.qm` 放入 `languages/`。QRC 翻译使用 `TranslationCatalog::QrcPrefix`，且需自行内嵌每个 `.qm`。

`set_language()` 会重新翻译存活的 QML 绑定。`active_locale` / `effective_locale` 表示规范化后的应用语言，`rinui_locale` 与 `app_catalogs` 表示实际加载的翻译与回退情况。RinUI 翻译缺失时可以回退 `en_US`，同时保留应用请求的语言；RinUI 自带翻译不等于应用文字已翻译。

QML 中可调用 `Theme.setTheme(Theme.mode.Auto)`、`Theme.setThemeColor("#605ed2")`，并读取 `Theme.currentTheme.isDark`。Rust 中使用示例的 `with_theme_manager_mut` 限定访问范围；`toggle_theme("Auto")` 表示选中 Auto 模式。调用 QML 或进入事件循环前释放主题 guard，重叠或重入访问可能返回 `HostError::ReentrantCall`。

`set_root_property` 和 `invoke_root_method` 会对失效／非本 Session 的根对象、不存在的属性／方法以及不兼容类型返回错误。使用 `load` 返回的 `LoadedRoot.root_id`，不要自行构造数字 ID。

## 暴露 Rust QObject

需要可从 QML 调用的 Rust 行为时，可为上述接入工程添加 CXX-Qt 后端。将以下依赖加入对应的已有配置节，不要重复节标题。CXX-Qt 版本统一使用 **0.10.0**。原生工厂以 `UniquePtr` 返回生成的 QObject。

```toml [Cargo.toml]
cxx = "1.0"
cxx-qt = "0.10.0"

[build-dependencies]
cxx-qt-build = "0.10.0"
```

添加以下文件。若已经使用 QRC 构建脚本，请在同一个 builder 上组合 `.qrc("resources.qrc")`，保留资源注册。

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

在 `src/main.rs` 顶部加入 `mod backend;`，在 `run()` 创建宿主前调用 `cxx_qt::init_crate!(rinui_starter);`，并在 **`host.load(...)` 之前**加入：

```rust
host.expose_qobject("Backend", backend::ffi::make_backend())?;
```

在 QML 的 `Column` 中加入 `Text { text: Backend.greeting() }`，再执行 `cargo run`。包名需保持一致，因为生成的 C++ include 路径和 `init_crate!` 标识符依赖它。

`expose_qobject` 将所有权转交给引擎管理的存储，不要单独销毁对象；引擎会先于暴露的对象销毁。`HostHandle` 可供后端持有，但只能用于 GUI 线程；所属 Session 结束后即失效，不会延长宿主生命周期。`Backend` 等上下文属性由应用定义，`rinui-host` 不会自动提供。

## Windows 背景与窗口配置

先加载窗口，再应用背景效果；请求 Windows 专属效果前应检查平台能力。Rust 的 `ThemeManager::apply_backdrop_effect(QString)` 返回：

| 状态码 | 含义 |
| --- | --- |
| `0` | 应用成功。 |
| `-1` | 效果不支持或参数无效。 |
| `-2` | 非 Windows、没有注册窗口，或调用不可用／重入。 |

Windows 10 支持 Acrylic；Windows 11 支持 Mica；Tabbed 要求 Windows 11 22H2。Windows 11 21H2 请求 Acrylic／Tabbed 时回退 Mica。当前宿主未对 Windows 11 22H2+ 开放 Acrylic 支持。可通过 `host.runtime().capabilities()` 检查。QML 的 `Theme.setBackdropEffect()` 包装函数不会转发 Rust 状态码。

图标可在加载后通过 `host.set_icon(path, Some(loaded.root_id))` 设置，发布时使用应用 QRC 路径。参见 [FluentWindow](/zh/Windowing/FluentWindow) 和 [Runtime 与 Session](runtime-sessions)。

[rinui-host public API](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/src/lib.rs) · [Gallery backend](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/examples/gallery/src/backend.rs)
