# テーマ（`Theme`）

## 初期化

`Theme` は外観を管理する RinUI の QML シングルトンです。`RinUIHost` は QML 読み込み前にバックエンドを初期化します。独自エンジンの接続方法は [Runtime と Session](/ja/guide/runtime-sessions)を参照してください。

```qml
import QtQuick
import RinUI

Item {
    Component.onCompleted: Theme.setTheme(Theme.mode.Auto)
}
```

## モードとアクセント {#modes-and-accent-color}

| API | 動作 |
| --- | --- |
| `Theme.setTheme(Theme.mode.Light / Dark / Auto)` | ライト、ダーク、システム追従を選択。 |
| `Theme.toggleMode()` | 実際の表示をライト／ダークで切り替え。Auto を含む循環ではありません。 |
| `Theme.getTheme()` | 設定モード：`"Light"`、`"Dark"`、`"Auto"`。 |
| `Theme.currentTheme.isDark` / `Theme.isDark()` | Auto も含めた実際の表示。 |
| `Theme.setThemeColor(color)` | 16 進文字列または QML color でアクセントを設定。 |
| `Theme.getThemeColor()` | 現在の QML color。初期化前は `"transparent"`。 |

`HostConfig.config_location` でテーマ設定の保存方法を指定します。保存設定と Rust からの呼び出しは [設定と Rust/QML 連携](/ja/guide/rust-host)を参照してください。

## Windows の背景効果

ウィンドウの読み込み後に `Theme.setBackdropEffect(Theme.effect.Mica / Acrylic / Tabbed / None)` を使います。`Theme.getBackdropEffect()` は `"mica"`、`"acrylic"`、`"tabbed"`、`"none"` を返します。

| 効果 | 対応プラットフォーム |
| --- | --- |
| Mica | Windows 11。 |
| Acrylic | Windows 10。Windows 11 21H2 では Mica にフォールバック。 |
| Tabbed | Windows 11 22H2 以降。Windows 11 21H2 では Mica にフォールバック。 |
| None | Windows の背景材質を無効化。他の OS は通常の背景を使用。 |

Rust からの背景効果の設定と戻り値は [設定と Rust/QML 連携](/ja/guide/rust-host)を参照してください。

[FluentWindow](/ja/Windowing/FluentWindow)、[プロジェクト構成とリソース](/ja/guide/project-structure)も参照してください。
