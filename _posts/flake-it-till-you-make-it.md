---
layout: post
title: Flake it till you make it
subtitle: Excerpt from Soulshaping by Jeff Brown
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [books, test]
---

vscode에 개발환경 내 입맛에 맞게 구축하기
dracula theme 사용
java 개발을 위한 내 눈에 맞는 각각의 색깔 설정
Settings.json
```json
{
  "workbench.colorTheme": "One Dark Pro",
  "editor.fontFamily": "JetBrains Mono",
  "editor.fontSize": 14,
  "editor.fontLigatures": true,
  "editor.lineHeight": 21,
  "editor.semanticTokenColorCustomizations": {
    "[One Dark Pro]": {
      "rules": {
        "variable": "#ffffff",
        "class": "#ffffff",
        "keyword": "#6acc6f",
        "annotation": "#6acc6f",
        "interface": "#ffffff",
        "method": "#78f5da",
        "modifier": "#ff9900",
        "property": "#7fcffd",
        "struct": "#ff0000"
      }
      }
    },
    "editor.tokenColorCustomizations": {
      "textMateRules": [{
        "scope": "keyword.control",
        "settings": {
          "foreground": "#77e9b4"
        }
      }, {
        "scope": "variable",
        "settings": {
          "foreground": "#f1fd83"
        }
      }
  ]},
  "workbench.colorCustomizations": {
    
    "tab.activeBackground": "#282c34",
    "activityBar.background": "#282c34",
    "sideBar.background": "#282c34",
    "tab.activeBorder": "#d19a66",
    "terminal.background": "#222222",
    "terminal.foreground": "#ffffff",
    "terminalCursor.background": "#E0E0E0",
    "terminalCursor.foreground": "#E0E0E0",
    "terminal.ansiBlack": "#000000",
    "terminal.ansiBlue": "#6FB3D2",
    "terminal.ansiBrightBlack": "#B0B0B0",
    "terminal.ansiBrightBlue": "#6FB3D2",
    "terminal.ansiBrightCyan": "#76C7B7",
    "terminal.ansiBrightGreen": "#A1C659",
    "terminal.ansiBrightMagenta": "#D381C3",
    "terminal.ansiBrightRed": "#FB0120",
    "terminal.ansiBrightWhite": "#FFFFFF",
    "terminal.ansiBrightYellow": "#FDA331",
    "terminal.ansiCyan": "#76C7B7",
    "terminal.ansiGreen": "#A1C659",
    "terminal.ansiMagenta": "#D381C3",
    "terminal.ansiRed": "#FB0120",
    "terminal.ansiWhite": "#E0E0E0",
    "terminal.ansiYellow": "#FDA331"
  },
  "terminal.integrated.fontFamily": "D2Coding"
}
```

application.yml
```yml
spring:
  output:
    ansi:
      enabled: always
```

spring frame-work 사용 시 application 파일에 위의 설정을 넣어줘야 vscode terminal에서 색깔 구분을 해준다
