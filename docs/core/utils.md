# Utility Tools (`Utils`)

`RinUIHost` initializes the Rust theme backend that supplies persisted colors to `Utils`. See [Configuration and Rust/QML Integration](/guide/rust-host) for configuration. The default stored accent is `#605ed2`; application fonts and icons remain QML resources.

## Overview

`Utils` is a QML singleton for global resources, fonts, and animation durations. Modify its properties to customize the appearance and behavior of your application.

You can use the utility tools `Utils` like this:
```qml
import QtQuick
import RinUI

Text {
    font.family: Utils.fontFamily
    font.pixelSize: 14
    text: "RinUI-rs"
}
```

## Appearance and Theme
| Property                   | Description                                                                                                                                                                                                                          |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| primaryColor               | Theme primary color, default reads the backend configuration file default value `#605ed2` <br/> It is recommended to modify the theme color through [`Theme.setThemeColor(color)`](/core/theme#modes-and-accent-color)                    |
| colors                     | The color table of the current theme, shared by all controls                                                                                                                          |
| appearance                 | Appearance settings of the current theme (such as rounded corners, shadows, opacity)                                                                                                                                                 |
| typography                 | Currently only stores font size information                                                                                                                                                                                          |

## Fonts and Icons
| Property                            | Description                                                                                              |
|-------------------------------------|----------------------------------------------------------------------------------------------------------|
| `fontFamily`                        | Default application font, default is `(Qt.application.font.family)`                                      |
| `iconFontFamily`                    | Font name for font icons (usually provided by `FontIconLoader`)                                          |
| `fontIconSource`                    | Path to the `.ttf` file of the font icon                                                                 |
| `fontIconIndexSource`               | Path to the `.js` file of the font icon index                                                            |
| `fontIconIndex`                     | Icon index object, can directly access characters through Utils.fontIconIndex['icon-name']               |
| `dpiScale`                          | Read-only `devicePixelRatio` of the current screen, intended for diagnostics only                         |

## Animation Durations

| Property                         | Description                              |
|----------------------------------|------------------------------------------|
| `animationSpeed`                 | Standard animation speed (250ms)         |
| `animationSpeedExpander`         | Expander animation speed (333ms)         |
| `animationSpeedFaster`           | Fast animation speed (167ms)             |
| `animationSpeedMiddle`           | Moderately slow speed (667ms)            |
| `appearanceSpeed`                | Interface switching speed (187ms)        |
| `progressBarAnimationSpeed`      | Progress bar animation time (1550ms)     |

## Sizing and DPI

`Utils.dp(value)` is deprecated. It remains for source compatibility, but it is now an identity function: it returns the input unchanged and logs a deprecation warning on first use.

Qt Quick dimensions such as `width`, `height`, and `font.pixelSize` already use logical pixels. Do not multiply them by the device pixel ratio:

```qml
Text {
    width: 320
    height: 48
    font.pixelSize: 14
    text: "RinUI-rs"
}
```

Avoid `Utils.dp(320)` and do not manually multiply by `Utils.dpiScale`; doing so applies HiDPI scaling twice.
