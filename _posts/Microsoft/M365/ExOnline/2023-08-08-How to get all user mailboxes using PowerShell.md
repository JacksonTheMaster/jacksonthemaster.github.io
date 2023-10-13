---
title: "ExOnline: Get all user mailboxes"
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
$users = Get-Mailbox -Resultsize Unlimited -RecipientTypeDetails usermailbox | Select *
````

The command $users = Get-Mailbox -Resultsize Unlimited -RecipientTypeDetails usermailbox | Select * is using the Get-Mailbox cmdlet to get all mailboxes that have a recipient type detail of user mailbox. The ResultSize parameter specifies the maximum number of results to return. If you set this parameter to Unlimited, it returns all mailboxes. The Select * part returns all properties of each mailbox.

Before you run this command, you need to connect to Exchange Online PowerShell. You can do this by using the Connect-ExchangeOnline cmdlet.