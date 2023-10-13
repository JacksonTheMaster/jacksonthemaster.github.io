---
title: "ExOnline: Get information about Microsoft 365 Groups and display it in a grid view window."
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
$groups = Get-UnifiedGroup | Select DisplayN*, AccessType ,HiddenFromAddressListsEnabled, HiddenFromExchangeClientsEnabled, ManagedByDetails, WhenC* | Out-GridView
````

The command is using the Get-UnifiedGroup cmdlet to view Microsoft 365 Groups in your cloud-based organization. It selects some properties of the groups, such as Display Name, Access Type, Hidden Status, Managed By Details, and Creation Date. It then outputs the results to a grid view window where you can interact with the data.

To run this command, you need to connect to Exchange Online PowerShell first by using the Connect-ExchangeOnline cmdlet. You may also want to use Set-UnifiedGroup cmdlet to modify Microsoft 365 Groups.