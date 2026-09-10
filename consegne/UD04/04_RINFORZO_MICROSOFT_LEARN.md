# UD04 — Rinforzo Microsoft Learn

## AZ-104: Implement and manage storage in Azure

# 1. Selezione dei moduli

Per il rinforzo UD04 il focus rimane sui quattro ambiti indicati dal percorso:

- Configure storage accounts;
- Configure Azure Blob Storage;
- Configure Azure Storage security;
- Configure Azure Files.

L'obiettivo è consolidare Storage Account, ridondanza, endpoint, Blob, SAS, sicurezza, lifecycle management e differenze tra Blob Storage e Azure Files.

---

# 2. Modulo 1 — Configure storage accounts

## Concetti da fissare

Uno **Storage Account** è il contenitore amministrativo dei servizi Azure Storage. All'interno dello stesso account possono essere esposti servizi come Blob, Files, Queue e Table.

La scelta della ridondanza dipende dal requisito:

- **LRS** replica localmente i dati ed è più economico;
- **ZRS** replica tra zone distinte nella stessa region ed è indicato quando serve resilienza rispetto al guasto di una singola Availability Zone.

Un endpoint Storage può essere tecnicamente raggiungibile ma l'operazione può comunque fallire se l'identità non è autorizzata. Viceversa, un'identità può avere il ruolo corretto ma non riuscire ad accedere se un controllo di rete impedisce la connessione.

---

# 3. Modulo 2 — Configure Azure Blob Storage

La gerarchia concettuale è:

```text
Storage Account
   ↓
Container
   ↓
Blob
```

I tier principali sono:

```text
Hot / Cool / Archive
```

La scelta dipende da frequenza di accesso, costo di conservazione e costo/tempo di recupero.

---

# 4. Modulo 3 — Configure Azure Storage security

Una SAS deve essere progettata secondo il minimo privilegio:

```text
risorsa
+
permessi
+
intervallo temporale
```

Principio:

```text
solo ciò che serve
+
solo dove serve
+
solo per il tempo necessario
```

Non bisogna confondere:

```text
RBAC
→ autorizzazioni di un'identità
```

con:

```text
SAS
→ delega temporanea
```

e:

```text
Account key
→ segreto condiviso molto potente
```

Azure Storage applica inoltre encryption at rest. Le chiavi possono essere gestite dal servizio oppure, negli scenari che lo richiedono, dal cliente tramite customer-managed keys.

---

# 5. Modulo 4 — Configure Azure Files

Azure Files fornisce una **file share gestita**, mentre Blob Storage fornisce **object storage**.

Meccanismi di protezione dati da ricordare:

- snapshot;
- soft delete;
- recupero.

Strumenti utili:

- Portale Azure;
- Azure Storage Explorer;
- Azure CLI;
- AzCopy.

---

# 6. Laboratorio di rinforzo — Storage Security & Lifecycle Challenge

## Scenario

Una società deve archiviare documenti di progetto in Azure con i seguenti requisiti:

1. container non pubblico;
2. accesso amministrativo tramite identità;
3. condivisione temporanea di un singolo Blob in sola lettura;
4. gestione automatica del ciclo di vita;
5. verifica delle impostazioni di rete dello Storage Account;
6. confronto tra Blob Storage e Azure Files.

Configurazione di riferimento:

```text
Resource Group: rg-ud04-rinforzo
Storage Account: stcearinforzo01
Region: italynorth
Account type: StorageV2
SKU: Standard_LRS
```

---

## Fase 1 — Verifica Storage Account

Impostazioni rilevanti:

```text
Kind: StorageV2
Region: italynorth
Redundancy: Standard_LRS
Secure transfer required: Enabled
Minimum TLS version: TLS1_2
Allow Blob anonymous access: Disabled
Default access tier: Hot
```

Servizi disponibili:

```text
Blob
Files
Queue
Table
```

Endpoint:

```text
Blob endpoint:
https://stcearinforzo01.blob.core.windows.net/

File endpoint:
https://stcearinforzo01.file.core.windows.net/
```

Networking da verificare:

```text
Public network access
Network routing
Firewall rules
Private endpoints
```

Le impostazioni di rete stabiliscono se l'endpoint può essere raggiunto. RBAC stabilisce invece se il principal può eseguire l'operazione richiesta. Sono due controlli distinti.

---

## Fase 2 — Container privato

Container:

```text
documents
```

Livello di accesso:

```text
Private
```

Comando:

```bash
az storage container create \
  --account-name stcearinforzo01 \
  --name documents \
  --auth-mode login
```

File da caricare:

```text
report.txt
```

Verifica prevista:

```text
Blob        Tier
----------  -----
report.txt  Hot
```

Il container deve rimanere privato e senza accesso anonimo.

---

## Fase 3 — Accesso tramite identità

Per caricare o modificare Blob tramite Microsoft Entra ID è necessario un ruolo del **data plane**.

Ruolo appropriato:

```text
Storage Blob Data Contributor
```

Scope:

```text
Storage Account stcearinforzo01
```

`Contributor` sullo Storage Account è un ruolo di management plane e non concede automaticamente accesso ai Blob tramite Microsoft Entra ID. `Storage Blob Data Contributor` autorizza invece lettura, scrittura ed eliminazione dei Blob nel data plane.

Comando di verifica:

```bash
az storage blob list \
  --account-name stcearinforzo01 \
  --container-name documents \
  --auth-mode login \
  --output table
```

---

## Fase 4 — SAS temporanea

Per il Blob:

```text
documents/report.txt
```

la SAS deve avere:

```text
Permission: Read
Scope: singolo Blob
Durata: circa 10–15 minuti
Protocollo: HTTPS
Tipo preferibile: user delegation SAS
```

Comando di riferimento:

```bash
SAS_URL="$(az storage blob generate-sas \
  --account-name stcearinforzo01 \
  --container-name documents \
  --name report.txt \
  --permissions r \
  --expiry '<scadenza-15-minuti>' \
  --auth-mode login \
  --as-user \
  --full-uri \
  --output tsv)"
```

Verifica:

```bash
curl --fail --silent --show-error "$SAS_URL" --output /tmp/report-sas.txt
```

Dopo la verifica:

```bash
unset SAS_URL
```

Il token SAS non deve essere riportato nella consegna.

---

## Fase 5 — Lifecycle Management

Regola di riferimento:

```text
Rule name: archive-project-documents
Enabled: true
Blob type: BlockBlob
Prefix: documents/archive/
```

Struttura logica:

```text
scope
+
condition
+
action
```

Esempio:

```text
Scope: documents/archive/
Condition: Blob più vecchio di 30 giorni
Action: spostamento a Cool
```

Un'ulteriore azione può prevedere l'eliminazione dopo un periodo più lungo, se coerente con il requisito.

Non è necessario attendere l'esecuzione della policy perché lifecycle management è asincrono.

---

## Fase 6 — Azure Files

File Share di riferimento:

```text
Name: project-share
Protocol: SMB
Quota: 5 GiB
```

Differenze rispetto al Blob Container:

```text
Blob Container:
- object storage
- accesso a oggetti Blob
- adatto a documenti, immagini, log e contenuti applicativi

Azure File Share:
- file share gestita
- struttura file/directory
- accesso SMB/NFS negli scenari supportati
- adatta ad applicazioni che richiedono semantics da filesystem
```

Snapshot:

```text
Scopo: creare un punto nel tempo utile per il recupero
```

Soft delete:

```text
Scopo: consentire il recupero di una File Share eliminata entro il periodo di retention
```

Azure Storage Explorer può essere utilizzato per navigare account e servizi Storage, gestire container, Blob e file share e trasferire dati. AzCopy è invece uno strumento CLI specializzato nel trasferimento efficiente di dati.

---

# 7. Domande di rinforzo

## 1

Un amministratore può configurare lo Storage Account ma non riesce a caricare Blob con la propria identità.

**Risposta: B — Management plane vs data plane**

La configurazione dello Storage Account appartiene al management plane, mentre l'accesso ai Blob appartiene al data plane e richiede ruoli dati specifici.

## 2

Devi concedere accesso in sola lettura a un singolo Blob per 15 minuti.

**Risposta: C — SAS limitata**

Una SAS permette di limitare scope, permesso e durata.

## 3

Devi aumentare la resilienza rispetto al guasto di una singola Availability Zone mantenendo i dati nella stessa regione.

**Risposta: B — ZRS**

ZRS replica tra zone distinte della stessa regione.

## 4

Quale funzionalità permette di spostare o eliminare automaticamente Blob in base a condizioni temporali?

**Risposta: B — Lifecycle Management**

## 5

Quale servizio è più adatto quando un'applicazione richiede una file share gestita anziché object storage?

**Risposta: B — Azure Files**

## 6

Una SAS con permessi Read/Write/Delete valida per un anno è richiesta per leggere un singolo Blob per pochi minuti.

**Risposta: B — Least privilege**

Concede più permessi e per più tempo del necessario.

## 7

Qual è il principale vantaggio dello snapshot di una Azure File Share?

**Risposta: B — Crea un punto nel tempo utile per recupero**

## 8

Una regola Lifecycle appena creata non ha ancora modificato un Blob. Questo significa necessariamente che la configurazione è errata?

**Risposta: B — No**

Lifecycle Management è asincrono e non agisce necessariamente immediatamente.

---

# 8. Checklist finale

## 1. Storage Account e servizi associati

Lo Storage Account è il contenitore amministrativo di servizi come Blob, Files, Queue e Table.

## 2. LRS vs ZRS

LRS replica localmente; ZRS replica tra Availability Zone della stessa region.

## 3. Management plane vs data plane

Il management plane gestisce la risorsa Azure; il data plane gestisce i dati contenuti nel servizio.

## 4. Blob Container vs Azure File Share

Blob Container = object storage. Azure File Share = file share gestita.

## 5. Hot / Cool / Archive

Hot è adatto a dati consultati frequentemente; Cool a dati meno frequenti; Archive a dati raramente consultati e non immediatamente disponibili.

## 6. SAS vs account key vs Entra ID/RBAC

RBAC autorizza identità, SAS delega accesso temporaneo, account key è un segreto condiviso ad ampio impatto.

## 7. Lifecycle management

Applica azioni automatiche ai Blob sulla base di scope, condizioni e azioni.

## 8. Object replication

Replica oggetti Blob tra Storage Account secondo una policy specifica. È distinta dalla ridondanza interna e dal backup.

## 9. Encryption e customer-managed keys

Azure Storage cifra i dati at rest. Le chiavi possono essere gestite dal servizio oppure dal cliente.

## 10. Snapshots e soft delete

Gli snapshot forniscono punti nel tempo; soft delete permette il recupero di risorse eliminate entro il periodo di retention.

## 11. Storage Explorer / AzCopy

Storage Explorer offre gestione grafica. AzCopy è orientato al trasferimento di dati via CLI.

## 12. Rete e autorizzazione

La rete controlla raggiungibilità e connettività; autorizzazione e RBAC controllano cosa un'identità può fare.

---

# 9. Cleanup

Ordine corretto:

1. rimuovere eventuali role assignment temporanee;
2. eliminare variabili contenenti SAS o credenziali;
3. eliminare file locali temporanei;
4. eliminare il Resource Group temporaneo;
5. attendere la cancellazione;
6. verificare:

```bash
az group exists --name rg-ud04-rinforzo
```

Risultato atteso:

```text
false
```
