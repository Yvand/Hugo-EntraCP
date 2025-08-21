---
title: "Configuration"
description: "Follow this article to configure EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2021-08-06T11:15:29Z
draft: false
images: []
weight: 115
toc: true
---

Follow this article to configure EntraCP in your SharePoint farm.

EntraCP lets you configure many settings through administration pages, or using PowerShell;

- Add / remove Microsoft Entra ID tenants
- [Set the proxy]({{< relref "docs/how-to/configure-proxy" >}})
- Enable / disable the augmentation
- Customize the display of the permissions
- Customize the claim types

The administration pages can be found in the central administration > Security.

## Configuration using PowerShell

### Add a Microsoft Entra ID tenant

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Add a Microsoft Entra ID tenant using a client secret
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "<TENANT>.onmicrosoft.com"
$tenant.ClientId = "<CLIENTID>"
$tenant.ClientSecret = "<CLIENTSECRET>"
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)

# Add a Microsoft Entra ID tenant using a client certificate
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "<TENANT>.onmicrosoft.com"
$tenant.ClientId = "<CLIENTID>"
$certPath = "<CERTIFICATE_PFX_FILEPATH>"
$certPassword = ConvertTo-SecureString -String "<CERTIFICATE_PFX_PASSWORD>" -Force -AsPlainText
$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($certPath, $certPassword, [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)
$tenant.ClientCertificateWithPrivateKey = $cert
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)
```
