# rinui-rs とは？

rinui-rs は [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI) の Rust ホストです。Rust と Qt Quick（QML）で Fluent スタイルのデスクトップアプリを構築できます。

アプリ UI は引き続き QML を使い、起動、設定、翻訳、エンジン設定、ネイティブ統合は Rust が担当します。独立アプリには `RinUIHost`、複数の利用側や既存 Qt GUI アプリには `RinUIRuntime` と Session を使います。

```qml
import QtQuick
import RinUI
```

## 機能

- 入力、ナビゲーション、コレクション、ダイアログ、状態表示の QML コントロール。
- ライト／ダーク／自動テーマ、アクセントカラー、Windows 背景効果。
- 内蔵 RinUI QRC とファイルシステム開発モード。
- Windows、macOS、Linux のプラットフォーム統合。
- Rust/QML コンテキストプロパティ、QObject バックエンド、型付きルート連携、翻訳カタログ。

## 最初に読むページ

- [はじめに](getting-started)：rinui-rs Gallery と最小 Rust アプリ。
- [プロジェクト構成とリソース](project-structure)：import、リソースモード、QRC、配布。
- [設定と Rust/QML 連携](rust-host)：設定、テーマ、翻訳、Rust/QML 連携。
- [Runtime と Session](runtime-sessions)：共有／専用エンジンと既存 Qt アプリ。
- [コンポーネント](/ja/components/)：QML プロパティ、シグナル、使用例。

## 謝辞

QML コントロール、テーマ、再利用した文書・画像は RinLit と貢献者による [RinUI](https://github.com/RinLit-233-shiroko/Rin-UI) に由来します。rinui-rs は [unDefFtr](https://github.com/unDefFtr/rinui-rs) が管理しています。Qt Quick、CXX-Qt、Fluent Design、Fluent UI System Icons にも感謝します。MIT ライセンスと原著作者の帰属を維持します。
