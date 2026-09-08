# Consegna UD02 — Laboratorio autonomo

## Requisito e piano

Lo scenario richiede la creazione di un ambiente Azure temporaneo e separato dagli altri laboratori, composto da un resource group dedicato, una rete virtuale con una subnet specifica e uno storage account vuoto. Il resource group viene utilizzato come confine del ciclo di vita perché tutte le risorse dello scenario devono poter essere gestite ed eliminate insieme. La località scelta è `italynorth`, già verificata durante il laboratorio guidato, così da mantenere coerenza tra gli ambienti. La VNet fornisce lo spazio di rete privato necessario al workload, mentre lo storage account fornisce uno spazio di archiviazione indipendente dalla rete. I tag permettono di identificare corso, unità, ambiente, scenario e data prevista di eliminazione. Il cleanup è considerato completato soltanto quando, dopo l'eliminazione del resource group, il comando `az group exists` restituisce `false`.

- requisito interpretato: ambiente Azure temporaneo di sviluppo, isolato dagli altri laboratori e completamente eliminabile come unica unità.
- risorse previste: resource group dedicato, VNet `vnet-cea-auto`, subnet `snet-workload` e storage account `StorageV2` Standard LRS.
- nomi e tag scelti: resource group `rg-cea-ud02-auto-bba1caf1`, VNet `vnet-cea-auto`, subnet `snet-workload`, storage account `stceaautobba1caf1`; tag `course=cloud-engineer-academy`, `unit=UD02`, `environment=dev`, `scenario=autonomous`, `deleteAfter=2026-09-10`.
- verifiche preliminari: sottoscrizione verificata come `Enabled` e predefinita, località `italynorth`, disponibilità del nome dello storage account e controllo delle variabili prima della creazione.

## Svolgimento

L'ambiente è stato creato interamente tramite Azure CLI, come richiesto dallo scenario.

Il resource group `rg-cea-ud02-auto-bba1caf1` è stato creato in `italynorth` e verificato con stato di provisioning `Succeeded`. I cinque tag previsti risultano presenti.

La VNet `vnet-cea-auto` è stata creata nello stesso resource group con spazio di indirizzamento `10.30.0.0/16`. La subnet `snet-workload` è stata verificata con prefisso `10.30.10.0/24`. Anche la VNet contiene i tag richiesti.

Lo storage account `stceaautobba1caf1` è stato creato come `StorageV2`, SKU `Standard_LRS`, con HTTPS obbligatorio, TLS minimo `TLS1_2` e accesso Blob pubblico anonimo disabilitato. Lo stato di provisioning è `Succeeded` e i tag richiesti sono presenti.

Comandi di verifica essenziali:

```bash
az group show   --name "$AUTO_RG"   --query "{Name:name,Location:location,State:properties.provisioningState,Tags:tags}"   --output jsonc
```

Output ripulito:

```text
Name: rg-cea-ud02-auto-bba1caf1
Location: italynorth
State: Succeeded
Tags: course, unit, environment, scenario, deleteAfter
```

```bash
az network vnet show   --resource-group "$AUTO_RG"   --name "$AUTO_VNET"   --query "{Name:name,Location:location,Address:addressSpace.addressPrefixes,Tags:tags}"   --output jsonc
```

Output ripulito:

```text
Name: vnet-cea-auto
Location: italynorth
Address: 10.30.0.0/16
Tags: course, unit, environment, scenario, deleteAfter
```

```bash
az network vnet subnet show   --resource-group "$AUTO_RG"   --vnet-name "$AUTO_VNET"   --name "$AUTO_SUBNET"   --query "{Name:name,AddressPrefix:addressPrefix,AddressPrefixes:addressPrefixes}"   --output jsonc
```

Output ripulito:

```text
Name: snet-workload
AddressPrefix: 10.30.10.0/24
```

```bash
az storage account show   --resource-group "$AUTO_RG"   --name "$AUTO_STORAGE"   --query "{Name:name,Location:location,Kind:kind,Sku:sku.name,HttpsOnly:enableHttpsTrafficOnly,MinimumTls:minimumTlsVersion,PublicBlobAccess:allowBlobPublicAccess,State:provisioningState,Tags:tags}"   --output jsonc
```

Output ripulito:

```text
Name: stceaautobba1caf1
Location: italynorth
Kind: StorageV2
Sku: Standard_LRS
HttpsOnly: true
MinimumTls: TLS1_2
PublicBlobAccess: false
State: Succeeded
Tags: course, unit, environment, scenario, deleteAfter
```

Inventario CLI:

```text
Name               Type                                Location    Environment  Scenario    DeleteAfter
-----------------  ----------------------------------  ----------  -----------  ----------  -----------
vnet-cea-auto      Microsoft.Network/virtualNetworks   italynorth  dev          autonomous  2026-09-10
stceaautobba1caf1  Microsoft.Storage/storageAccounts   italynorth  dev          autonomous  2026-09-10
```

ID anonimizzati conservati nella relazione:

```text
/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-auto-bba1caf1/providers/Microsoft.Network/virtualNetworks/vnet-cea-auto
/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-auto-bba1caf1/providers/Microsoft.Storage/storageAccounts/stceaautobba1caf1
```

La verifica indipendente tramite Azure Portal ha mostrato lo stesso insieme di risorse osservato tramite CLI. Nel portale sono stati inoltre ricontrollati spazio di indirizzamento della VNet, subnet, proprietà principali dello storage account e tag, tutti coerenti con i requisiti dello scenario.

## Diagnosi

- errore o anomalia analizzata: nessuna anomalia bloccante durante la creazione; il nome dello storage account è risultato disponibile al primo controllo.
- ipotesi: la configurazione soddisfa i requisiti se resource group, VNet, subnet, storage account e tag coincidono con i valori richiesti e se portale e CLI mostrano lo stesso inventario.
- controllo: verifica separata tramite `az group show`, `az network vnet show`, `az network vnet subnet show`, `az storage account show`, `az resource list` e controllo indipendente tramite Azure Portal.
- correzione: non è stata necessaria alcuna correzione durante lo scenario autonomo.
- verifica successiva: tutti i controlli CLI e la verifica tramite portale hanno confermato i valori richiesti.

## Cleanup e consegna

- risorse eliminate: resource group `rg-cea-ud02-auto-bba1caf1` e tutte le risorse contenute, incluse VNet, subnet e storage account.
- controllo finale: `az group exists --name "$AUTO_RG"`.
- risultato finale del cleanup: `false`, quindi il resource group autonomo non esiste più.
- hash abbreviato e messaggio del commit: da completare dopo il commit `Completa lo scenario Azure autonomo`.
