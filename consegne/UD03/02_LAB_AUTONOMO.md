# Consegna UD03 — Laboratorio autonomo

## Progettazione dell'accesso minimo

Per il team FinOps utilizzerei come principal il gruppo di sicurezza temporaneo `grp-cea-readers-c06273`, con ruolo `Reader` applicato esclusivamente al resource group `rg-cea-identity-c06273`.

| Principal anonimizzato | Ruolo | Scope | Diretta/ereditata | Motivazione |
|---|---|---|---|---|
| Gruppo `grp-cea-readers-c06273` | `Reader` | `/subscriptions/<omitted>/resourceGroups/rg-cea-identity-c06273` | Diretta | Permette di consultare le risorse del solo ambiente applicativo senza modificarle e applica il principio del minimo privilegio. |
| Utente amministrativo anonimizzato | `Owner` | `/subscriptions/<omitted>` | Ereditata sul resource group | Ruolo preesistente a livello di sottoscrizione, distinto dall'assegnazione Reader creata per il gruppo. |

Non utilizzerei `Contributor` perché consentirebbe modifiche alle risorse, non necessarie per uno scenario di sola consultazione. Non utilizzerei `Owner` perché, oltre a concedere privilegi di gestione delle risorse, permette anche la gestione degli accessi RBAC. Entrambi sarebbero quindi eccessivi rispetto al requisito.

## Ricostruzione delle assegnazioni applicabili

La verifica tramite Azure Portal e Azure CLI ha mostrato una role assignment `Reader` diretta sul resource group per il gruppo temporaneo e una role assignment `Owner` applicata a un utente a livello di sottoscrizione, quindi ereditata dal resource group.

Comando utilizzato:

```bash
az role assignment list   --scope "$RG_SCOPE"   --include-inherited   --query "[].{Role:roleDefinitionName,PrincipalType:principalType,Scope:scope}"   --output jsonc   | sed -E 's#/subscriptions/[^/]+#/subscriptions/<omitted>#g'
```

Output anonimizzato e normalizzato:

```json
[
  {
    "PrincipalType": "User",
    "Role": "Owner",
    "Scope": "/subscriptions/<omitted>"
  },
  {
    "PrincipalType": "Group",
    "Role": "Reader",
    "Scope": "/subscriptions/<omitted>/resourceGroups/rg-cea-identity-c06273"
  }
]
```

L'assegnazione `Reader` è **diretta**, perché lo scope coincide con il resource group del laboratorio. L'assegnazione `Owner` è **ereditata**, perché è definita a livello di sottoscrizione.

Una role assignment `Reader` non riduce eventuali privilegi più ampi già posseduti dallo stesso principal. In Azure RBAC le autorizzazioni consentite normalmente si combinano tra gli scope applicabili. Nel caso osservato, tuttavia, l'output mostra `Owner` su un principal di tipo `User` e `Reader` su un principal di tipo `Group`: non si deve quindi assumere che le due righe rappresentino lo stesso principal.

## Budget e Cost Management

Il budget non è stato creato perché, nello scope del resource group, il pulsante per aggiungerlo risultava disabilitato. La limitazione è stata documentata senza modificare ruoli e senza tentare escalation.

Se il budget fosse stato disponibile, i campi previsti sarebbero stati:

- nome: `budget-cea-c06273`;
- periodo: mensile;
- importo: basso ma realistico per il laboratorio;
- soglia: 80% del costo effettivo;
- destinatario: esclusivamente l'indirizzo dell'utente, se richiesto.

Un budget non costituisce un limite rigido di spesa: serve a monitorare i costi e generare notifiche al superamento di soglie configurate, ma non arresta automaticamente le risorse.

## Verifica del lock e della lettura

Il resource group è stato verificato tramite:

```bash
az group show   --name "$LAB_RG"   --query "{Name:name,Location:location,State:properties.provisioningState,Tags:tags}"   --output jsonc
```

Output:

```json
{
  "Location": "italynorth",
  "Name": "rg-cea-identity-c06273",
  "State": "Succeeded",
  "Tags": {
    "course": "cloud-engineer-academy",
    "deleteAfter": "2026-09-10",
    "environment": "lab",
    "unit": "UD03"
  }
}
```

La lettura con `az group show` è riuscita nonostante il management lock, dimostrando che `CanNotDelete` impedisce l'eliminazione ma non impedisce operazioni di lettura.

Il lock attivo è stato verificato con:

```bash
az lock list   --resource-group "$LAB_RG"   --query "[].{Name:name,Level:level,Notes:notes}"   --output table
```

Output:

```text
Name             Level         Notes
---------------  ------------  --------------------------------------
lock-cea-delete  CanNotDelete  Blocco temporaneo per laboratorio UD03
```

Il tentativo di eliminare il resource group, già eseguito nel laboratorio guidato, è fallito con errore `ScopeLocked`. Il successivo controllo `az group exists --name "$LAB_RG"` ha restituito `true`, confermando che il lock ha impedito l'eliminazione.

## Diagnosi dei casi

### Caso A — `Please run 'az login' to setup account.`

- **Causa probabile:** sessione Azure CLI assente, scaduta o non più valida.
- **Controllo:** eseguire `az account show --output table`.
- **Rimedio minimo:** autenticarsi nuovamente con `az login`, quindi verificare che la sottoscrizione prevista sia attiva prima di proseguire.
- **Osservazione dal laboratorio:** durante UD03 è stato incontrato anche un blocco dovuto ai Security Defaults. È stata configurata l'autenticazione a più fattori e il login è stato ripetuto senza disabilitare le impostazioni di sicurezza del tenant.

### Caso B — `AuthorizationFailed ... Microsoft.Authorization/roleAssignments/write ...`

- **Causa probabile:** autenticazione riuscita, ma autorizzazione RBAC insufficiente per creare una role assignment sullo scope indicato.
- **Controllo:** verificare da `Check access` e da CLI quali ruoli sono applicabili allo scope.
- **Rimedio minimo:** far eseguire l'assegnazione da un principal già autorizzato oppure richiedere il minimo ruolo necessario attraverso il processo amministrativo previsto. Non tentare auto-escalation e non modificare ruoli estranei al laboratorio.

### Caso C — `ScopeLocked: The scope is locked and can't be deleted.`

- **Causa probabile:** presenza di un management lock `CanNotDelete` sul resource group.
- **Controllo:** eseguire `az lock list --resource-group "$LAB_RG" --output table`.
- **Rimedio minimo:** durante il cleanup rimuovere esclusivamente il lock temporaneo creato dal laboratorio e poi ripetere l'eliminazione.

## Distinzione tra autenticazione, autorizzazione e lock

L'**autenticazione** conferma l'identità che sta accedendo ad Azure. L'**autorizzazione** determina quali operazioni quella identità può eseguire, in base alle role assignment RBAC e agli scope applicabili. Il **management lock** è invece un controllo di governance applicato alla risorsa o allo scope e può bloccare specifiche operazioni, come l'eliminazione, anche quando il principal possiede normalmente i permessi per eseguirle.

## Checklist di cleanup — da eseguire soltanto nella sezione finale del laboratorio guidato

1. Eliminare l'eventuale budget temporaneo, se creato.
2. Rimuovere esclusivamente la role assignment `Reader` creata sul resource group per `grp-cea-readers-c06273`.
3. Individuare e rimuovere il lock temporaneo `lock-cea-delete`.
4. Rimuovere `cea-lab-c06273` dal gruppo `grp-cea-readers-c06273`.
5. Eliminare il gruppo `grp-cea-readers-c06273`.
6. Eliminare l'utente temporaneo `cea-lab-c06273`.
7. Eliminare il resource group `rg-cea-identity-c06273`.
8. Attendere il completamento dell'eliminazione.
9. Verificare con:

   ```bash
   az group exists --name "$LAB_RG"
   ```

   Il risultato finale deve essere:

   ```text
   false
   ```

Il cleanup non è stato eseguito durante il laboratorio autonomo, come richiesto dalla traccia.
