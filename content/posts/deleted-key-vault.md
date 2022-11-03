---
title: "A vault with the same name already exists in deleted state"
date: 2022-11-03
draft: false
tags: ['azure']
---

When setting up a deployment pipeline with Azure DevOps I came across the following error:

```bash
A vault with the same name already exists in deleted state. You need to either recover or purge existing key vault. Follow this link https://go.microsoft.com/fwlink/?linkid=2149745 for more information on soft delete.
```

The cause of the error was that after a previous attempt to deploy I decided to clean up the resource groups I created, which included an Azure Key Vault.
This resulted in a 'soft deleted' Key Vault, which marks the vault as deleted for a specified period (90 days by default). The vault can then be recovered before the retention period expires.

But how do I 'hard delete' my Key Vault? The answer is you must 'purge' the Key Vault and is pretty straight forward to do so via the Azure CLI.

```bash
# Log in to the Azure CLI
az login

# List all the soft deleted Key Vaults in the currrently selected subscription
az keyvault list-deleted

# Purge the Key Vault
az keyvault purge --name mykeyvault
```