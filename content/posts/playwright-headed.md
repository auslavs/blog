---
title: "Playwright - Running tests in headed mode with Jetbrains Rider"
date: 2025-01-16
draft: false
tags: ['dotnet','playwright', 'rider']
---

Playwright is a cross-platform, cross-browser tool for testing web apps. It supports .NET, Node.js, Python, and Java.
In my experience, support varies quite a bit between Node.js, which I understand to be the most popular way of using the API, and the .NET API.

It has an extension for VS Code, which is not supported when using the .NET API. However, it seems to work well with Visual Studio, and the docs focus on this IDE.

If you want to run tests in *Headed* mode, meaning you can see the browser while you are running tests, they suggest that you use a RunSettings file.c I think this is supposed to work in JetBrains Rider, but I've not had any success. Instead, you can enable headed mode by editing the project's run configuration and entering "HEADED=1" into the environment variables text input.

![JetBrains Rider Run Configuration](/images/rider-run-configuration.png "JetBrains Rider Run Configuration")

You can now run your tests in headed mode and see the browser while the tests are running.

For futher information, see the [Playwright documentation](https://playwright.dev/dotnet/docs/running-tests) on running and debugging tests.