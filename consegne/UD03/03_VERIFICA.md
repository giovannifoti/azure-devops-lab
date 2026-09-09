# Consegna UD03 — Verifica

## Parte A — Scelta singola

1. **B. L'autenticazione è riuscita, ma manca un'autorizzazione applicabile.**  
   Il fatto che l'utente riesca ad accedere al portale dimostra che l'autenticazione è avvenuta. L'impossibilità di leggere il resource group riguarda invece l'autorizzazione sullo scope.

2. **D. Password.**  
   Una role assignment Azure è formata da principal, role definition e scope. La password non ne fa parte.

3. **C. Reader sul resource group.**  
   `Reader` sul solo resource group richiesto applica il principio del minimo privilegio: consente la consultazione senza permettere modifiche.

4. **B. Ereditato.**  
   Un ruolo assegnato a livello di sottoscrizione si applica normalmente anche agli scope figli, come resource group e risorse, salvo condizioni o meccanismi specifici che ne modifichino l'effetto.

5. **B. Contributor.**  
   `Contributor` può creare e modificare risorse, ma normalmente non dispone dell'azione necessaria per creare role assignment RBAC.

6. **C. Impedisce l'eliminazione finché applicabile.**  
   Un lock `CanNotDelete` impedisce l'eliminazione dello scope protetto, ma non sostituisce RBAC e non impedisce le normali operazioni di lettura.

7. **B. Genera una condizione di notifica, ma non costituisce un tetto automatico.**  
   Un budget di Cost Management permette di monitorare la spesa e attivare notifiche al raggiungimento delle soglie, ma non arresta automaticamente le risorse.

8. **B. Microsoft Entra ID con ruolo appropriato.**  
   La creazione di utenti cloud appartiene alla gestione delle identità Microsoft Entra e richiede un ruolo amministrativo appropriato.

## Parte B — Risposte brevi

9. **Perché assegnare ruoli a un gruppo è spesso preferibile alle assegnazioni individuali?**  
   Assegnare il ruolo a un gruppo semplifica la gestione degli accessi: il ruolo viene definito una sola volta e gli utenti ottengono o perdono l'accesso modificando la membership del gruppo. Questo riduce il numero di assegnazioni individuali, facilita audit e manutenzione e rende più semplice applicare criteri coerenti.

10. **Distingui ruolo Microsoft Entra e ruolo Azure con un esempio per ciascuno.**  
    Un ruolo Microsoft Entra regola attività amministrative sulla directory e sulle identità. Per esempio, `User Administrator` può gestire utenti e gruppi secondo le autorizzazioni previste.  
    Un ruolo Azure RBAC regola invece l'accesso alle risorse Azure. Per esempio, `Reader` su un resource group consente di visualizzare le risorse contenute senza modificarle.

11. **Un utente ha Reader sul resource group e Contributor ereditato dalla sottoscrizione. Qual è l'accesso effettivo e perché?**  
    L'accesso effettivo comprende i privilegi di `Contributor` sul resource group, perché il ruolo ereditato dalla sottoscrizione continua ad applicarsi allo scope figlio. L'assegnazione `Reader` non riduce i privilegi già concessi: le autorizzazioni consentite applicabili normalmente si combinano.

12. **Elenca in ordine almeno quattro controlli per diagnosticare `AuthorizationFailed`.**  
    Un percorso di diagnosi ordinato è:
    1. verificare l'autenticazione e la sottoscrizione corrente con `az account show`;
    2. identificare la risorsa e lo scope su cui l'operazione è stata tentata;
    3. leggere attentamente nell'errore l'azione negata, ad esempio `Microsoft.Authorization/roleAssignments/write`;
    4. controllare le role assignment dirette ed ereditate sullo scope con `az role assignment list --scope <SCOPE> --include-inherited`;
    5. confrontare il ruolo effettivo con l'azione richiesta e verificare da `Check access` nel portale;
    6. applicare il rimedio minimo, facendo eseguire l'operazione a un principal già autorizzato o richiedendo il minimo ruolo necessario, senza tentare escalation.

13. **Spiega la differenza tra tag `deleteAfter`, lock `CanNotDelete` e budget.**  
    `deleteAfter` è un tag, quindi un metadato che documenta una data prevista di cleanup ma non impedisce né esegue automaticamente l'eliminazione.  
    `CanNotDelete` è un management lock che impedisce l'eliminazione dello scope finché il lock rimane applicabile.  
    Un budget è uno strumento di Cost Management che monitora la spesa rispetto a una soglia e può generare notifiche, ma non costituisce normalmente un limite rigido né arresta automaticamente le risorse.

## Parte C — Caso situazionale

14. **Individua almeno due scelte o interpretazioni errate.**  
    La prima scelta errata è assegnare `Contributor` sull'intera sottoscrizione a un tecnico che deve soltanto consultare una VNet: ruolo e scope sono più ampi del necessario.  
    La seconda interpretazione errata è presumere che `Contributor` permetta di assegnare ruoli RBAC: normalmente non include `Microsoft.Authorization/roleAssignments/write`.  
    Una terza interpretazione errata sarebbe attribuire il fallimento del cleanup al ruolo RBAC: l'errore `ScopeLocked` indica invece la presenza di un management lock.

15. **Proponi il ruolo e lo scope iniziali più appropriati.**  
    Se il tecnico deve consultare soltanto quella specifica VNet, la scelta minima è `Reader` applicato direttamente alla VNet. Se invece deve consultare l'intero ambiente di rete contenuto in `rg-network-prod`, `Reader` sul resource group è lo scope più appropriato. In entrambi i casi non è necessario assegnare `Contributor` sull'intera sottoscrizione.

16. **Spiega separatamente perché non riesce ad assegnare il ruolo e perché non riesce a eliminare lo scope.**  
    L'assegnazione di `Reader` al collega fallisce perché `Contributor` permette la gestione delle risorse ma normalmente non consente la creazione di role assignment RBAC; manca quindi l'autorizzazione `Microsoft.Authorization/roleAssignments/write`. Il rimedio minimo è far eseguire l'assegnazione da un principal già autorizzato oppure concedere, tramite il processo previsto, il minimo ruolo necessario per la gestione degli accessi sullo scope corretto.  
    L'eliminazione fallisce invece per una causa distinta: `ScopeLocked` indica che sullo scope è presente un management lock, ad esempio `CanNotDelete`. Per il cleanup bisogna individuare il lock, rimuovere soltanto quello previsto e poi ripetere l'eliminazione.
