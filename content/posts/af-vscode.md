---
title: "Azure Functions - Debugging in VS Code"
date: 2021-07-03
draft: false
---

Dear Future Me,

If you are trying to debug Azure Functions locally using Visual Studio Code and you come across the error:

> "Failed to attach to process: Only 64-bit processes can be debugged."

This is because vscode requires you to run the Azure Functions runtime as a x64 bit process.

The instructions don't normally include this because it assumes you are have installed Azure Functions Core Tools using npm. But if I know you, you probably installed it using chocolatey.

The Azure Functions Core Tools [github repo](https://github.com/Azure/azure-functions-core-tools) has instructions on installing the tools with chocolatey, including a nice little note if you want to use it with vscode.

*Notice: To debug functions under vscode, the 64-bit version is required*
```bash
choco install azure-functions-core-tools-3 --params "'/x64'"
```

Run the install, then you should be able to debug with vscode straight away.