# Consegna UD03 — Domande di controllo

1. **Perché autenticazione riuscita e autorizzazione sufficiente non sono equivalenti?**  
   L'autenticazione verifica l'identità dell'utente, cioè conferma chi sta effettuando l'accesso. L'autorizzazione stabilisce invece che cosa quell'identità può fare su una determinata risorsa. Un utente può quindi autenticarsi correttamente in Azure ma non avere il ruolo o i permessi necessari per creare, modificare o eliminare una risorsa.

2. **Quale differenza operativa esiste tra ruolo Microsoft Entra e ruolo Azure?**  
   I ruoli Microsoft Entra regolano attività amministrative relative alla directory e alle identità, ad esempio la gestione di utenti, gruppi e altre impostazioni del tenant. I ruoli Azure RBAC regolano invece l'accesso alle risorse Azure, come sottoscrizioni, resource group, reti virtuali e storage account. Sono quindi due sistemi di autorizzazione distinti, applicati a oggetti e scope differenti.

3. **Quali tre elementi formano una role assignment?**  
   Una role assignment è composta da tre elementi:
   - **principal**, cioè l'identità a cui viene assegnato l'accesso, ad esempio un utente, un gruppo o una managed identity;
   - **role definition**, cioè il ruolo che specifica quali azioni sono consentite, ad esempio `Reader` o `Contributor`;
   - **scope**, cioè il livello sul quale il ruolo viene applicato, ad esempio sottoscrizione, resource group o singola risorsa.

4. **Perché Reader su un resource group è preferibile a Contributor sulla sottoscrizione quando serve soltanto consultare quel progetto?**  
   Perché applica il principio del minimo privilegio. `Reader` permette di consultare le risorse senza modificarle, mentre `Contributor` concede capacità operative molto più ampie. Inoltre assegnare il ruolo sul singolo resource group limita l'accesso al solo progetto interessato, evitando di estendere inutilmente i permessi all'intera sottoscrizione.

5. **Perché un ruolo ereditato non si rimuove dalla risorsa figlia?**  
   Un ruolo ereditato deriva da una role assignment definita a uno scope superiore, ad esempio sulla sottoscrizione o sul resource group. La risorsa figlia riceve quindi quei permessi per ereditarietà, ma non possiede localmente l'assegnazione da rimuovere. Per eliminarla bisogna intervenire nello scope in cui la role assignment è stata originariamente creata.

6. **Un tag `deleteAfter` impedisce l'eliminazione? Motiva.**  
   No. Un tag è soltanto un metadato associato alla risorsa e non applica automaticamente alcun controllo di sicurezza o protezione. `deleteAfter` può documentare una data prevista di cleanup o essere utilizzato da una procedura automatizzata, ma da solo non impedisce né provoca l'eliminazione della risorsa.

7. **Che cosa cambia tra lock `CanNotDelete` e ruolo Reader?**  
   `CanNotDelete` è un management lock applicato a una risorsa o a uno scope e serve a impedire l'eliminazione accidentale, anche a utenti che normalmente avrebbero il permesso di cancellare quella risorsa. `Reader` è invece un ruolo RBAC che concede soltanto autorizzazioni di lettura. Il primo protegge la risorsa dall'eliminazione; il secondo definisce quali operazioni può eseguire un principal.

8. **Perché un budget non è sufficiente a garantire che la spesa non superi una cifra?**  
   Un budget di Azure Cost Management serve principalmente a monitorare la spesa e generare notifiche quando vengono raggiunte determinate soglie. Non costituisce normalmente un limite rigido e non arresta automaticamente le risorse quando viene superato l'importo impostato. Per ottenere azioni automatiche servono procedure, policy o automazioni aggiuntive.
