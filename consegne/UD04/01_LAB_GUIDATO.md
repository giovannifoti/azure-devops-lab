# UD04 — Laboratorio guidato — Dall'account al Blob protetto

## 1. Verifica strumenti e repository

Il laboratorio è stato eseguito dal repository personale:

```text
~/workspace/azure-devops-lab
```

Azure CLI era già installata e autenticata, quindi non è stato necessario reinstallare strumenti.

La sottoscrizione Azure prevista risultava attiva con stato `Enabled`.

Nella consegna non vengono riportati Subscription ID, Object ID, tenant ID o dati personali.

---

## 2. Nomi e file di laboratorio

Sono state definite le seguenti variabili:

```text
LAB_SUFFIX=892c6163
LAB_RG=rg-cea-storage-892c6163
LAB_STORAGE=stcea892c6163
LAB_CONTAINER=documents
LAB_LOCATION=italynorth
LAB_DELETE_AFTER=2026-09-11
```

Su macOS il calcolo della data è stato adattato usando la sintassi BSD:

```bash
LAB_DELETE_AFTER="$(date -u -v+1d +%F)"
```

È stato creato il file tecnico richiesto:

```text
consegne/UD04/01_DOCUMENTO_LAB.txt
```

Contenuto:

```text
Documento didattico UD04 - nessun dato personale
```

La verifica del nome dello storage account ha restituito:

```text
NameAvailable: True
```

---

## 3. Creazione Resource Group e Storage Account da CLI

È stato creato il Resource Group:

```text
rg-cea-storage-892c6163
```

nella region:

```text
italynorth
```

con i tag:

```text
course=cloud-engineer-academy
unit=UD04
environment=lab
deleteAfter=2026-09-11
```

È stato quindi creato lo Storage Account:

```text
stcea892c6163
```

con configurazione:

```text
Kind: StorageV2
SKU: Standard_LRS
HTTPS only: true
Minimum TLS: TLS1_2
Anonymous Blob public access: false
Access tier: Hot
Provisioning state: Succeeded
```

La verifica delle proprietà ha confermato:

```json
{
  "BlobEndpoint": "https://stcea892c6163.blob.core.windows.net/",
  "Https": true,
  "Kind": "StorageV2",
  "PublicBlob": false,
  "Sku": "Standard_LRS",
  "Tls": "TLS1_2"
}
```

Non sono state lette o pubblicate chiavi durante questa fase.

---

## 4. Primo container e primo Blob dal portale

Dal portale Azure è stato creato il container:

```text
documents
```

con livello di accesso:

```text
Private (no anonymous access)
```

Nel container è stato caricato il file:

```text
01_DOCUMENTO_LAB.txt
```

mantenendo il tier `Hot`.

Il Blob reale è stato quindi verificato con il nome:

```text
01_DOCUMENTO_LAB.txt
```

La traccia riportava successivamente `documento-lab.txt`; durante il laboratorio è stato mantenuto il nome reale del Blob caricato, evitando una richiesta verso un oggetto inesistente.

---

## 5. Ruolo dati con scope minimo

Sul solo Storage Account è stata assegnata al principal del laboratorio la role assignment:

```text
Storage Blob Data Contributor
```

Lo scope è rimasto limitato allo Storage Account e non è stato esteso all'intera sottoscrizione.

Dopo la propagazione RBAC, la verifica con Microsoft Entra ID tramite:

```bash
az storage container list   --account-name "$LAB_STORAGE"   --auth-mode login
```

ha restituito il container:

```text
documents
```

La verifica Blob ha restituito:

```text
Blob                  Tier   Bytes
--------------------  -----  -----
01_DOCUMENTO_LAB.txt  Hot    49
```

Questo dimostra che il ruolo dati era operativo sul data plane.

---

## 6. Operazioni Blob successive da CLI

È stato creato un file temporaneo locale:

```text
/tmp/ud04-temporaneo.txt
```

ed è stato caricato nel container `documents` come:

```text
temporary/temporaneo.txt
```

L'upload è stato completato correttamente.

È stato poi scaricato il Blob originale usando il nome reale:

```text
01_DOCUMENTO_LAB.txt
```

nel file locale:

```text
/tmp/documento-lab-scaricato.txt
```

Il confronto:

```bash
cmp   consegne/UD04/01_DOCUMENTO_LAB.txt   /tmp/documento-lab-scaricato.txt
```

ha restituito exit code:

```text
0
```

quindi il file scaricato coincide con l'originale.

La lista finale dei Blob nel container `documents` era:

```text
01_DOCUMENTO_LAB.txt      Hot  49
temporary/temporaneo.txt  Hot  30
```

---

## 7. Confronto Shared Key senza esposizione

Una account key è stata recuperata temporaneamente in una variabile di ambiente:

```text
AZURE_STORAGE_KEY
```

La chiave non è mai stata stampata, copiata nella consegna o inserita in output condivisi.

È stata utilizzata per una sola operazione di lettura:

```bash
az storage blob list   --account-name "$LAB_STORAGE"   --container-name "$LAB_CONTAINER"   --auth-mode key
```

L'operazione ha restituito:

```text
01_DOCUMENTO_LAB.txt
temporary/temporaneo.txt
```

Subito dopo la variabile è stata rimossa:

```text
AZURE_STORAGE_KEY rimossa
```

Il confronto mostra perché Microsoft Entra ID è preferibile per utenti e applicazioni moderne: consente accesso tramite identità e ruoli, mentre una account key è un segreto condiviso con privilegi molto più ampi.

---

## 8. User delegation SAS

Su macOS la scadenza è stata calcolata con:

```bash
SAS_EXPIRY="$(date -u -v+30M '+%Y-%m-%dT%H:%MZ')"
```

È stata generata una **user delegation SAS** con queste proprietà:

```text
Risorsa: singolo Blob 01_DOCUMENTO_LAB.txt
Container: documents
Permesso: r (sola lettura)
Durata: 30 minuti
Autenticazione: Microsoft Entra ID / user delegation
Trasporto: HTTPS
```

L'URL firmato è stato conservato esclusivamente in una variabile temporanea e non è mai stato pubblicato.

Il download tramite SAS è riuscito e il confronto con l'originale ha restituito:

```text
cmp exit code: 0
```

La variabile è stata quindi eliminata:

```text
SAS_URL rimossa
```

Nessun token SAS o URL firmato compare nella consegna.

---

## 9. Lifecycle management

Dal portale Azure è stata creata la regola:

```text
delete-temporary
```

con configurazione:

```text
Enabled: true
Blob type: block blob
Prefix: documents/temporary/
Delete after last modification: 1 day
```

La verifica CLI ha restituito:

```json
[
  {
    "DeleteAfter": 1.0,
    "Enabled": true,
    "Name": "delete-temporary",
    "Prefixes": [
      "documents/temporary/"
    ]
  }
]
```

La regola è quindi limitata al percorso `documents/temporary/` e non coinvolge tutti i Blob dell'account.

Non è stata attesa l'eliminazione effettiva del Blob perché lifecycle management è asincrono e l'elaborazione può iniziare dopo ore; il laboratorio richiede di verificare la configurazione della policy.

---

## 10. Cleanup finale

Il cleanup è stato eseguito dopo il laboratorio autonomo e la verifica.

È stata rimossa la role assignment temporanea:

```text
Storage Blob Data Contributor
```

creata per il laboratorio sullo Storage Account.

Sono stati eliminati i file temporanei locali:

```text
/tmp/ud04-temporaneo.txt
/tmp/documento-lab-scaricato.txt
/tmp/documento-sas.txt
/tmp/documento-archive-sas.txt
```

Sono state inoltre rimosse le variabili sensibili eventualmente presenti nella shell:

```text
AZURE_STORAGE_KEY
SAS_URL
```

La verifica ha confermato:

```text
Variabili sensibili rimosse
```

Il Resource Group eliminato era:

```text
rg-cea-storage-892c6163
```

È stato avviato il cleanup con:

```bash
az group delete   --name "$LAB_RG"   --yes   --no-wait
```

ed è stato atteso il completamento con:

```bash
az group wait   --name "$LAB_RG"   --deleted
```

La verifica finale:

```bash
az group exists --name "$LAB_RG"
```

ha restituito:

```text
false
```

Il risultato `false` conferma che il Resource Group e le risorse contenute non esistono più.

---

## Esito finale

Il laboratorio ha permesso di verificare:

- creazione ripetibile di Resource Group e Storage Account da CLI;
- differenza tra management plane e data plane;
- container Blob privato e upload dal portale;
- accesso dati tramite Microsoft Entra ID con `Storage Blob Data Contributor`;
- upload, download e verifica di integrità da CLI;
- confronto controllato con Shared Key senza pubblicarla;
- user delegation SAS limitata per risorsa, permessi e durata;
- lifecycle management filtrato sul prefisso `documents/temporary/`;
- cleanup completo con verifica finale `false`;
- assenza di account key, SAS, URL firmati, Subscription ID, Object ID e dati personali nei file di consegna.
