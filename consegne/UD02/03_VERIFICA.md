# Consegna UD02 — Verifica

## Parte A — Scelte operative

Per le domande 1–8 riporta risposta e motivazione.

1. **B. IaaS.**  
   È il modello più coerente quando serve controllo completo del sistema operativo e la possibilità di installare componenti non supportati da un servizio gestito. Con IaaS il cliente gestisce sistema operativo, configurazione e software della macchina virtuale, mentre il provider gestisce l'infrastruttura fisica sottostante.

2. **C. Configurazione dell'applicazione, identità, accessi e dati.**  
   In Azure App Service il provider gestisce infrastruttura, sistema operativo e piattaforma sottostante, mentre il cliente rimane responsabile dell'applicazione, delle configurazioni, delle identità, degli accessi e dei dati.

3. **C. È un contenitore logico per risorse che possono condividere ciclo di vita e governance.**  
   Un resource group permette di organizzare risorse correlate e di gestirle come un insieme dal punto di vista operativo, autorizzativo e del cleanup.

4. **B. La località indica dove Azure conserva i metadati del resource group; le risorse possono avere località proprie.**  
   La region del resource group non obbliga tutte le risorse contenute a essere nella stessa region.

5. **B. `az account show`.**  
   Prima di creare risorse è importante verificare la sottoscrizione attiva, lo stato e il contesto corrente, così da evitare di operare nella sottoscrizione sbagliata. Un controllo utile è:

   ```bash
   az account show      --query "{Name:name,State:state,IsDefault:isDefault}"      --output table
   ```

   Il risultato atteso deve mostrare la sottoscrizione prevista, `State` uguale a `Enabled` e `IsDefault` uguale a `True`.

6. **C. `stcea02a7f9`.**  
   Il nome è compatibile perché contiene soltanto lettere minuscole e numeri, senza spazi, trattini, underscore o lettere maiuscole.

7. **B. Il tag documenta l'intenzione, ma serve ancora una procedura o una policy che esegua l'eliminazione.**  
   Il tag `deleteAfter` non esegue automaticamente alcuna azione: rappresenta un'informazione utile per governance, automazioni o procedure di cleanup.

8. **C. `az group exists --name <NOME>` restituisce `false`.**  
   Il ritorno del prompt dopo `--no-wait` indica soltanto che la richiesta di eliminazione è stata avviata. Per dimostrare che il cleanup è concluso si può usare:

   ```bash
   az group wait --name "<NOME>" --deleted
   az group exists --name "<NOME>"
   ```

   Il risultato finale atteso è:

   ```text
   false
   ```

## Parte B — Risposte brevi

9. **Cloud pubblico, privato e ibrido.**  
   Una stessa azienda potrebbe eseguire un portale web e servizi scalabili su Azure nel cloud pubblico, mantenere un sistema legacy con dati particolarmente sensibili nel proprio datacenter come cloud privato e collegare i due ambienti tramite rete e identità comuni. In questo caso l'insieme costituisce un modello ibrido, perché combina risorse pubbliche e private che cooperano.

10. **Relazione fra tenant Microsoft Entra, sottoscrizione, resource group e risorsa.**  
    Il tenant Microsoft Entra rappresenta il contesto di identità e directory. Una sottoscrizione Azure è associata a un tenant e costituisce un confine amministrativo e di fatturazione per le risorse. All'interno della sottoscrizione i resource group organizzano logicamente risorse correlate. Le singole risorse, come VNet o storage account, vengono quindi create nella sottoscrizione e normalmente appartengono a un resource group.

11. **Availability zone e region non sono sinonimi.**  
    Una region è un'area geografica Azure che può contenere una o più availability zone. Le availability zone sono sedi fisicamente separate all'interno della stessa region, progettate per ridurre il rischio che un singolo guasto infrastrutturale coinvolga tutte le copie di un servizio.

12. **Differenza operativa tra Azure Portal e Azure CLI.**  
    Il Portale è più immediato per esplorare servizi, proprietà e opzioni tramite interfaccia grafica. La CLI è più efficace per verifiche ripetibili e precise, perché consente di interrogare direttamente proprietà specifiche e di ottenere output filtrati. Nel laboratorio il Portale è stato utile per la seconda verifica visiva, mentre la CLI ha permesso di controllare stato di provisioning, indirizzamento, SKU, TLS, tag e inventario.

13. **Perché applicare i tag anche alle risorse.**  
    I tag presenti sul resource group non vengono ereditati automaticamente dalle risorse contenute. Per poter filtrare, inventariare, governare o automatizzare operazioni direttamente sulle singole risorse, i tag devono quindi essere applicati esplicitamente anche a esse.

## Parte C — Interpretazione tecnica

1. Nell'ID anonimizzato:

   ```text
   /subscriptions/<omitted>/resourceGroups/rg-cea-test/providers/Microsoft.Storage/storageAccounts/stceatest01
   ```

   sono riconoscibili:
   - sottoscrizione: presente ma anonimizzata come `<omitted>`;
   - resource group: `rg-cea-test`;
   - provider: `Microsoft.Storage`;
   - tipo di risorsa: `storageAccounts`;
   - nome della risorsa: `stceatest01`.

2. La risorsa si trova in `italynorth` e possiede i tag:

   ```text
   environment = lab
   unit = UD02
   ```

3. Per verificare che la risorsa appartenga realmente al resource group userei:

   ```bash
   az resource list      --resource-group "rg-cea-test"      --query "[?name=='stceatest01'].{Name:name,Type:type,Location:location}"      --output table
   ```

   Il risultato atteso deve mostrare `stceatest01`, tipo `Microsoft.Storage/storageAccounts` e località `italynorth`. In alternativa è possibile interrogare direttamente lo storage account indicando il resource group.

4. Se il resource group contiene soltanto risorse del laboratorio, rimuoverei l'intero ambiente eliminando direttamente il resource group:

   ```bash
   az group delete      --name "rg-cea-test"      --yes      --no-wait
   ```

   In questo modo il resource group viene usato come confine del ciclo di vita dell'ambiente.

5. Dopo l'eliminazione attenderei il completamento e verificherei che il gruppo non esista più:

   ```bash
   az group wait      --name "rg-cea-test"      --deleted

   az group exists      --name "rg-cea-test"
   ```

   Il risultato finale atteso è:

   ```text
   false
   ```
