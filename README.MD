# RinUI-rs Documentation

English, 简体中文 and 日本語 documentation for [RinUI-rs](https://github.com/unDefFtr/rinui-rs). Build Fluent-style desktop applications with Rust and Qt Quick (QML).

Application UI uses QML. Rust handles startup, configuration, translation and platform integration. This repository contains the VitePress documentation site.

## Read the guides

- [English](docs/guide/index.md) · [简体中文](docs/zh/guide/index.md) · [日本語](docs/ja/guide/index.md)
- [Getting Started](docs/guide/getting-started.md): RinUI-rs Gallery and a complete minimal Rust application.
- [Configuration and Rust/QML Integration](docs/guide/rust-host.md): configuration, theme, translation and Rust/QML interaction.
- [Runtime and Sessions](docs/guide/runtime-sessions.md): multiple engines and existing Qt applications.
- [Project Structure and Resources](docs/guide/project-structure.md): resource modes, application QRC and packaging.

## Develop this documentation site

Install Node.js and npm, then run:

```bash
npm ci
npm run docs:dev
```

Open the local URL printed by VitePress. To validate and preview the production site:

```bash
npm run docs:build
npm run docs:preview
```

The Guide sidebar is shared by the `guide/`, `core/`, and `Design/` routes for each locale. When adding a Guide article under a new route prefix, map that prefix to the same localized sidebar in `docs/.vitepress/config.mts`.

The Rust examples expect a complete `rinui-rs` checkout beside the example application, including its upstream submodule. See the getting-started guides for Qt and C++ requirements.

## Attribution and license

RinUI-rs and documentation adaptation: Copyright © 2026 unDefFtr.
Upstream RinUI QML components, reused documentation and images: Copyright © 2025 RinLit and contributors. Screenshots retained from upstream illustrate the shared QML components.

Thanks to [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI), [CXX-Qt](https://github.com/KDAB/cxx-qt), Qt Quick, Fluent Design, Fluent UI System Icons and [VitePress](https://github.com/vuejs/vitepress).

This documentation is licensed under the [MIT License](LICENSE). Dependency notices are maintained in [RinUI-rs/THIRD_PARTY_NOTICES.md](https://github.com/unDefFtr/rinui-rs/blob/5895b3f/THIRD_PARTY_NOTICES.md).
