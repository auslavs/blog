---
title: "Azure Functions - Help! Why do my function keys keep changing?"
date: 2021-07-23
draft: true
---

I have some Azure Function Apps running as part of an [ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load) process.
It's all the same code base that has barely been updated in 2 years and not in the last year.
So around 12 apps all running the same code, with a complete CI/CD pipeline.

I needed to spin up a new instance which includes:

* A Resource Group
* A Storage Account for the Function App to reside on and store my data in blob storage
* The Function App itself
* Application Insights for monitoring
* Configure some settings

It's such a trivial app, so I usually configure it in the portal. However, I really should have just automated it earlier to prevent the chance of misconfiguration. So I put together some PowerShell using Azure CLI commands and ran it.

The script created the storage account fine but failed on the Function App.
What happened I wonder, so I clear the console and rerun the Function App part anticipating it to fail to inspect the output. But it ran successfully... weird. So I check the portal, and there it is. Well, it must have been a transient error I tell myself and carry on.

I deploy the app using my CI/CD pipeline and run a couple of tests on the production instance. After that, everything is working, so I let the users know they can start using it.

A couple of days later, I notice that data had stopped coming through. I check with the users, and they report that when calling the HTTP endpoint, they are now getting a 401 Unauthorized Error.

