---
title: "Azure Functions - Help! Why do my function keys keep changing?"
date: 2021-10-05
draft: false
---

I created an Azure Functions Application a little while ago and found that the function keys would change every 24 hours.
I should note that this particular app was triggered by an HTTP call every 24 hours.
So it would only work once, and then the caller would be locked out the next time it tried to call the app.

I raised a ticket with Microsoft, and the support engineer let me know that function key changes may be triggered by:

1. A customer revokes/renews function keys or host keys in Portal/Azure CLI
2. A customer makes a PUT/POST/DELETE request via the Key Management API
3. A customer removes the webjobs secret files from the function app's file system or azure-webjobs-secrets blob storage
4. A customer changes MACHINEKEY_DecryptionKey app setting
5. A customer uses keyvault to store the function keys and change the value of it.

But none of these events were applicable to my scenario, and it was looking like the only option was to blow away the app and rebuild it.
This was trivial, but seeing as the app's provisioning was automated, I was worried that it would just recreate the problem.

Through this [reponse](https://github.com/Azure/Azure-Functions/issues/1248#issuecomment-552052187) to a GitHub issue on the Azure Functions repository called out that there can be issued caused by two function apps with the same name used on the same storage account.

How does that apply to my problem? Well, I had used the command line tool [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/what-is-azure-cli) to provision my Function App, and there was a hiccup along the way.

I had used PowerShell script and Azure CLI to provision the app, which included:

* A Resource Group
* A Storage Account for the Function App to reside on and store my data in blob storage
* The Function App itself
* Application Insights for monitoring
* Configuration of some settings, e.g. Set HTTPS only.


The script created the storage account fine but failed on the Function App.
I tried running the part of the script that creates the Function App again to see what happened in the logs, but it succeeded.
Assuming it was some transient error, I just continued on.

So in summary, I had asked to create the Function App before Azure had fully created the Storage Account, causing the creation of the Function App to partially fail.
The support engineer later told me this could take minutes for Azure's internals to sync when creating new services.

This resulted in some remnants of the first Function App still residing in the Storage Account when I reattempted to create the app.
Therefore I had two Function Apps with the same name, stored in the same Storage Account, and whenever the app went through a cold start, it detected something off and re-generated the keys.

Currently, my only fix is to blow away the whole resource group and add in a five-minute delay between creating the Storage Account and the Function App.

I'm sure there is a more elegant solution out there, but it will have to wait for another day.
