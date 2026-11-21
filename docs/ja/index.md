---
layout: home

hero:
  name: "Rust + Qt Quick"
  text: "Rust ネイティブホストで使う Fluent QML コントロール"
  tagline: "Rust / CXX-Qt 6 と RinUI QML で、ウィンドウ、ナビゲーション、テーマを備えるデスクトップアプリを構築します。"
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
      text: はじめに
      link: /ja/guide/getting-started
    - theme: alt
      text: コンポーネント
      link: /ja/BasicInput/Button

features:
  - icon:
      src: /assets/images/home/icons/navigationview.png
      alt: NavigationView
      width: 48
      height: 48
    title: デスクトップアプリの土台
    details: NavigationView、FluentPage、タイトルバーを組み合わせ、ナビゲーションとページを一貫して構築できます。
    link: /ja/Navigation/NavigationView
    linkText: NavigationView
  - icon:
      src: /assets/images/home/icons/colorpicker.png
      alt: ColorPicker
      width: 48
      height: 48
    title: アプリ全体で統一されたテーマ
    details: ライト、ダーク、自動モードが同じカラー、アクセント、モーションを共有します。
    link: /ja/core/theme
    linkText: テーマシステム
  - icon:
      src: /assets/images/home/icons/button.png
      alt: Button
      width: 48
      height: 48
    title: 使い慣れた QML コントロール
    details: 親しみやすいコントロール名を保ちながら、Fluent の状態、入力、コレクション、ダイアログを提供します。
    link: /ja/BasicInput/Button
    linkText: コントロールを見る
---

## RinUI を選ぶ理由

`rinui-rs` は上流 RinUI コントロールに再利用可能な Rust / CXX-Qt 6 ホストを提供します。UI は QML のまま、起動、設定、プラットフォーム統合は Rust が担当します。

- **完成したウィンドウ構成**：`FluentWindow`、`NavigationView`、`FluentPage` をそのまま組み合わせられます。
- **ネイティブなデスクトップ体験**：Windows で Mica、Acrylic、カスタムタイトルバー、標準ウィンドウ操作に対応します。
- **統一されたデザインシステム**：Rust が QML コントロール、テーマ、アイコン、翻訳をホストします。

<GalleryCarousel section="home" />

## RinUI が向いているプロジェクト

RinUI は、デスクトップウィンドウと充実したインタラクションが必要で、QML の開発速度も保ちたいプロジェクトに向いています。

- Rust と Qt Quick で構築するデスクトップツールやコンテンツ管理アプリ。
- Fluent の視覚言語を取り入れつつ、ウィンドウ構成やコントロール状態を一から作らずに済ませたい社内アプリケーション。
- ナビゲーション、設定ページ、ダイアログ、テーブル、日付選択、テーマ切り替えを必要とするプロトタイプや継続開発中の製品。

Web コンポーネントライブラリの代替ではなく、Qt Widgets だけで構成する必要があるプロジェクトにも適していません。

## 使い慣れた QML から始める

<div align="center">
  <img src="/assets/images/docs/FluentWindow/example.png" alt="使い慣れた QML から始める">
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
            title: "ホーム",
            icon: "ic_fluent_home_20_regular",
            page: Qt.resolvedUrl("pages/Home.qml")
        }
    ]
}
```

Rust から QML を読み込みます。完全な Cargo 設定は[はじめに](/ja/guide/getting-started)を参照してください：

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

次のページも参照してください：

- [はじめに](/ja/guide/getting-started)
- [ウィンドウとページの構成](/ja/Windowing/FluentWindow)
- [NavigationView](/ja/Navigation/NavigationView)
- [テーマとウィンドウ背景](/ja/core/theme)

## 謝辞

Rust ホストは [unDefFtr/rinui-rs](https://github.com/unDefFtr/rinui-rs)、QML コントロールと再利用した文書・画像は RinLit と上流 RinUI の貢献者によるものです。画像は上流 QML コンポーネントを紹介します。

[![コントリビューター](http://contrib.nn.ci/api?repo=rinlit-233-shiroko/Rin-UI)](https://github.com/RinLit-233-shiroko/Rin-UI/graphs/contributors)
