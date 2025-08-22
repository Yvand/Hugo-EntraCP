---
title: "Announcing EntraCP"
summary: "AzureCP becomes EntraCP, a new claims provider rebuilt from the ground up, with significant improvements."
date: 2023-09-22
lastmod: 2023-12-27
draft: false
weight: 50
images: []
categories: []
tags: []
contributors: []
pinned: true
homepage: true
---

## Main improvements

- A new, greater design.
- The SharePoint solution type was changed to `Application` (instead of `Web Front End`), which makes administrative operations (install / update / delete) much easier and safer by design.
- A single location to configure the proxy for all the SharePoint processes / servers in the farm.
- Revisited NuGet dependencies, upgraded to their latest versions.

## Overview of the changes

- A new solution name: EntraCP.wsp
- A new claims provider: EntraCP
- No compatibility with AzureCP
- The minimum SharePoint version required is 2016
- The minimum .NET Framework required is 4.8
- It no longer works on SharePoint 2013

## Why those changes

Simply put, the weight of legacy: AzureCP had too many early design mistakes, that I could not fix without making breaking changes that created significant risks when updating it. So, I always privileged the most secure approach.

Nevertheless, a few months ago I decided to rewrite the code entirely, knowing it would have to be a completely new, renamed claims provider, with no compatibility with AzureCP.

In the middle of this work, Microsoft [renamed Azure Active Directory to Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/new-name), which made a nice timing with my work, and a new, easy name to pick up.
