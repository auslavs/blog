---
title: "Azure CLI - My commonly used commands"
date: 2023-10-13
draft: false
---

### Subscriptions

#### Subscriptions - List all

```bash
az account list --output table
```

#### Subscriptions - Show current

```bash
az account show --output table
```

#### Subscriptions - Set current

```bash
az account set --subscription "{Subscription Name}"
```

### Function App

#### Function App - List all

```bash
az functionapp list --query "[].{Name:name, Location:location, State:state}" --output table
```

#### Function App - list stopped

```bash
az functionapp list --query "[?state=='Stopped'].{Name:name, Location:location, State:state}" --output table
```

### App Service

#### App Service - list stopped

```bash
az webapp list --query "[?state=='Stopped'].{Name:name, Location:location, State:state}" --output table
```

### Cosmos DB

```bash
az cosmosdb list --output table
```

### Save CLI output as JSON

```bash
az cosmosdb list -o json | Out-File -Path "C:\database.json"
```