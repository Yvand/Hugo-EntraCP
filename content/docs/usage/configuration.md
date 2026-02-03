---
title: "Configuration"
description: "Follow this article to configure EntraCP in your SharePoint farm."
lead: ""
date: 2021-05-20T10:45:52Z
lastmod: 2026-01-26
draft: false
images: []
weight: 115
toc: true
---

This page lists the various settings that can be configured in EntraCP, through the administration pages, or using PowerShell.

## Configure the user identifier

{{< callout context="caution" title="Important" icon="outline/alert-triangle" >}} Changing the user identifier settings may impact users with existing permissions. {{< /callout >}}

{{< tabs "user-identifier" >}}
{{< tab "Central administration" >}}

Configure the user identifier using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- Locate the section **Configuration for the user identifier** to update those settings.
- Click on **Ok** to save your changes.

{{< /tab >}}
{{< tab "PowerShell" >}}

Configure the user identifier using Powershell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Show current settings for the user identifier
$settings.ClaimTypes.UserIdentifierConfig

# Update the properties for the user identifier
$settings.ClaimTypes.UpdateUserIdentifier([Yvand.EntraClaimsProvider.Configuration.DirectoryObjectProperty]::UserPrincipalName)
$settings.ClaimTypes.UpdateIdentifierForGuestUsers([Yvand.EntraClaimsProvider.Configuration.DirectoryObjectProperty]::Mail)

# Commit the changes
$config.ApplySettings($settings, $true)
```

{{< /tab >}}
{{< /tabs >}}

## Configure the augmentation

During the augmentation, EntraCP gets the group membership of a user from Entra ID, and sends it to SharePoint. This may happen when users sign-in, when using features such as **Check permissions**, or when processing OAuth2 requests.

{{< callout context="note" title="Note" icon="outline/info-circle" >}} Augmentation works best when the group identifier is set to property **Id** (default setting). {{< /callout >}}

{{< tabs "configure-augmentation" >}}
{{< tab "Central administration" >}}

Configure the augmentation using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- Locate the checkbox **Enable augmentation** in the section **Configuration for the group identifier**.
- Click on **Ok** to save your changes.

{{< /tab >}}
{{< tab "PowerShell" >}}

Configure the augmentation using Powershell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Show if augmentation is enabled
$settings.EnableAugmentation
# Disable the augmentation
$settings.EnableAugmentation = $false

# Commit the changes
$config.ApplySettings($settings, $true)
```

{{< /tab >}}
{{< /tabs >}}

## Configure the group identifier

{{< callout context="caution" title="Important" icon="outline/alert-triangle" >}} Changing the group identifier settings may impact existing group permissions. {{< /callout >}}

{{< tabs "group-identifier" >}}
{{< tab "Central administration" >}}

Configure the group identifier using the Central Administration:

- Navigate to the SharePoint Central Administration > Security > EntraCP Global configuration.
- Locate the section **Configuration for the group identifier** to update those settings.
- Click on **Ok** to save your changes.

{{< /tab >}}
{{< tab "PowerShell" >}}

Configure the group identifier using Powershell:

```powershell
Add-Type -AssemblyName "Yvand.EntraCP, Version=1.0.0.0, Culture=neutral, PublicKeyToken=65dc6b5903b51636"
$config = [Yvand.EntraClaimsProvider.EntraCP]::GetConfiguration()
$settings = $config.Settings

# Show current settings for the group identifier
$settings.ClaimTypes.GroupIdentifierConfig

# Update the property for the group identifier
$settings.ClaimTypes.UpdateGroupIdentifier([Yvand.EntraClaimsProvider.Configuration.DirectoryObjectProperty]::Id)

# Commit the changes
$config.ApplySettings($settings, $true)
```

{{< /tab >}}
{{< /tabs >}}

## Set the credentials

Visit the page [Set the credentials]({{< relref "set-credentials" >}}).

## Set the proxy

Visit the page [Set the proxy]({{< relref "../how-to/configure-proxy" >}}).
