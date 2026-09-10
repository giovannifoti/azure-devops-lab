# UD04 — Laboratorio autonomo — Archivio documentale controllato

## Scenario

L'applicazione deve conservare documenti consultati frequentemente per 30 giorni e file temporanei eliminabili dopo un giorno. Gli utenti applicativi devono leggere i documenti senza ricevere account key e l'accesso pubblico anonimo non è consentito.

Il laboratorio autonomo riutilizza lo storage account creato nel laboratorio guidato.

---

## 1. Scelta del servizio

Per questo scenario sceglierei **Azure Blob Storage**.

I dati da gestire sono documenti e file organizzati come oggetti, quindi il modello Blob è coerente con il requisito. Gli altri servizi non sono equivalenti:

- **Azure Files** è adatto quando applicazioni o utenti richiedono una file share con struttura di directory accessibile tramite SMB/NFS;
- **Queue Storage** è pensato per messaggi semplici e disaccoppiamento asincrono tra componenti;
- **Table Storage** è un archivio NoSQL chiave/attributi per dati schemaless.

Il requisito riguarda quindi oggetti documentali, non una condivisione filesystem, una coda di messaggi o dati tabellari NoSQL.

---

## 2. Valutazione LRS e ZRS

Nel laboratorio è stato utilizzato **Standard_LRS** perché è una scelta economica e sufficiente per dati didattici, temporanei e ricostruibili.

In produzione, se il requisito include resilienza rispetto al guasto di una singola Availability Zone nella region primaria, sceglierei **ZRS**. ZRS replica i dati tra zone distinte della stessa region e fornisce quindi protezione rispetto a un guasto zonale, mentre LRS mantiene più copie localmente senza offrire la stessa resilienza zonale.

La maggiore resilienza di ZRS comporta normalmente un costo superiore rispetto a LRS, quindi la scelta deve dipendere dai requisiti di disponibilità e continuità del servizio.

---

## 3. Creazione del container privato `archive`

È stato creato da CLI il secondo container:

```text
archive
```

usando autenticazione Microsoft Entra:

```bash
az storage container create \
  --account-name "$LAB_STORAGE" \
  --name archive \
  --auth-mode login \
  --public-access off
```

Esito:

```text
Created: True
```

Il container è quindi stato creato come privato, senza accesso pubblico anonimo.

---

## 4. Caricamento e verifica del Blob

È stata caricata una copia di:

```text
consegne/UD04/01_DOCUMENTO_LAB.txt
```

nel container `archive` con nome:

```text
current/documento.txt
```

usando `--auth-mode login`.

La verifica del Blob ha restituito:

```text
Blob:  current/documento.txt
Tier:  Hot
Bytes: 49
```

Il documento è quindi presente nel container previsto, nel tier `Hot`, con dimensione coerente con il file originale.

---

## 5. User delegation SAS con minimo privilegio

È stata progettata e generata una **user delegation SAS** con le seguenti caratteristiche:

```text
Risorsa:   singolo Blob archive/current/documento.txt
Permesso:  sola lettura (r)
Durata:    massimo 15 minuti
Metodo:    Microsoft Entra ID / user delegation
Trasporto: HTTPS
```

La SAS è stata utilizzata esclusivamente per scaricare il singolo Blob previsto.

Il file ottenuto tramite SAS è stato confrontato con l'originale tramite `cmp`:

```text
exit code: 0
```

Il codice `0` conferma che il file scaricato tramite SAS coincide con l'originale.

La variabile contenente l'URL firmato è stata eliminata immediatamente:

```text
SAS_URL rimossa
```

Nessun token SAS o URL firmato è riportato in questa consegna.

---

## 6. Analisi degli errori

### `AuthorizationPermissionMismatch`

**Piano:** data plane.

**Causa probabile:** il principal autenticato riesce ad autenticarsi ma non possiede sullo scope corretto un ruolo dati sufficiente, per esempio `Storage Blob Data Reader` o `Storage Blob Data Contributor`, oppure la nuova role assignment non si è ancora propagata.

**Controllo:** verificare account autenticato, ruolo dati, principal, scope della role assignment e attendere la propagazione prima di ripetere l'operazione. Non è corretto risolvere assegnando automaticamente ruoli più ampi.

### `ResourceNotFound: The specified container does not exist`

**Piano:** data plane.

**Causa probabile:** il container indicato non esiste nello storage account interrogato oppure è stato usato un nome di container/account errato.

**Controllo:** verificare storage account, nome esatto del container e presenza della risorsa, per esempio tramite una lista dei container usando un metodo di autenticazione autorizzato.

### `curl: (22) The requested URL returned error: 403`

**Piano:** data plane/accesso mediante SAS.

**Causa probabile:** il server ha ricevuto la richiesta ma non la autorizza. Possibili motivi sono SAS scaduta, permessi insufficienti, SAS riferita a una risorsa diversa o firma/token non più valido per la richiesta eseguita.

**Controllo:** verificare senza pubblicare il token che la SAS sia ancora valida, che abbia il permesso richiesto, che sia limitata alla risorsa corretta e che la richiesta utilizzi HTTPS. Se necessario, generare una nuova SAS minima invece di riutilizzare token scaduti o ampliare inutilmente i permessi.

---

## 7. Relazione tra prefisso e lifecycle management

La regola lifecycle del laboratorio guidato usa il prefisso:

```text
documents/temporary/
```

Il prefisso include sia il nome del container sia il percorso Blob.

Il Blob del laboratorio autonomo si trova invece in:

```text
archive/current/documento.txt
```

Di conseguenza non soddisfa il filtro della regola `delete-temporary`: appartiene a un container diverso (`archive` invece di `documents`) e a un percorso diverso (`current/` invece di `temporary/`).

La lifecycle rule non si applica quindi a `archive/current/documento.txt`.

---

## 8. Driver di costo e cleanup

I principali driver di costo dello scenario sono:

- capacità occupata;
- tipo di ridondanza, per esempio LRS o ZRS;
- access tier dei Blob;
- numero e tipo di operazioni;
- eventuali costi di recupero/rehydration;
- trasferimento dei dati.

Il cleanup deve essere eseguito soltanto dopo laboratorio autonomo e verifica finale, seguendo un ordine controllato:

1. rimuovere la role assignment `Storage Blob Data Contributor` creata per il laboratorio;
2. eliminare i file temporanei locali;
3. eliminare eventuali variabili sensibili rimaste nella shell;
4. eliminare il Resource Group del laboratorio, che rimuove storage account, container e Blob contenuti;
5. attendere il completamento della cancellazione;
6. verificare che `az group exists --name "$LAB_RG"` restituisca `false`.

Le SAS non vengono pubblicate e, oltre alla loro naturale scadenza, nessun token viene conservato nei file della consegna.

---

## Esito

Il laboratorio autonomo ha verificato:

- uso di Blob Storage coerente con il modello dati richiesto;
- confronto tra LRS e ZRS;
- creazione del container privato `archive`;
- caricamento e verifica di `current/documento.txt`;
- accesso al singolo Blob tramite user delegation SAS di sola lettura e durata massima 15 minuti;
- rimozione immediata della variabile contenente la SAS;
- distinzione tra problemi di autorizzazione, risorsa inesistente e accesso SAS negato;
- corretta separazione tra il prefisso lifecycle `documents/temporary/` e il Blob `archive/current/documento.txt`;
- pianificazione del cleanup senza pubblicazione di segreti.
