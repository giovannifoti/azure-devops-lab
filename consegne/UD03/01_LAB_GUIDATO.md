# Consegna UD03 — Laboratorio guidato

## Contesto anonimizzato

- sottoscrizione e tenant verificati: sì; la sottoscrizione prevista è risultata attiva con stato `Enabled`. Tenant ID, Subscription ID, Object ID e indirizzi personali non sono riportati nella consegna.
- percorso Entra eseguito: **A**
- resource group temporaneo: `rg-cea-identity-c06273` in `italynorth`

## Identità e assegnazione RBAC

| Principal anonimizzato | Ruolo | Scope | Diretta/ereditata | Motivo |
|---|---|---|---|---|
| Gruppo di sicurezza `grp-cea-readers-c06273` | `Reader` | `/subscriptions/<omitted>/resourceGroups/rg-cea-identity-c06273` | Diretta | Consentire la sola consultazione del resource group applicando il principio del minimo privilegio. |
| Utente amministrativo anonimizzato | `Owner` | `/subscriptions/<omitted>` | Ereditata sul resource group | Ruolo preesistente a livello di sottoscrizione, distinto dall'assegnazione Reader creata dal laboratorio. |

Nel percorso A è stato creato un utente cloud temporaneo con alias non personale `cea-lab-c06273` e successivamente un gruppo di sicurezza `grp-cea-readers-c06273` con membership `Assigned`. L'utente temporaneo è stato aggiunto al gruppo e la presenza del membro è stata verificata dal portale.

La creazione degli oggetti Microsoft Entra ha richiesto che l'account corrente fosse autorizzato a creare utenti e gruppi. Non sono stati richiesti privilegi aggiuntivi e non sono stati modificati ruoli amministrativi per completare il laboratorio.

Sul resource group `rg-cea-identity-c06273` è stata creata dal portale una role assignment `Reader` per il gruppo `grp-cea-readers-c06273`. La verifica tramite `Check access` ha mostrato per il gruppo una sola assegnazione `Reader` con ambito `Questa risorsa`, senza assegnazioni di rifiuto.

La verifica CLI anonimizzata ha restituito:

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

L'output evidenzia che il ruolo `Owner` preesistente è applicato a un utente a livello di sottoscrizione, mentre `Reader` è stato assegnato direttamente al gruppo sul solo resource group del laboratorio. Le due righe non vanno quindi interpretate come se descrivessero lo stesso principal.

## Governance e costi

- tag e significato: il resource group è stato creato con `course=cloud-engineer-academy`, `unit=UD03`, `environment=lab` e `deleteAfter=2026-09-10`. I tag servono a classificare l'ambiente, identificarne l'unità didattica e documentare la data prevista di cleanup; non costituiscono un meccanismo di sicurezza.
- lock e operazione impedita: creato il management lock `lock-cea-delete` con livello `CanNotDelete` e nota `Blocco temporaneo per laboratorio UD03`. Il tentativo controllato di eliminazione del resource group con `az group delete --name "$LAB_RG" --yes` è fallito con errore `ScopeLocked`.
- stato di Cost Analysis: sezione consultata sul resource group; il gruppo non conteneva risorse a consumo, quindi il costo poteva risultare nullo o non ancora disponibile.
- budget creato o limitazione documentata: il budget non è stato creato perché il pulsante di aggiunta risultava disabilitato nello scope del resource group. La limitazione è stata documentata senza modificare ruoli o tentare escalation.
- motivo per cui il budget non blocca la spesa: un budget di Cost Management serve a monitorare la spesa e generare notifiche al raggiungimento delle soglie; non arresta automaticamente le risorse e non costituisce da solo un limite rigido di spesa.

Verifica del lock:

```text
Name             Level         Notes
---------------  ------------  --------------------------------------
lock-cea-delete  CanNotDelete  Blocco temporaneo per laboratorio UD03
```

Tentativo di eliminazione protetto dal lock:

```text
Code: ScopeLocked
Message: l'operazione di eliminazione non può essere eseguita perché lo scope è protetto da un lock.
```

Controllo successivo:

```bash
az group exists --name "$LAB_RG"
```

Risultato:

```text
true
```

Il risultato `true` ha dimostrato che il resource group esisteva ancora e che il lock aveva impedito il cleanup accidentale.

## Cleanup

Il cleanup è stato completato dopo laboratorio autonomo e verifica.

Sono stati rimossi esclusivamente gli oggetti temporanei creati durante il laboratorio:

- role assignment `Reader` del gruppo `grp-cea-readers-c06273`;
- management lock `lock-cea-delete`;
- membership dell'utente temporaneo nel gruppo;
- gruppo `grp-cea-readers-c06273`;
- utente temporaneo `cea-lab-c06273`;
- resource group `rg-cea-identity-c06273`.

Il budget non richiedeva cleanup perché non era stato creato.

Durante il cleanup è stato osservato che il lock `CanNotDelete` impediva anche la rimozione della role assignment nello scope protetto. È stato quindi necessario rimuovere prima il lock e successivamente la role assignment temporanea.

Il resource group è stato eliminato dal portale. Un successivo tentativo CLI:

```bash
az group delete   --name "$LAB_RG"   --yes   --no-wait
```

ha restituito:

```text
Code: ResourceGroupNotFound
Message: Resource group 'rg-cea-identity-c06273' could not be found.
```

Questo errore non indica un cleanup fallito: conferma che il resource group era già stato eliminato dal portale prima dell'esecuzione del comando CLI.

La verifica finale è stata eseguita con:

```bash
az group exists --name "$LAB_RG"
```

Risultato finale:

```text
false
```

Il risultato `false` conferma che il resource group non esiste più e che il cleanup è concluso.

## Rilevanza professionale

L'autenticazione stabilisce chi è l'utente che accede ad Azure, mentre l'autorizzazione RBAC determina quali operazioni quel principal può eseguire e su quale scope. Una role assignment combina principal, ruolo e scope, e i permessi ereditati da livelli superiori devono essere considerati quando si interpreta l'accesso effettivo.

Un management lock opera invece come controllo di governance sulla risorsa: `CanNotDelete` impedisce l'eliminazione anche quando un principal possiede autorizzazioni che normalmente consentirebbero la cancellazione. L'esperienza di cleanup ha inoltre mostrato che il lock può interferire con la rimozione di oggetti sotto lo stesso scope, per cui l'ordine delle operazioni deve essere verificato e adattato al comportamento reale osservato.

Distinguere autenticazione, autorizzazione e governance permette di progettare accessi secondo il minimo privilegio, interpretare correttamente i permessi effettivi e prevenire modifiche o cleanup accidentali.
