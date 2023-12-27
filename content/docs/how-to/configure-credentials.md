---
title: "Configure the credentials"
description: ""
lead: ""
date: 2023-11-30
lastmod: 2023-12-27
draft: false
images: []
menu:
  docs:
    parent: "how-to"
    identifier: "configure-credentials"
weight: 200
toc: true
---

EntraCP connects to your Entra ID tenant using the credentials defined in the app registration you registered, and supports using either a client secret or a client certificate.

## Prerequisites

- The credentials of the app registration in [your Microsoft Entra ID tenant](https://entra.microsoft.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType//sourceType/Microsoft_AAD_IAM), [created for EntraCP]({{< relref "register-application" >}}).

## Add a new tenant

The tenant can be added from the EntraCP global configuration page in the central administration, or using PowerShell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Add a tenant using a client secret
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "TENANTNAME.onmicrosoft.com"
$tenant.SetCredentials("clientId", "clientSecret")
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)

# Add a tenant using a client certificate
$tenant = New-Object "Yvand.EntraClaimsProvider.Configuration.EntraIDTenant"
$tenant.Name = "TENANTNAME.onmicrosoft.com"
$certificateFilePath = "pfxCertificateFilePath"
$certificatePassword = "pfxCertificatePassword"
$tenant.SetCredentials("clientId", $certificateFilePath, $certificatePassword)
$settings.EntraIDTenants.Add($tenant)
$config.ApplySettings($settings, $true)

# Commit the configuration to the database
$config.Update($true)
```

## Update the credentials of a tenant

The EntraCP administration pages do not have a direct way to update the credentials, but it can be done by deleteding and re-adding the tenant with the new credentials.  
However, this can be achieved using PowerShell:

### Set a new client secret

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$config.UpdateTenantCredentials("tenantNameToUpdate", "newClientSecret")
$config.Update($true)
```

### Set a new client certificate

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$newCertificatePath = "C:\YvanData\config\certs\azurecp.aadcert.pfx"
$newCertificatePassword = "certicicatePassword"
$config.UpdateTenantCredentials("tenantNameToUpdate", $newCertificatePath, $newCertificatePassword)
$config.Update($true)
```
