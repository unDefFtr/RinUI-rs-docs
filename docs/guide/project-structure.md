# Project Structure and Resources

Keep Rust startup and business logic in `src/`, and pages and images in `qml/`. RinUI controls remain QML components imported with `import RinUI`. Start from the complete application in [Getting Started](getting-started).

```text
rinui-starter/
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

## Resource modes {#resource-modes}

`HostConfig::default()` uses `ResourceMode::Auto`. This selects `RINUI_DEV_PATH` when nonempty, otherwise embedded RinUI resources. An invalid environment path returns an error; it does not silently fall back.

| Mode | RinUI resource origin |
| --- | --- |
| `Embedded` | Compiled QRC, import prefix `qrc:/rinui`, assets under `:/rinui/RinUI`. |
| `Filesystem(path)` | Checkout containing `RinUI/qmldir`; passing the `RinUI/` directory itself is also supported. |
| `Auto` | Nonempty `RINUI_DEV_PATH`, otherwise `Embedded`. |

At startup, filesystem mode creates a sanitized temporary copy of the RinUI resources and leaves the source files intact. It does not watch files or hot reload them. Keep the host or session alive while the UI is in use. The temporary copy is cleaned up when the last associated engine and resource view are released. In `0.1.0-alpha.2`, the final session can release staged resources on Windows without the icon font holding the files open.

**Build time.** `rinui-host` builds its native bridges from `native/` in the crate and embeds the QML from `vendor/RinUI` by default. Set `RINUI_UPSTREAM_DIR` to use a different source for the embedded QML. It accepts a directory containing `RinUI/qmldir` or the `RinUI/` directory containing `qmldir`. The build fails if the override cannot supply `qmldir`.

**Runtime:** `RINUI_DEV_PATH` changes `Auto` selection. It cannot change resources already compiled into an `Embedded` application. For predictable releases, explicitly select `Embedded`.

**Gallery:** from the repository root, `cargo run -p rinui-gallery -- --dev /absolute/path/to/Rin-UI` selects filesystem resources. The directory must also have `examples/gallery.qml`; for a RinUI-only directory, supply `--examples-dir /absolute/path/to/examples` or `RINUI_EXAMPLES_DIR`. These Gallery options are not general `RinUIHost` options.

## Application files and QRC

RinUI's embedded resources do **not** automatically include your application's QML, images or translations. During development load an absolute path using `QmlSource::File`. QML relative URLs resolve from the declaring QML file, for example `source: "assets/images/logo.png"` from `qml/main.qml`.

For distribution, bundle the application resources separately. Starting from the minimal application, add the following Cargo entries and files. This example only bundles `main.qml`; add every referenced page, image and translation with aliases preserving their relative paths.

Add this line to the existing `[dependencies]` section, followed by a new build-dependencies section:

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

In `run()`, register the application's resource crate before creating the host, and replace the file load with the QRC URL:

```rust
cxx_qt::init_crate!(rinui_starter);
// ... create host ...
host.load(&QmlSource::Url("qrc:/app/main.qml".to_owned()))?;
```

Remove the now-unused `PathBuf` import. Run `cargo run` again. The host initializes its own embedded RinUI resources; `init_crate!` above initializes the application's resource collection. Use `qrc:/app/main.qml` for `QmlSource::Url`, and `:/app/...` where a Qt filename API expects a resource path.

Keep application resources outside the reserved `qrc:/rinui` namespace. Register custom module directories with `host.add_import_path(...)` or `configure_engine(...)` before loading QML.

## Packaging

Build the Gallery with `cargo build --release -p rinui-gallery`. From the `rinui-rs` root, its packaging scripts provide deployment references:

```bash [macOS / Linux]
bash scripts/package_macos.sh
bash scripts/package_linux.sh
```

```powershell [Windows]
.\scripts\package_windows.ps1
# MSIX: see packaging/windows/README.md and scripts/package_msix.ps1
```

These scripts target the Gallery and assemble Qt libraries, plugins, QML runtime modules and license notices; adapt them for your own binary and resources. Embedded QRC does not make Qt dynamically linked dependencies disappear. Test the package away from the development checkout and without development Qt paths.

## Troubleshooting and HiDPI

- `RinUI qmldir not found` during compilation: unset `RINUI_UPSTREAM_DIR` to use the bundled sources or point it to a valid directory. If building the Gallery and the upstream examples are missing, initialize its submodule with `git submodule update --init --recursive`.
- `module "RinUI" is not installed`: inspect `host.import_paths()` and the selected resource mode; check `RINUI_DEV_PATH`.
- QML loading errors: inspect `HostError::QmlLoadFailed` and `host.qml_warnings()`; check the QML filename, imports and application resource aliases.
- Missing Qt platform plugin or Qt5Compat module: deploy Qt's plugins and QML modules alongside the application.
- Qt Quick dimensions and `font.pixelSize` already use logical pixels. Do not multiply by `Utils.dpiScale`; `Utils.dp()` is a deprecated identity helper.

See [Theme](/core/theme), [Utils](/core/utils) and [Configuration and Rust/QML Integration](rust-host).

[rinui-host resources.rs](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/crates/rinui-host/src/resources.rs) · [Gallery CLI](https://github.com/unDefFtr/rinui-rs/blob/v0.1.0-alpha.2/examples/gallery/src/main.rs)
