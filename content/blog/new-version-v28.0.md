---
title: "New release v28.0"
summary: "New release v28.0.20241202.37 is available for download."
date: 2024-12-02
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: false
homepage: false
---

{{< callout context="caution" title="Important" icon="outline/alert-triangle" >}} This release contains a security update to address the vulnerability [CVE-2024-43485](https://github.com/advisories/GHSA-8g4q-xg66-9fp4) in System.Text.Json 6.0.x. {{< /callout >}}

## Overview

Link to this releasae: [Click here](https://github.com/Yvand/EntraCP/releases/tag/v28.0.20241202.37).  
Changelog in this version: [Click here](https://github.com/Yvand/EntraCP/blob/master/CHANGELOG.md#entracp-v2802024120237---enhancements--bug-fixes---published-in-december-12-2024).  
To update from a previous version of EntraCP: [Follow this article]({{< relref "/docs/usage/update" >}}).  
To upgrade from AzureCP: [See this article]({{< relref "/docs-azurecp/guides/upgrade-to-entracp" >}}).

## About the security update

EntraCP has [2 dependencies](https://github.com/Yvand/EntraCP/blob/6af9dbab552341948aae58b2b523e9b70d92a2ef/Yvand.EntraCP/Yvand.EntraCP.csproj#L130): [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) and [Azure.Identity](https://www.nuget.org/packages/Azure.Identity/).  
The [vulnerability](https://github.com/advisories/GHSA-8g4q-xg66-9fp4) was disclosed on October 8, 2024, and affects `System.Text.Json` 6.0.x, which was referenced by `Azure.Identity`.  
EntraCP v28 uses the latest version of both packages, which depend on `System.Text.Json` 8.0.5, that fixes the vulnerability.
