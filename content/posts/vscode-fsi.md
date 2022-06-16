---
title: "Visual Studio Code - Running F# scripts from VS Code tasks"
date: 2022-06-17
draft: false
tags: ['vscode','fsharp', 'fsi']
---

Below is an reference tasks.json file showing a single task that runs an F# script.

``` json
{
  "version": "2.0.0",
  "tasks": [
    {
      "type": "shell",
      "command": "dotnet fsi ./MyScript.fsx",
      "label": "My Script",
      "problemMatcher": [],
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "presentation": {
        "reveal": "silent",
        "panel": "dedicated",
        "showReuseMessage": false,
        "clear": true
      }
    }
  ]
}
```
