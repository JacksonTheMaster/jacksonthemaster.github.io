---
title: Migrate AD User to AAD.md
author: Jacky
date: 2023-08-08 11:33:00 +0800
categories: [Microsoft, Entra]
tags: [Microsoft, Entra]
pin: false
math: false
mermaid: false
---
# Im AD Löschen (oder in Lost&Found verschieben)

Delta Sync durchführen

**ACHTUNG:**

### Nach dem DelaSync etwas warten, dann nochmal DeltaSyncen, da ein Delete immer beim nächsten Sync noch einmal verifiziert wird.

Erst  jetzt kann der user als clouduser restored werden:

Admin Center - > Deleted users - > restore



# Source Anchor / immutable ID vom Cloudobjekt löschen.

```ps
Connect-AzureAD
```

## Object ID ziehen:

```ps
Get-AzureADUser -ObjectId "testuser05@domain.de"| select-object ObjectID
```

### Backup

Zur Sicherheit empfiehlt es sich, die bisherige ImmutableID  während der Arbeiten zwischenzuspeichern. 

```ps
Get-MsolUser -ObjectId 'OBJECTID_AAD' | select-object ImmutableID
```

## ImmutableID nullen

```ps
Connect-MsolService
```

```ps
Set-MsolUser -ObjectId 'OBJECTID_AAD' -ImmutableId "$null"
```

## abschließenden delta sync durchführen

delta sync durchführen
Wenn diese erfolgreich = migriertes cloudobjekt

Wenn man ungeduldig war und es feher wirft, alte immutableID setzen, warten, wiederholen. Beim zweiten mal einfach einen Kaffe dazwischen holen, das passt Zeitlich.

```ps
Set-MsolUser -ObjectId 'OBJECTID_AAD' -ImmutableId "IMMUTABLEID=="
```
