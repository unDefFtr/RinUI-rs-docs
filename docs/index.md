---
layout: home

hero:
  name: "Rust + Qt Quick"
  text: |-
    Fluent desktop apps
    with Rust and QML
  tagline: |-
    Build desktop apps with Rust and Qt Quick.
    Ready-to-use controls, themes, and navigation.
  image:
    light:
      src: /assets/images/brand/rinui-vertical-light.png
      width: 128
      height: 128
    dark:
      src: /assets/images/brand/rinui-vertical-dark.png
      width: 128
      height: 128
    alt: RinUI
  actions:
    - theme: brand
      text: Get Started
      link: /guide/getting-started
    - theme: alt
      text: Components
      link: /BasicInput/Button

features:
  - icon:
      src: /assets/images/home/icons/navigationview.png
      alt: NavigationView
      width: 48
      height: 48
    title: Complete desktop shell
    details: Build navigation, pages, title bars, and window materials with QML controls designed to work together.
    link: /Navigation/NavigationView
    linkText: NavigationView
  - icon:
      src: /assets/images/home/icons/colorpicker.png
      alt: ColorPicker
      width: 48
      height: 48
    title: One theme everywhere
    details: Light, dark, and automatic mode share one token system, accent color, and motion language.
    link: /core/theme
    linkText: Theme system
  - icon:
      src: /assets/images/home/icons/button.png
      alt: Button
      width: 48
      height: 48
    title: Familiar QML controls
    details: Use familiar control names with polished Fluent states, from basic input to collections and dialogs.
    link: /BasicInput/Button
    linkText: Browse controls
---

## Why RinUI-rs

RinUI-rs brings Fluent controls, themes, and navigation to Rust desktop applications. Build the interface in QML and manage application startup, configuration, and platform integration in Rust.

- **A complete window structure**: `FluentWindow`, `NavigationView`, and `FluentPage` work together out of the box.
- **Native desktop behavior**: Use Mica, Acrylic, custom title bars, and standard window controls on Windows.
- **One coherent design system**: Controls, themes, icons, and translation resources work together across the application.

<GalleryCarousel section="home" />

## Where RinUI-rs fits

RinUI-rs is intended for projects that need a desktop window and complete interactions while keeping the development speed of QML:

- Desktop tools and content managers built with Rust and Qt Quick.
- Internal applications that want a Fluent visual language without rebuilding the window shell and control states from scratch.
- Prototypes or actively developed products that need navigation, settings pages, dialogs, tables, date selection, and theme switching.

RinUI-rs is not a replacement for a web component library, nor is it intended for projects that must remain entirely on Qt Widgets.

## Start with QML you already know

<div align="center">
  <img src="/assets/images/docs/FluentWindow/example.png" alt="Start with QML you already know">
</div>

```qml
import QtQuick
import RinUI

FluentWindow {
    visible: true
    width: 960
    height: 640

    navigationItems: [
        {
            title: "Home",
            icon: "ic_fluent_home_20_regular",
            page: Qt.resolvedUrl("pages/Home.qml")
        }
    ]
}
```

Load the QML from Rust. The complete Cargo setup is in [Getting Started](/guide/getting-started):

```rust
use rinui_host::{AppIdentity, HostConfig, QmlSource, ResourceMode, RinUIHost};
use std::{path::PathBuf, process::ExitCode};

fn run() -> Result<i32, Box<dyn std::error::Error>> {
    let mut host = RinUIHost::new(HostConfig {
        identity: AppIdentity::new("RinUIDocs", "Starter"),
        resource_mode: ResourceMode::Embedded,
        ..HostConfig::default()
    })?;
    let qml = PathBuf::from(env!("CARGO_MANIFEST_DIR")).join("qml/main.qml");
    host.load(&QmlSource::File(qml))?;
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

Continue with:

- [Getting started](/guide/getting-started)
- [Window and page structure](/Windowing/FluentWindow)
- [NavigationView](/Navigation/NavigationView)
- [Theme system and window backdrops](/core/theme)

## Acknowledgments

RinUI-rs is maintained by [unDefFtr](https://github.com/unDefFtr/rinui-rs). QML controls, reused documentation and images: upstream RinUI by RinLit and contributors. Screenshots illustrate upstream QML components.

### Upstream Rin-UI contributors

[![Upstream Rin-UI contributors](https://contrib.rocks/image?repo=RinLit-233-shiroko/Rin-UI)](https://github.com/RinLit-233-shiroko/Rin-UI/graphs/contributors)

### rinui-rs contributors

[![rinui-rs contributors](https://contrib.rocks/image?repo=unDefFtr/rinui-rs)](https://github.com/unDefFtr/rinui-rs/graphs/contributors)
