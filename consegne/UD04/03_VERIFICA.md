# UD04 — Verifica — Azure Storage

## Parte A — Scelta singola

### 1. Quale servizio è più adatto a immagini applicative accessibili come oggetti HTTP?

**Risposta: A — Blob**

Azure Blob Storage è progettato per archiviare e accedere a dati come oggetti, ad esempio immagini, documenti, log e contenuti applicativi.

### 2. Quale servizio offre condivisioni file SMB gestite?

**Risposta: B — Files**

Azure Files fornisce condivisioni file gestite accessibili tramite SMB e, negli scenari supportati, NFS.

### 3. Contributor sullo storage account consente automaticamente lettura Blob via Entra ID?

**Risposta: B — No, serve un ruolo del piano dati**

`Contributor` riguarda la gestione della risorsa storage nel management plane. Per leggere o modificare i Blob tramite Microsoft Entra ID servono ruoli del data plane, come `Storage Blob Data Reader` o `Storage Blob Data Contributor`.

### 4. Quale ridondanza protegge da un guasto zonale nella region primaria?

**Risposta: B — ZRS**

ZRS replica i dati tra zone distinte della stessa region e protegge quindi da un guasto zonale nella region primaria.

### 5. Quale metodo concede un accesso delegato con permessi e scadenza?

**Risposta: B — SAS**

Una Shared Access Signature può delegare accesso a una risorsa specifica con permessi definiti e una scadenza.

### 6. Un budget Azure Storage:

**Risposta: C — segnala una soglia ma non blocca i consumi**

Un budget serve a monitorare la spesa e generare avvisi al raggiungimento di determinate soglie; non interrompe automaticamente il consumo.

### 7. Una lifecycle rule con prefisso `documents/temporary/` interessa:

**Risposta: B — soltanto Blob che corrispondono al filtro**

La regola viene applicata solo ai Blob che soddisfano il prefisso e gli altri filtri configurati.

### 8. Perché `--auth-mode login` è importante?

**Risposta: A — rende esplicito l'uso dell'identità Entra**

L'opzione indica alla CLI di usare l'identità autenticata tramite Microsoft Entra ID invece di affidarsi a una account key.

---

## Parte B — Risposte brevi

### 9. Distingui ridondanza e backup.

La **ridondanza** mantiene più copie dei dati per aumentare la resilienza rispetto a guasti infrastrutturali, zonali o geografici, a seconda della configurazione scelta.

Il **backup** serve invece a recuperare dati dopo eventi logici come cancellazioni, sovrascritture o modifiche indesiderate.

Le due funzioni non sono equivalenti: una modifica o cancellazione può essere replicata insieme ai dati, quindi avere più copie non significa avere automaticamente un backup.

### 10. Distingui management plane e data plane con un comando per ciascuno.

Il **management plane** riguarda la gestione della risorsa Azure tramite Azure Resource Manager.

Esempio:

```bash
az storage account show \
  --resource-group "$LAB_RG" \
  --name "$LAB_STORAGE"
```

Questo comando legge le proprietà dello storage account.

Il **data plane** riguarda invece le operazioni sui dati memorizzati nel servizio.

Esempio:

```bash
az storage blob list \
  --account-name "$LAB_STORAGE" \
  --container-name "$LAB_CONTAINER" \
  --auth-mode login
```

Questo comando legge l'elenco dei Blob nel container usando l'identità Microsoft Entra.

### 11. Elenca quattro proprietà di una SAS a minimo privilegio.

Una SAS coerente con il principio del minimo privilegio deve avere:

1. **permessi minimi necessari**, ad esempio sola lettura se non serve scrittura;
2. **scope più ristretto possibile**, per esempio un singolo Blob invece dell'intero account;
3. **scadenza breve**, limitata al tempo necessario;
4. **uso tramite HTTPS**, evitando trasmissione non protetta.

Inoltre il token non deve essere pubblicato in repository, log o screenshot.

### 12. Spiega perché Archive non è appropriato per dati da recuperare immediatamente.

Il tier **Archive** è offline e richiede una fase di reidratazione prima che il Blob torni accessibile. Per questo non è appropriato quando il requisito prevede recupero immediato o tempi di accesso molto brevi.

Archive è adatto a dati consultati raramente, quando tempi e costi di recupero sono accettabili.

### 13. Perché non bisogna salvare account key o SAS nel repository?

Perché account key e SAS sono credenziali o token di accesso.

Una **account key** concede un accesso molto ampio allo storage account. Una **SAS** può essere più limitata, ma finché è valida permette comunque le operazioni delegate.

Se questi segreti vengono pubblicati in un repository possono essere copiati e utilizzati da soggetti non autorizzati. Per questo non devono essere inseriti in file versionati, commit, screenshot o output condivisi.

---

## Parte C — Caso situazionale

Un'applicazione espone documenti privati. Il tecnico assegna `Contributor` allo storage account, omette `--auth-mode login`, condivide una account key e crea una SAS con permessi completi senza scadenza breve.

### 14. Individua almeno tre problemi.

Ci sono almeno quattro problemi:

1. **`Contributor` non è il ruolo corretto per il solo accesso ai Blob tramite Entra ID.** È un ruolo del management plane e concede capacità di gestione della risorsa più ampie del necessario.
2. **Omettere `--auth-mode login` non rende esplicito l'uso dell'identità Microsoft Entra.** Il laboratorio richiede invece di verificare l'accesso con identità.
3. **Condividere una account key è ad alto impatto.** La chiave concede accesso molto ampio e non applica il principio del minimo privilegio.
4. **Una SAS con permessi completi e senza scadenza breve è eccessiva.** Aumenta inutilmente la superficie di rischio perché concede più operazioni e per più tempo del necessario.

### 15. Proponi autorizzazione e scope più appropriati.

Se l'applicazione deve soltanto leggere documenti privati, assegnerei un ruolo del data plane come:

```text
Storage Blob Data Reader
```

al principal dell'applicazione, preferibilmente una managed identity o un service principal, sullo **scope minimo necessario**.

Se i documenti si trovano in un solo container, lo scope preferibile è quel container. In questo modo l'applicazione può leggere i Blob necessari senza ricevere `Contributor` sullo storage account e senza ricevere account key.

Se serve una SAS per un accesso temporaneo, dovrebbe essere limitata al singolo Blob o al minimo scope necessario, con sola lettura e scadenza breve.

### 16. Indica come verificheresti accesso e cleanup senza pubblicare segreti.

Per verificare l'accesso userei comandi che esplicitano Microsoft Entra ID senza mostrare chiavi o token, per esempio:

```bash
az storage blob list \
  --account-name "$LAB_STORAGE" \
  --container-name "$LAB_CONTAINER" \
  --auth-mode login \
  --output table
```

Per una SAS verificherei l'accesso salvando l'URL firmato soltanto in una variabile temporanea, senza stamparlo:

```bash
curl --fail --silent --show-error "$SAS_URL" --output /tmp/verifica.txt
```

Dopo la verifica eliminerei immediatamente la variabile:

```bash
unset SAS_URL
```

Nel cleanup finale:

1. rimuoverei la role assignment dati creata per il laboratorio;
2. eliminerei i file temporanei locali;
3. eliminerei eventuali variabili sensibili rimaste nella shell;
4. eliminerei il Resource Group del laboratorio;
5. attenderei il completamento della cancellazione;
6. verificherei:

```bash
az group exists --name "$LAB_RG"
```

Il risultato finale atteso è:

```text
false
```

Nessuna account key, SAS, URL firmato, Subscription ID, Object ID o dato personale deve essere riportato nella consegna.
