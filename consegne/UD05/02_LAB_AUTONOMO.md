# Laboratorio autonomo — Diagnosticare una regola incoerente

## Scenario

Il livello web deve poter raggiungere il database sulla porta TCP 5432. La VNet e le subnet risultano configurate correttamente; l'analisi si concentra quindi su NSG, priorità, associazioni e route, senza attribuire il problema a un'applicazione non ancora distribuita.

---

## 1. Inventario della configurazione

**Risposta:**

La rete virtuale è configurata con il seguente piano di indirizzamento:

```text
VNet: vnet-cea-f1baa6
Address space: 10.50.0.0/16

snet-web
- Prefix: 10.50.10.0/24
- NIC: nic-web-01
- Private IP: 10.50.10.4
- Public IP: assente

snet-data
- Prefix: 10.50.20.0/24
- NIC: nic-data-01
- Private IP: 10.50.20.4
- Public IP: assente
- NSG associato: nsg-data-f1baa6
```

La subnet `snet-data` risulta associata all'NSG previsto.

Evidenza anonimizzata:

```json
{
  "NSG": "/subscriptions/<omitted>/resourceGroups/rg-cea-network-f1baa6/providers/Microsoft.Network/networkSecurityGroups/nsg-data-f1baa6",
  "Prefix": "10.50.20.0/24",
  "Subnet": "snet-data"
}
```

Le due NIC risultano nelle subnet corrette:

```text
nic-web-01  -> 10.50.10.4 -> snet-web
nic-data-01 -> 10.50.20.4 -> snet-data
```

**Nota:**

La topologia e le associazioni risultano coerenti con il flusso previsto:

```text
10.50.10.0/24:any -> 10.50.20.0/24:5432 TCP
```

---

## 2. Introduzione del conflitto

È stata aggiunta temporaneamente la regola:

```text
Name: Deny-Web-Postgres-Auto
Priority: 250
Direction: Inbound
Access: Deny
Protocol: TCP
Source: 10.50.10.0/24
Destination: *
Destination port: 5432
```

La configurazione delle regole personalizzate è risultata:

```text
Priority  Name                    Access  Direction  Source          Port
250       Deny-Web-Postgres-Auto  Deny    Inbound    10.50.10.0/24  5432
300       Allow-Web-Postgres      Allow   Inbound    10.50.10.0/24  5432
```

---

## 3. Valutazione delle priorità

**Risposta:**

La prima regola corrispondente è `Deny-Web-Postgres-Auto`, perché negli NSG Azure una priorità numericamente più bassa viene valutata prima.

```text
250 < 300
-> Deny-Web-Postgres-Auto viene valutata prima
-> il traffico corrispondente viene negato
```

Entrambe le regole interessano lo stesso flusso principale: traffico inbound proveniente da `10.50.10.0/24` verso TCP 5432.

**Nota:**

La presenza della regola Allow con priorità 300 non annulla la Deny con priorità 250, perché la valutazione termina alla prima regola corrispondente.

---

## 4. NSG effettivi e route effettive

Sono stati eseguiti i controlli:

```bash
az network nic list-effective-nsg \
  --resource-group "$LAB_RG" \
  --name nic-data-01 \
  --output jsonc
```

Esito:

```text
NicMustBeAttachedToRunningVmToGetEffectiveSecurityGroups
```

È stato inoltre eseguito:

```bash
az network nic show-effective-route-table \
  --resource-group "$LAB_RG" \
  --name nic-data-01 \
  --output table
```

Esito:

```text
NicMustBeAttachedToRunningVmToGetEffectiveRoutes
```

**Risposta:**

Azure non restituisce effective security rules ed effective routes perché `nic-data-01` non è collegata a una macchina virtuale in esecuzione.

**Nota:**

Questa limitazione non impedisce di verificare direttamente:
- il prefisso della subnet;
- l'associazione dell'NSG;
- le regole configurate;
- le rispettive priorità.

---

## 5. Deduzione e test reale

**Risposta:**

L'assenza di una VM impedisce un test completo con IP Flow Verify e impedisce di verificare una connessione TCP reale verso un workload.

Il conflitto tra le regole è però già diagnosticabile dalla configurazione dell'NSG: `Deny-Web-Postgres-Auto` con priorità 250 e `Allow-Web-Postgres` con priorità 300 corrispondono allo stesso traffico, quindi la Deny viene valutata prima.

**Nota:**

```text
Analisi configurazione
-> possibile senza VM

Test end-to-end
-> richiede VM/workload reale
```

Non è quindi corretto dichiarare la connettività applicativa verificata in UD05.

---

## 6. Rimozione della regola autonoma

È stata rimossa esclusivamente la regola temporanea:

```text
Deny-Web-Postgres-Auto
```

Dopo la rimozione, la configurazione personalizzata dell'NSG è risultata:

```text
Priority  Name                Access  Direction  Source          Port
300       Allow-Web-Postgres  Allow   Inbound    10.50.10.0/24  5432
```

**Risposta:**

`Allow-Web-Postgres` è tornata a essere la prima regola personalizzata corrispondente al flusso previsto.

---

## 7. Diagnosi dei sintomi

### `InvalidAddressPrefix`

**Livello:** indirizzamento IP / configurazione CIDR.

**Controllo:** verificare sintassi, rete, prefisso e coerenza con lo spazio indirizzi della VNet o della subnet.

**Correzione minima:** correggere il CIDR errato senza modificare altre componenti della rete.

**Nota:** un prefisso non valido è un problema di configurazione dell'indirizzamento, non di DNS o applicazione.

---

### `SecurityRuleConflict`

**Livello:** NSG / regole di sicurezza.

**Controllo:** verificare regole esistenti, priorità, direzione, protocollo, origine, destinazione e porte.

**Correzione minima:** modificare o rimuovere soltanto la regola in conflitto, oppure assegnare una priorità coerente e disponibile.

**Nota:** le priorità devono essere univoche nello stesso NSG per la stessa direzione.

---

### Un nome DNS non viene risolto

**Livello:** DNS.

**Controllo:** verificare resolver configurato, record DNS e risoluzione nome -> indirizzo IP.

**Correzione minima:** correggere il record o la configurazione DNS interessata.

**Nota:** se il nome non viene risolto, il problema si presenta prima della verifica di routing e NSG verso l'indirizzo di destinazione.

---

### La porta risulta filtrata da un NSG

**Livello:** sicurezza di rete.

**Controllo:** verificare quali NSG sono associati a subnet/NIC e analizzare le regole applicabili in ordine di priorità.

**Correzione minima:** correggere o rimuovere la regola che blocca il traffico, oppure configurare l'Allow necessario con una priorità coerente.

**Nota:**

```text
DNS      -> risolve nome in IP
Routing  -> determina il percorso
NSG      -> consente o nega il traffico
Servizio -> deve essere realmente in ascolto sulla porta
```

---

## Esito

Il conflitto di priorità è stato individuato senza attribuirlo a un'applicazione non ancora distribuita. La regola autonoma è stata rimossa e la configurazione finale mantiene `Allow-Web-Postgres` con priorità 300.

Le verifiche effettive di NSG, route e connettività end-to-end richiedono una NIC collegata a una VM in esecuzione e verranno completate quando sarà disponibile il workload previsto.
