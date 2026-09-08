# Consegna UD02 — Laboratorio guidato

## Contesto verificato

- Azure Portal accessibile: sì.
- Azure CLI autenticata: sì.
- sottoscrizione corretta verificata senza pubblicarne l'ID: `Azure subscription 1`, stato `Enabled`, impostata come predefinita.
- località scelta e motivo: `italynorth` (Italy North), già verificata e utilizzata per mantenere le risorse del laboratorio nella stessa area geografica.

## Ambiente creato

| Elemento | Nome tecnico | Tipo | Località | Scopo |
|---|---|---|---|---|
| Resource group | `rg-cea-ud02-2088dddf` | `Microsoft.Resources/resourceGroups` | `italynorth` | Raggruppare le risorse temporanee dell'UD02 e costituire il confine di lifecycle e cleanup del laboratorio. |
| Rete virtuale | `vnet-cea-ud02` | `Microsoft.Network/virtualNetworks` | `italynorth` | Fornire lo spazio di indirizzamento privato del laboratorio, con address space `10.20.0.0/16` e subnet `snet-app` `10.20.1.0/24`. |
| Storage account | `stcea2088dddf` | `Microsoft.Storage/storageAccounts` | `italynorth` | Verificare la creazione e le proprietà di un account `StorageV2` Standard con ridondanza LRS, senza caricare dati. |

## Decisioni e verifiche

Le risorse sono state collocate nello stesso resource group perché appartengono allo stesso laboratorio temporaneo, condividono lo stesso ciclo di vita e devono poter essere eliminate insieme al termine dell'attività. Il resource group rappresenta quindi un confine operativo utile sia per l'inventario sia per il cleanup.

Rimangono a carico dell'utente le scelte di configurazione e governo delle risorse, come località, nomi, indirizzamento della rete, impostazioni dello Storage Account, tag, controllo degli accessi e verifica finale della rimozione. Azure gestisce invece l'infrastruttura sottostante necessaria a erogare i servizi.

Sono stati applicati nomi tecnici coerenti e quattro tag comuni (`course`, `unit`, `environment`, `deleteAfter`) per rendere immediatamente riconoscibili appartenenza, ambiente e data prevista di cleanup. Il tag `deleteAfter` è stato impostato a `2026-09-10`.

La VNet è stata verificata da CLI con address space `10.20.0.0/16`; la subnet `snet-app` è risultata configurata con `10.20.1.0/24`. Lo Storage Account è stato verificato con le seguenti proprietà principali: `StorageV2`, `Standard_LRS`, trasferimento HTTPS obbligatorio, TLS minimo `TLS1_2`, accesso Blob pubblico anonimo disabilitato e provisioning `Succeeded`. Non è stato creato alcun Private Endpoint.

L'inventario CLI ha mostrato due risorse principali nel resource group:

```text
Name           Type                                Location    Unit    DeleteAfter
-------------  ----------------------------------  ----------  ------  -----------
vnet-cea-ud02  Microsoft.Network/virtualNetworks   italynorth  UD02    2026-09-10
stcea2088dddf  Microsoft.Storage/storageAccounts   italynorth  UD02    2026-09-10
```

Gli ID sono stati anonimizzati prima di essere conservati:

```text
/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-2088dddf/providers/Microsoft.Network/virtualNetworks/vnet-cea-ud02
/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-2088dddf/providers/Microsoft.Storage/storageAccounts/stcea2088dddf
```

Il portale si è rivelato più efficace per esplorare e comprendere proprietà e opzioni durante la prima creazione delle risorse. La CLI è risultata più adatta per controlli ripetibili e selettivi, perché consente di interrogare direttamente proprietà specifiche come stato di provisioning, tipo, SKU, indirizzamento e tag senza dover navigare tra più schermate.

## Cleanup

- operazione di eliminazione: eliminazione dell'intero resource group `rg-cea-ud02-2088dddf` con `az group delete --name "$LAB_RG" --yes --no-wait`.
- controllo utilizzato: `az group wait --name "$LAB_RG" --deleted`, seguito da `az group exists --name "$LAB_RG"`.
- risultato finale: `false`; il resource group non esiste più e il cleanup del laboratorio guidato è concluso.
- eventuale anomalia e soluzione: nessuna anomalia durante il cleanup; il comando di attesa si è concluso e la verifica finale ha restituito il risultato atteso.

## Rilevanza professionale

Inventario, tag e verifica del cleanup rendono la procedura ripetibile perché permettono di identificare in modo coerente le risorse, ricostruirne il contesto e controllare lo stato reale con comandi deterministici. I tag consentono di classificare ambiente, unità e scadenza operativa; l'inventario rende verificabile ciò che è stato effettivamente creato; il controllo finale del cleanup dimostra che le risorse temporanee sono state rimosse e riduce il rischio di lasciare componenti inutilizzati o soggetti a costi.
