---
title: "ExOnline: New Group"
author: "Jakob Langisch"
date: 2023-02-17 11:02:00 +0800
categories: [Microsoft, ExOnline]
tags: [Microsoft, ExOnline]
pin: false
math: false
mermaid: false
---
powershell
````powershell
New-UnifiedGroup
````
The command New-UnifiedGroup is used to create new Microsoft 365 Groups in your cloud-based organization. Before using this command, you need to connect to Exchange Online PowerShell by using [[Connect-ExchangeOnline]][1](https://learn.microsoft.com/en-us/powershell/module/exchange/new-unifiedgroup?view=exchange-ps). You can specify various parameters to set different settings of a group, such as its display name, primary email address, access type, language, etc. Some parameters are only available on this command and can’t be changed later, such as HiddenGroupMembershipEnabled.
For a current full list of parameters, see the source below.

To add members, owners, and subscribers to Microsoft 365 Groups, use Add-UnifiedGroupLinks

Source: [New-UnifiedGroup (ExchangePowerShell) | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/exchange/new-unifiedgroup?view=exchange-ps)