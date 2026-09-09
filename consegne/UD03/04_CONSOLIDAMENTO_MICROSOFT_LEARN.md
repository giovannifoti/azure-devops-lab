# UD03 — Consolidamento Microsoft Learn

## Modulo 1 — Understand Microsoft Entra ID

### 1. Tenant Entra ID e subscription Azure sono la stessa cosa?

No. Il **tenant Microsoft Entra ID** rappresenta il contesto di identità e directory: contiene utenti, gruppi, applicazioni e altri oggetti di identità. La **subscription Azure** è invece un contenitore amministrativo e di fatturazione per le risorse Azure ed è associata a un tenant.

### 2. Qual è il ruolo di Microsoft Entra ID?

Microsoft Entra ID è il servizio cloud di gestione delle identità e degli accessi. Permette di autenticare utenti e applicazioni, gestire utenti e gruppi e fornire identità utilizzabili dai servizi e dalle applicazioni cloud.

### 3. Qual è la differenza generale tra Microsoft Entra ID e Active Directory Domain Services?

Microsoft Entra ID è un servizio cloud di identità e accesso pensato per applicazioni e servizi cloud. Active Directory Domain Services è invece il servizio di directory tradizionale basato su dominio, utilizzato tipicamente in ambienti Windows aziendali per funzionalità come domain join, Group Policy e autenticazione Kerberos/NTLM.

### 4. Perché un'applicazione cloud può utilizzare Microsoft Entra ID?

Perché Microsoft Entra ID può fornire autenticazione e gestione delle identità alle applicazioni cloud, consentendo di riconoscere gli utenti e applicare controlli di accesso senza dover gestire internamente un sistema separato di identità.

---

## Modulo 2 — Create, configure, and manage identities

### Domanda: 12 sistemisti devono ricevere gli stessi permessi su un Resource Group. Quale soluzione è preferibile?

**Risposta: B — Creare un gruppo e assegnare il ruolo al gruppo.**

È preferibile perché l'accesso viene gestito in modo centralizzato: si assegna il ruolo una sola volta al gruppo e si aggiungono o rimuovono gli utenti tramite la membership. Questo riduce il numero di assegnazioni individuali e semplifica manutenzione e audit.

---

## Modulo 3 — Gerarchia di gestione Azure

```text
Management Group
    ↓
Subscription
    ↓
Resource Group
    ↓
Resource
```

Questa gerarchia è importante perché lo scope di Azure RBAC e Azure Policy può essere applicato a livelli diversi e, quando previsto, le impostazioni definite a uno scope superiore possono influire sugli scope figli.

---

## Modulo 5 — Azure RBAC

Una **Role Assignment** è composta da:

```text
Security Principal
        +
Role Definition
        +
Scope
```

### Security Principal

Può essere, ad esempio:

- utente;
- gruppo;
- service principal;
- managed identity.

### Ruoli principali

| Ruolo | Significato generale |
|---|---|
| Reader | Visualizza le risorse |
| Contributor | Gestisce le risorse |
| Owner | Gestisce risorse e accessi |

### Scope

```text
Management Group
    ↓
Subscription
    ↓
Resource Group
    ↓
Resource
```

Le autorizzazioni assegnate a uno scope superiore possono essere ereditate dagli scope inferiori.

---

## Modulo 4 — Azure Policy e Policy Initiatives

### Concetti fondamentali

```text
Policy Definition
    ↓
singola regola
```

```text
Policy Initiative
    ↓
insieme di più Policy Definition
con un obiettivo comune
```

```text
Policy Assignment
    ↓
applicazione della policy o initiative
a uno scope
```

```text
Compliance
    ↓
verifica della conformità
```

### Governance Baseline prevista dallo scenario

```text
Training Governance Baseline
    |
    +-- Allowed locations
    |
    +-- Require Environment=Training
```

Le risorse di formazione devono rispettare sia le regioni consentite sia il tag:

```text
Environment = Training
```

---

## Test di conformità — Previsioni

### Test A — Regione autorizzata, tag mancante

Configurazione:

```text
Name: vnet-test01
Region: regione autorizzata
Tag: assente
```

**Previsione:** la configurazione non è conforme alla regola che richiede `Environment=Training`. Se la Policy Definition applicata usa un effetto di blocco come `Deny`, la creazione viene impedita.

### Test B — Tag corretto, regione non autorizzata

Configurazione:

```text
Name: vnet-test02
Region: regione NON autorizzata
Environment = Training
```

**Previsione:** la configurazione non è conforme alla policy sulle regioni consentite. Se l'effetto applicato è `Deny`, la creazione viene impedita.

### Test C — Configurazione conforme

Configurazione:

```text
Name: vnet-test03
Region: regione autorizzata
Environment = Training
```

**Previsione:** la configurazione è conforme a entrambe le regole e può essere creata.

---

## RBAC vs Azure Policy

Un utente è `Owner` del Resource Group. Una Policy consente solo determinate regioni. L'utente prova a distribuire una risorsa in una regione non consentita.

**Risposta:** no, essere `Owner` non consente automaticamente di ignorare la Policy.

```text
RBAC
CHI può fare COSA
```

```text
POLICY
QUALE CONFIGURAZIONE è consentita
```

RBAC stabilisce le operazioni che un'identità può eseguire sullo scope. Azure Policy valuta invece se la configurazione della risorsa rispetta le regole definite.

---

## Modulo 6 — Self-service password reset

### Che cos'è SSPR?

SSPR, Self-Service Password Reset, permette agli utenti di reimpostare autonomamente la propria password quando i prerequisiti e i metodi di autenticazione richiesti sono configurati.

### Perché riduce il carico amministrativo?

Riduce le richieste al supporto IT per il reset delle password, perché l'utente può completare autonomamente la procedura di recupero.

### Prerequisiti generali

La disponibilità pratica dipende da configurazione del tenant, licenze, autorizzazioni e registrazione dei metodi di autenticazione.

### Autenticazione vs autorizzazione

L'autenticazione stabilisce **chi è** l'utente. L'autorizzazione stabilisce **cosa può fare** quell'identità.

---

# Quiz finale — Stile AZ-104

## 1

Un utente ha `Reader` su un Resource Group ma `Contributor` sulla subscription che contiene il Resource Group.

**Risposta: B — Contributor.**

Il ruolo `Contributor` assegnato sulla subscription viene ereditato dal Resource Group figlio. `Reader` non riduce i privilegi già concessi.

## 2

Un tecnico deve creare e modificare VM, dischi e reti ma non deve assegnare ruoli RBAC.

**Risposta: B — Contributor.**

## 3

Quale componente Azure RBAC identifica chi riceve le autorizzazioni?

**Risposta: C — Security principal.**

## 4

Quale componente Azure RBAC definisce quali operazioni sono consentite?

**Risposta: A — Role definition.**

## 5

L'organizzazione deve impedire la distribuzione di risorse al di fuori delle regioni autorizzate.

**Risposta: B — Azure Policy.**

## 6

Una risorsa critica di produzione non deve essere eliminata accidentalmente.

**Risposta: C — Resource Lock CanNotDelete.**

## 7

Un Resource Group ha un lock `CanNotDelete`.

**Risposta: B — Può essere modificato ma non eliminato.**

## 8

È stato impostato un Budget mensile di 100 euro. Cosa accade normalmente superando 100 euro?

**Risposta: C — Il Budget può generare avvisi, ma non blocca automaticamente la spesa.**

## 9

Un utente deve visualizzare le risorse di un Resource Group senza modificarle.

**Risposta: A — Reader.**

## 10

Quale descrizione distingue correttamente RBAC e Policy?

**Risposta: B — RBAC stabilisce chi può eseguire operazioni; Policy stabilisce quali configurazioni sono consentite.**

---

# Matrice di riepilogo UD03

| Tecnologia | Domanda principale |
|---|---|
| Microsoft Entra ID | Chi è l'identità? |
| Azure RBAC | Cosa può fare quell'identità e su quale scope? |
| Azure Policy | Quali configurazioni sono consentite? |
| Policy Initiative | Come raggruppo più Policy sotto uno stesso obiettivo? |
| Resource Lock | Come proteggo una risorsa da modifiche/eliminazioni? |
| Budget | Quanto sto spendendo e quando devo essere avvisato? |

---

# Autovalutazione finale

1. **Microsoft Entra ID vs AD DS:** Entra ID è il servizio cloud di identità e accesso; AD DS è il servizio directory tradizionale basato su dominio.
2. **Tenant vs subscription:** il tenant gestisce identità e directory; la subscription organizza e fattura risorse Azure.
3. **Utente vs gruppo:** l'utente rappresenta una singola identità; il gruppo raccoglie più identità e semplifica la gestione degli accessi.
4. **Security principal, role definition e scope:** identificano rispettivamente chi riceve l'accesso, quali operazioni può eseguire e dove si applicano.
5. **Reader vs Contributor vs Owner:** Reader visualizza; Contributor gestisce risorse; Owner gestisce risorse e accessi.
6. **RBAC vs Azure Policy:** RBAC controlla chi può fare cosa; Policy controlla quali configurazioni sono consentite.
7. **Policy Definition vs Initiative:** una Definition è una singola regola; una Initiative raggruppa più definizioni con uno stesso obiettivo.
8. **Assignment e compliance:** l'Assignment applica una Policy o Initiative a uno scope; la Compliance misura se le risorse rispettano le regole.
9. **CanNotDelete vs ReadOnly:** CanNotDelete impedisce l'eliminazione; ReadOnly limita anche le modifiche.
10. **Budget vs limite tecnico di spesa:** il budget monitora e può notificare; non è normalmente un tetto automatico.
11. **Funzione generale di SSPR:** permette agli utenti di reimpostare autonomamente la password quando configurazione e prerequisiti lo consentono.
