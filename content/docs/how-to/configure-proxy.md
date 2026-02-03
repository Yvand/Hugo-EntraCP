---
title: "Configure the proxy"
description: ""
lead: ""
draft: false
images: []
toc: true
---

If the SharePoint servers access to internet through a proxy, it must be configured for both EntraCP and Windows (for certificate validation).

## Configure the proxy for EntraCP

{{< callout context="note" title="Note" icon="outline/info-circle" >}} Do NOT set the proxy in web.config files: EntraCP stores the proxy in its own configuration, and uses it in the whole SharePoint farm. {{< /callout >}}

### Limitations

- The proxy must be in the format **http://host[:port]**.
- Due to a limitation in .NET Framework 4.8, only a HTTP proxy is supported. Setting a HTTPS proxy will cause the following error:

`[EntraCP] Unexpected error : Could not authenticate for tenant 'XXX.onMicrosoft.com': NotSupportedException: The ServicePointManager does not support proxies with the https scheme.`

### Set the proxy in EntraCP configuration

{{< tabs "add-credentials" >}}
{{< tab "Central administration" >}}

Browse the central administration > Security > EntraCP global configuration, and locate the section **Proxy**.

{{< /tab >}}
{{< tab "PowerShell" >}}

Run the script below to set / update the proxy:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings
$settings.ProxyAddress = "http://localhost:8888" # Set your proxy address here
$config.ApplySettings($settings, $true)
```

{{< /tab >}}
{{< /tabs >}}

## Configure the proxy for Windows (certificate validation)

Windows needs internet access to check the CRL of the HTTPS certificates, when EntraCP connects to Entra ID.  
If it cannot connect to the CRL endpoints, the typical behavior is a random timeout while using the people picker / EntraCP.

Perform the steps below, on each SharePoint server, to configure the proxy in Windows:

{{< callout context="caution" title="Important" icon="outline/alert-triangle" >}} The steps below must be executed on all the servers in your SharePoint farm. {{< /callout >}}

### Configure the WinHTTP proxy

Run netsh as shown below in an elevated command prompt:

```shell
# Show the proxy configuration
netsh winhttp show proxy
# Set the proxy
netsh winhttp set proxy proxyservername:portnumber bypass-list="*.contoso.local;<local>"
# Remove the proxy
netsh winhttp reset proxy
```

### Configure the WinINET proxy

The PowerShell script below sets the WinINET proxy config machine wide (instead of per-user by default).

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
