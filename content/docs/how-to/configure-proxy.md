---
title: "Configure proxy"
description: ""
lead: ""
date: 2023-09-14
lastmod: 2023-09-14
draft: false
images: []
menu:
  docs:
    parent: "how-to"
    identifier: "configure-proxy"
weight: 200
toc: true
---

In EntraCP, the proxy is now set directly in the EntraCP configuration, you no longer need to set it on all the .config files manually.

## Configure the proxy for EntraCP

The proxy can be set in the central administration > Security > EntraCP global configuration, or using PowerShell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings
$settings.ProxyAddress = "http://localhost:8888"
$config.ApplySettings($settings, $true)
```

## Configure the system proxy for certificate validation

EntraCP connects to multiple Microsoft Graph endpoints, secured with HTTPS.  
Windows will try to validate their certificate by connecting to their CRL endpoints.  
If Windows cannot connect to those CRL, the typical behavior is random timeouts that last for a few minutes, during which EntraCP (and SharePoint) hangs.  
Windows uses the system-wide proxy, which is configured using `netsh winhttp` command ([more info](https://support.microsoft.com/en-us/topic/how-the-windows-update-client-determines-which-proxy-server-to-use-to-connect-to-the-windows-update-website-08612ae5-3722-886c-f1e1-d012516c22a1)):

```shell
# Show proxy configuration
netsh winhttp show proxy
# Set proxy configuration
netsh winhttp set proxy proxyservername:portnumber
# Reset proxy configuration
netsh winhttp reset proxy
```
