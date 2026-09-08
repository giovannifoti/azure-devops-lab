# Consegna UD02 — Domande di controllo prima dell'attività pratica

1. **Perché una macchina virtuale lascia al cliente più responsabilità operative rispetto ad App Service?**  
   Una macchina virtuale rientra nel modello IaaS e lascia al cliente la gestione del sistema operativo, degli aggiornamenti, delle configurazioni, del software installato e di una parte maggiore della sicurezza operativa. Con Azure App Service, invece, Microsoft gestisce l'infrastruttura sottostante, il sistema operativo e la piattaforma, mentre il cliente si concentra soprattutto su applicazione, dati, identità, accessi e configurazioni.

2. **Qual è la differenza tra tenant, sottoscrizione e resource group?**  
   Il tenant Microsoft Entra rappresenta il contesto di identità e directory dell'organizzazione. La sottoscrizione Azure è un contenitore amministrativo e di fatturazione associato a un tenant, all'interno del quale vengono create le risorse. Il resource group è invece un contenitore logico interno alla sottoscrizione che raggruppa risorse correlate per facilitarne gestione, governance e ciclo di vita.

3. **Perché la località del resource group non obbliga tutte le risorse a usare la stessa region?**  
   La località del resource group indica dove Azure archivia i metadati del gruppo, non impone automaticamente la region delle risorse contenute. Ogni risorsa può avere una propria località compatibile con il servizio scelto e con i requisiti dell'architettura.

4. **Quale differenza esiste tra una region e un'availability zone?**  
   Una region è un'area geografica Azure che ospita uno o più datacenter. Un'availability zone è una sede fisicamente separata all'interno di una region, con infrastrutture indipendenti, usata per aumentare la resilienza rispetto a guasti locali. Quindi una zone appartiene a una region e non è sinonimo di region.

5. **Perché `az account show` deve precedere la creazione di una risorsa?**  
   `az account show` permette di verificare quale sottoscrizione è attiva, il suo stato e se è quella impostata come predefinita. Questo controllo riduce il rischio di creare risorse nella sottoscrizione sbagliata, con conseguenze su costi, permessi e organizzazione dell'ambiente.

6. **Perché i tag non devono essere usati come meccanismo di sicurezza?**  
   I tag sono metadati usati per classificazione, inventario, governance, cost allocation e automazioni. Non impediscono accessi, non applicano autorizzazioni e non proteggono direttamente una risorsa. La sicurezza deve essere gestita con strumenti specifici come ruoli RBAC, identità, policy, regole di rete e controlli di accesso.

7. **Quale vantaggio offre Azure CLI rispetto alla sola operazione nel portale?**  
   Azure CLI permette di eseguire controlli e operazioni in modo ripetibile, preciso e facilmente documentabile. Consente inoltre di filtrare gli output, verificare proprietà specifiche e riutilizzare gli stessi comandi in procedure o automazioni. Il portale resta utile per l'esplorazione visiva e come verifica indipendente.

8. **Perché l'esecuzione del comando di eliminazione non dimostra da sola che il cleanup sia concluso?**  
   Un comando come `az group delete --no-wait` avvia l'eliminazione ma non attende necessariamente il completamento. Il ritorno del prompt dimostra soltanto che la richiesta è stata accettata. Per confermare il cleanup bisogna attendere la cancellazione e verificare che il resource group non esista più, ad esempio con:

   ```bash
   az group wait --name "<NOME>" --deleted
   az group exists --name "<NOME>"
   ```

   Il risultato finale atteso è:

   ```text
   false
   ```
