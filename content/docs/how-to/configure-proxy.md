---
title: "Configure the proxy"
description: ""
lead: ""
date: 2023-09-14
lastmod: 2023-12-27
draft: false
images: []
toc: true
---

If the SharePoint servers access to internet through a proxy, it must be configured for both EntraCP and Windows (to validate the certificates).

## Configure the proxy for EntraCP

In EntraCP, the proxy is set directly in EntraCP's configuration (no need to set it on any web.config file).  
It can be set in the central administration > Security > EntraCP global configuration, or using PowerShell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings
$settings.ProxyAddress = "http://localhost:8888"
$config.ApplySettings($settings, $true)
```

## Configure the proxy for certificate validation

{{< callout context="caution" title="Important" icon="outline/alert-triangle" >}} The steps below need to be applied in all the SharePoint servers of the farm. {{< /callout >}}

EntraCP connects to Microsoft Graph using HTTPS, and Windows will try to validate the certificates using the links in their CRL.  
If Windows cannot connect to those links, the typical behavior is random timeouts during a few minutes while using the people picker / EntraCP.  
Apply the steps below on each SharePoint server to fully configure the proxy:

- Configure the WinHTTP proxy

Run netsh as shown below in an elevated command prompt:

```shell
# Show the proxy configuration
netsh winhttp show proxy
# Set the proxy
netsh winhttp set proxy proxyservername:portnumber bypass-list="*.contoso.local;<local>"
# Remove the proxy
netsh winhttp reset proxy
```

- Configure the WinINET proxy machine wide:

The PowerShell script below sets the WinINET proxy config machine wide (instead of per-user by default)

```powershell
# Based on https://blog.workinghardinit.work/2020/03/06/configure-wininet-proxy-server-with-powershell/
# Edit the variables below to fit your environment
$proxy = "127.0.0.1:8888"
$proxyEnabled = 1
$bypasslist = "*.contoso.local;<local>"

# Enable machine wide proxy settings (0: per-machine proxy / 1 (or not set): per-user)
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\Internet Settings" -Name "ProxySettingsPerUser" -PropertyType DWORD -Value 0 -Force

$proxyBytes = [system.Text.Encoding]::ASCII.GetBytes($proxy)
$bypassBytes = [system.Text.Encoding]::ASCII.GetBytes($bypasslist)
$defaultConnectionSettings = [byte[]]@(@(70, 0, 0, 0, 0, 0, 0, 0, $proxyEnabled, 0, 0, 0, $proxyBytes.Length, 0, 0, 0) + $proxyBytes + @($bypassBytes.Length, 0, 0, 0) + $bypassBytes + @(1..36 | % { 0 }))

$registryPaths = @("HKLM:\Software\Microsoft\Windows\CurrentVersion\Internet Settings", "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Internet Settings")
foreach ($registryPath in $registryPaths) {
    Set-ItemProperty -Path $registryPath -Name ProxyServer -Value $proxy
    Set-ItemProperty -Path $registryPath -Name ProxyEnable -Value $proxyEnabled
    Set-ItemProperty -Path $registryPath -Name ProxyOverride -Value $bypasslist
    Set-ItemProperty -Path "$registryPath\Connections" -Name DefaultConnectionSettings -Value $defaultConnectionSettings
}

# Run Bitsadmin to set the proxy for localsystem
if ($proxyEnabled -eq 1) {
    Bitsadmin /util /setieproxy localsystem MANUAL_PROXY $proxy $bypasslist
} else {
    Bitsadmin /util /setieproxy localsystem NO_PROXY
}
```
