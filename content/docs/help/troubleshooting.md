---
title: "Troubleshooting"
description: ""
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
menu:
  docs:
    parent: "help"
    identifier: "troubleshooting"
weight: 300
toc: true
---

This article groups tips & tricks to help you troubleshoot EntraCP if it's not working as expected.

## Inspect the SharePoint logs

EntraCP records all its activity in the SharePoint logs, under Product / Area "EntraCP". It records a lot of information and can be managed with PowerShell:

```powershell
# Show / set the logging level for EntraCP
Get-SPLogLevel| ?{$_.Area -like "EntraCP"}
"EntraCP:*"| Set-SPLogLevel -TraceSeverity Verbose

# Merge EntraCP logs from all SharePoint servers from the last 10 minutes
Merge-SPLogFile -Path "C:\Data\EntraCP_logging.log" -Overwrite -Area "EntraCP" -StartTime (Get-Date).AddMinutes(-10)
```

You can use [ULS Viewer](https://www.microsoft.com/en-us/download/details.aspx?id=44020) to inspect the logs.

## Use the built-in trobleshooting page

EntraCP comes with a built-in troubleshooting page, located in `16\TEMPLATE\ADMIN\EntraCP\TroubleshootEntraCP.aspx`.  
It is primarily designed to:

- Validate that the assembly bindings were correctly set in the `machine.config` file.
- Test the connection to your Microsoft Entra ID tenant, without using the EntraCP configuration.

It is written entirely with inline code, which means that you can easily modify as you need.  
For security reasons, by default it can be called only from the central administration site, but it you can copy the page anywhere under `16\TEMPLATE\LAYOUTS`, and you'll be able to call it from any SharePoint site.

## Test the connectivity with Microsoft Graph

EntraCP may fail to connect to Microsoft Graph for various reasons. The PowerShell script below connects to the typical Microsoft Graph endpoints and may be run on the SharePoint servers to test the connectivity (set or remove the proxy settings depending on your configuration):

```powershell
Invoke-WebRequest -Uri "https://login.microsoftonline.com" -UseBasicParsing [-ProxyUseDefaultCredentials] [-Proxy "http://127.0.0.1:8888"]
Invoke-WebRequest -Uri "https://graph.microsoft.com" -UseBasicParsing [-ProxyUseDefaultCredentials] [-Proxy "http://127.0.0.1:8888"]
```

## Obtain the access token using PowerShell

This PowerShell script requests an access token to Microsoft Graph, as done by EntraCP (set or remove the proxy settings depending on your configuration):

```powershell
$clientId = "<CLIENTID>"
$clientSecret = "<CLIENTSECRET>"
$tenantName = "<TENANTNAME>.onmicrosoft.com"
$headers = @{ "Content-Type" = "application/x-www-form-urlencoded" }
$body = "grant_type=client_credentials&client_id=$clientId&client_secret=$clientSecret&resource=https%3A//graph.microsoft.com/"
$response = Invoke-RestMethod "https://login.microsoftonline.com/$tenantName/oauth2/token" -Method "POST" -Headers $headers -Body $body [-ProxyUseDefaultCredentials] [-Proxy "http://127.0.0.1:8888"]
$response | ConvertTo-Json
```

## Inspect the traffic to Microsoft Graph

You can intercept the traffic between EntraCP (running in SharePoint) and Microsoft Graph using [Fiddler Classic](https://www.telerik.com/fiddler/fiddler-classic) as a local proxy.  
Once Fiddler was installed locally and its root certificate trusted (mandatory), you can intercept the traffic by updating the web.config file of your SharePoint site as below:

```xml
<system.net>
    <defaultProxy useDefaultCredentials="True">
        <proxy proxyaddress="http://localhost:8888" bypassonlocal="False" />
    </defaultProxy>
</system.net>
```

{{< alert icon="💡" text="To view the traffic in Fiddler, make sure to set the filter to \"All Processes\" or \"Non-Browsers\" (in the bottom left)." />}}
