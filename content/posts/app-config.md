---
title: "ConfigurationBuilder - How do I do that again?"
date: 2021-12-28
draft: false
tags: ['dotnet','fsharp']
---

Often when starting a new F# app, there comes a time when I want to read some configuration settings from a file, which can be overridden by environmental variables when running in production. This is closely followed by head-scratching as I try to remember how [ConfigurationBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationbuilder?view=dotnet-plat-ext-6.0) works and what NuGet packages I need.

The below example should set you straight until Microsoft changes the API again, which should be just before the next time I type `dotnet new ...`
<br/><br/>

Code: 
```fsharp
#r "nuget: Microsoft.Extensions.Configuration, 6.0.0"
#r "nuget: Microsoft.Extensions.Configuration.Json, 6.0.0"
#r "nuget: Microsoft.Extensions.Configuration.EnvironmentVariables, 6.0.0"
#r "nuget: Microsoft.Extensions.Configuration.Binder, 6.0.0"

open Microsoft.Extensions.Configuration

[<CLIMutable>]
type Config =
  { Setting1: string
    Setting2: string
    Setting3: string }
  member x.Evaluate =
    { Setting1 = x.Setting1
      Setting2 = x.Setting2
      Setting3 = x.Setting3 }

let GetHostSettings =
  ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .AddEnvironmentVariables()
    .Build()
    .Get<Config>()
```

Settings file:

```json
{
  "Setting1": "xxxxxx",
  "Setting2": "xxxxxx",
  "Setting3": "xxxxxx"
}
```

