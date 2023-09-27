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

## Configure the proxy for Windows

EntraCP connects to multiple, HTTPS Microsoft Graph sites, and Windows will try to validite their certificates using their CRL endpoints.  
If Windows cannot connect to those CRL endpoints, the typical behavior is random timeouts that last for a few minutes, during which EntraCP (and SharePoint) hangs.  
To avoid this, you also need to configure the proxy for Windows using `netsh winhttp` command ([more info](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/configure-proxy-internet)):

```shell
# Show proxy configuration
netsh winhttp show proxy
# Set proxy configuration
netsh winhttp set proxy <proxy>:<port>
netsh winhttp set proxy "10.0.0.6:8080"
# Reset proxy configuration
netsh winhttp reset proxy
```
