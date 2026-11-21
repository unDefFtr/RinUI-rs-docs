# What is rinui-rs?

`rinui-rs` is a native Rust / CXX-Qt 6 host for [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI), a Fluent-style Qt Quick (QML) component library.

Application UI remains QML. Rust owns startup, configuration, translations, engine setup and native platform integration. Use `RinUIHost` for a standalone app, or `RinUIRuntime` with sessions when integrating multiple consumers or an existing Qt GUI application.

```qml
import QtQuick
import RinUI
```

## Capabilities

- Familiar QML controls: input, navigation, collections, dialogs and status information.
- Light, dark and automatic theme modes, accent colors and Windows backdrop integration.
- Embedded RinUI QRC resources or filesystem development resources.
- Windows, macOS and Linux platform integration.
- Rust/QML context properties, QObject backends, typed root-object interaction and translation catalogs.

## Start here

- [Getting Started](getting-started): build the Gallery and a minimal Rust application.
- [Project Structure and Resources](project-structure): imports, resource modes, QRC and packaging.
- [Rust Host integration](rust-host): configuration, theme, translation and Rust/QML interaction.
- [Runtime and Sessions](runtime-sessions): shared/private engines and existing Qt applications.
- [Components](/components/): QML properties, signals and usage examples.

## Acknowledgments

The QML controls, themes and reused documentation/images come from [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI) by RinLit and contributors. The Rust host is maintained in [unDefFtr/rinui-rs](https://github.com/unDefFtr/rinui-rs). Thanks also to Qt Quick, CXX-Qt, Fluent Design and Fluent UI System Icons. Original attribution is retained under the MIT License.
