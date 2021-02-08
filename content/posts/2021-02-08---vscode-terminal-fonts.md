---
title: VSCode Terminal Fonts
date: "2021-02-08T20:20:32.669Z"
template: "post"
draft: false
slug: "vscode-terminal-fonts"
category: "quickshots"
tags:
  - "vscode"
  - "terminal"
description: "Just a short tutorial on how you can adjust the font for the VSCode terminal."
socialImage: "/media/vscode.svg"
---
![vscode](/media/vscode.svg.png)

## Introduction

Just a short tutorial on how you can adjust the font for the VSCode terminal.  If you use Oh-My-Zsh and you want to avoid some nasty font issues, you can follow the steps below.

![image-before](/media/image-before.png)

## Steps

### Open the settings

Press ⌘ + Shift + P, search for "Open Settings" and open the settings.json (not UI)

### Choose your font.

Choose your preferered font. (for instance by checking your iterm settings)

### Add parameter

```json
"terminal.external.osxExec": "iTerm.app",
"terminal.integrated.automationShell.osx": "/bin/zsh",
"terminal.explorerKind": "external",
"terminal.integrated.fontFamily": "meslolgs nf"
```

### Done!

Save your work and see the beauty :)

![image-after](/media/image-after.png)


## Additional Links and sources

* https://medium.com/@youngstone89/how-to-change-font-for-terminal-in-visual-studio-code-c3305fe6d4c2

