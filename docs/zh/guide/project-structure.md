# 项目结构与资源

Rust 启动与业务逻辑放在 `src/`，页面和图片放在 `qml/`。RinUI 控件仍通过 QML 的 `import RinUI` 导入。完整起点见[快速开始](getting-started)。

```text
projects/
├── rinui-rs/
└── rinui-starter/
    ├── Cargo.toml
    ├── build.rs              # optional: application QRC / CXX-Qt backend
    ├── resources.qrc         # optional: application resources
    ├── src/
    │   ├── main.rs
    │   └── backend.rs        # optional: application QObject
    └── qml/
        ├── main.qml
        ├── pages/Home.qml
        └── assets/images/logo.png
```

## 资源模式 {#resource-modes}

`HostConfig::default()` 使用 `ResourceMode::Auto`：非空 `RINUI_DEV_PATH` 优先，否则使用内嵌 RinUI 资源。环境变量指向无效路径会返回错误，不会静默回退。

| 模式 | RinUI 资源来源 |
| --- | --- |
| `Embedded` | 编译进程序的 QRC，导入前缀为 `qrc:/rinui`，资源位于 `:/rinui/RinUI`。 |
| `Filesystem(path)` | 包含 `RinUI/qmldir` 的 checkout，也接受直接传入 `RinUI/` 目录。 |
| `Auto` | 优先使用非空 `RINUI_DEV_PATH`，否则选择 `Embedded`。 |

文件系统模式在启动时读取资源，不自动监听或热重载。界面使用期间需保持宿主／Session 存活。

**构建期：** `RINUI_UPSTREAM_DIR` 可覆盖用于内嵌的上游源码位置，接受上游仓库目录或其 `RinUI/` 子目录；原生源码仍需完整 `rinui-rs` checkout。

**运行期：** `RINUI_DEV_PATH` 影响 `Auto` 的选择，不能修改已编译进 `Embedded` 应用的资源。发布时建议显式选择 `Embedded`。

**Gallery：** 在仓库根目录执行 `cargo run -p rinui-gallery -- --dev /absolute/path/to/Rin-UI` 可加载文件系统资源。该目录还应包含 `examples/gallery.qml`；若仅提供 RinUI 目录，需另传 `--examples-dir /absolute/path/to/examples` 或设置 `RINUI_EXAMPLES_DIR`。这些是 Gallery 参数，不是通用 `RinUIHost` 参数。

## 应用资源与 QRC

RinUI 内嵌资源**不会自动包含**应用自己的 QML、图片和翻译。开发时使用绝对路径与 `QmlSource::File` 加载。QML 相对 URL 以声明它的 QML 文件为基准，例如在 `qml/main.qml` 中使用 `source: "assets/images/logo.png"`。

发布时需单独打包应用资源。在最小工程中添加以下 Cargo 配置与文件。此例仅内嵌 `main.qml`；请将引用的页面、图片、翻译逐个加入，并保持别名之间的相对路径。

将下列依赖加入已有的 `[dependencies]`，然后新增构建依赖节：

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

在 `run()` 中创建宿主前注册应用资源，并将文件加载替换为 QRC URL：

```rust
cxx_qt::init_crate!(rinui_starter);
// ... create host ...
host.load(&QmlSource::Url("qrc:/app/main.qml".to_owned()))?;
```

删除不再使用的 `PathBuf` 导入，重新运行 `cargo run`。宿主会初始化其自身的内嵌 RinUI 资源；上述 `init_crate!` 初始化应用自己的资源集合。`QmlSource::Url` 使用 `qrc:/app/main.qml`，Qt 文件名 API 需要资源路径时使用 `:/app/...`。

应用资源应避开保留的 `qrc:/rinui` 命名空间。自定义 QML 模块目录需在加载前通过 `host.add_import_path(...)` 或 `configure_engine(...)` 注册。

## 发布打包

使用 `cargo build --release -p rinui-gallery` 构建 Gallery。在 `rinui-rs` 根目录执行以下脚本，可参考其部署方式：

```bash [macOS / Linux]
bash scripts/package_macos.sh
bash scripts/package_linux.sh
```

```powershell [Windows]
.\scripts\package_windows.ps1
# MSIX: see packaging/windows/README.md and scripts/package_msix.ps1
```

这些脚本面向 Gallery，负责组织 Qt 库、插件、QML 运行时模块和许可证声明；用于自己的应用时需调整可执行文件与应用资源。内嵌 QRC 不等于免除 Qt 动态库依赖。请脱离开发 checkout 和开发 Qt 路径验证发布包。

## 排查问题与 HiDPI

- 编译期找不到上游资源：初始化子模块，核对 `RINUI_UPSTREAM_DIR`。
- `module "RinUI" is not installed`：查看 `host.import_paths()` 和资源模式，检查 `RINUI_DEV_PATH`。
- QML 加载失败：检查 `HostError::QmlLoadFailed` 与 `host.qml_warnings()`，核对文件名、导入、应用资源别名。
- 缺少 Qt 平台插件或 Qt5Compat 模块：发布包中需携带 Qt 插件与 QML 模块。
- Qt Quick 尺寸与 `font.pixelSize` 已使用逻辑像素，不要乘 `Utils.dpiScale`；`Utils.dp()` 是已弃用的恒等辅助函数。

参见[主题系统](/zh/core/theme)、[辅助工具](/zh/core/utils)和 [Rust Host 接入](rust-host)。

[rinui-host resources.rs](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/crates/rinui-host/src/resources.rs) · [Gallery CLI](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/examples/gallery/src/main.rs)
