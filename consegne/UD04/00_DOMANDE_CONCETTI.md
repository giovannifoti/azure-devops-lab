# Consegna UD04 — Domande di controllo

1. **Perché Azure Blob e Azure Files non sono intercambiabili?**  
   Azure Blob e Azure Files rispondono a modelli di accesso differenti. Blob organizza i dati come oggetti all'interno di container ed è adatto, per esempio, a immagini, documenti, log, backup e contenuti applicativi. Azure Files espone invece condivisioni e directory accessibili tramite SMB o NFS ed è pensato per applicazioni o utenti che richiedono una vera file share. La scelta dipende quindi dal modello di utilizzo dei dati, non soltanto dal loro volume.

2. **Qual è la differenza tra management plane e data plane?**  
   Il **management plane** riguarda la gestione della risorsa Azure tramite Azure Resource Manager: per esempio creare uno storage account, modificarne le impostazioni di rete o leggerne le proprietà. Il **data plane** riguarda invece le operazioni sui dati contenuti nel servizio, come caricare, leggere o eliminare un Blob. I due piani sono distinti e possono richiedere autorizzazioni differenti.

3. **Perché Contributor sullo storage account non implica accesso Blob con Entra ID?**  
   Perché `Contributor` è un ruolo del management plane e consente di gestire la risorsa storage, ma non concede automaticamente permessi sui dati Blob tramite Microsoft Entra ID. Per accedere al data plane servono ruoli specifici, come `Storage Blob Data Reader` o `Storage Blob Data Contributor`, assegnati allo scope appropriato.

4. **Perché geo-ridondanza e backup risolvono problemi diversi?**  
   La geo-ridondanza replica i dati in più copie e, in alcune configurazioni, anche in una seconda region per aumentare la resilienza rispetto a guasti infrastrutturali o regionali. Un backup serve invece a recuperare dati dopo eventi logici come cancellazioni, sovrascritture o modifiche indesiderate. Una cancellazione può infatti essere replicata insieme ai dati, quindi più copie non equivalgono automaticamente a un backup.

5. **Quali fattori valuteresti prima di scegliere Archive?**  
   Prima di scegliere il tier `Archive` valuterei la frequenza prevista di accesso, il tempo massimo accettabile per il recupero, la necessità di reidratazione, i costi di recupero e delle operazioni e l'eventuale permanenza minima richiesta. Un file non dovrebbe essere spostato in Archive soltanto perché è vecchio: bisogna verificare che il suo profilo di utilizzo sia compatibile con un tier offline.

6. **Perché una account key ha un impatto maggiore di una SAS limitata?**  
   Una account key concede un accesso molto ampio allo storage account e rappresenta quindi un segreto ad alto impatto. Una SAS può invece essere limitata a specifiche operazioni, risorse e durata. Se progettata correttamente, una SAS riduce la superficie di rischio perché delega soltanto l'accesso strettamente necessario per un periodo definito.

7. **Quali proprietà rendono una SAS coerente con il minimo privilegio?**  
   Una SAS è coerente con il principio del minimo privilegio quando usa permessi minimi necessari, scope più ristretto possibile, scadenza breve e trasporto HTTPS. Inoltre non deve essere pubblicata in repository, log o screenshot. Quando applicabile, una `user delegation SAS` è preferibile a una SAS firmata con account key perché utilizza credenziali Microsoft Entra invece della chiave condivisa dell'account.

8. **Perché non possiamo verificare una policy lifecycle aspettando pochi minuti?**  
   Una policy di lifecycle management non viene eseguita come un job immediato. Le regole vengono valutate ed elaborate in modo asincrono e l'elaborazione può iniziare dopo ore. Per questo pochi minuti non sono sufficienti per dimostrare che una regola abbia effettivamente spostato o eliminato i Blob selezionati; la verifica deve tenere conto dei tempi di elaborazione del servizio e dei filtri configurati.
