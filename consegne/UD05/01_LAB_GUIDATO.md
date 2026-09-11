# Laboratorio guidato — Reti virtuali, subnet, NSG e NIC

## 1. Contesto e obiettivo

Il laboratorio ha previsto la creazione e verifica di una rete virtuale Azure con due subnet, un Network Security Group applicato alla subnet dati e due interfacce di rete private.

L'obiettivo era verificare configurazione, priorità delle regole NSG e limiti delle verifiche effettive in assenza di una VM.

---

## 2. Piano di indirizzamento

**Risposta:**

È stata creata la seguente topologia:

```text
VNet: vnet-cea-f1baa6
Address space: 10.50.0.0/16

snet-web
└── 10.50.10.0/24

snet-data
└── 10.50.20.0/24
```

Evidenza:

```json
{
  "AddressSpace": [
    "10.50.0.0/16"
  ],
  "Subnets": [
    {
      "Name": "snet-web",
      "Prefix": "10.50.10.0/24"
    },
    {
      "Name": "snet-data",
      "Prefix": "10.50.20.0/24"
    }
  ],
  "VNet": "vnet-cea-f1baa6"
}
```

**Nota:**

Le due subnet appartengono allo spazio della VNet e non si sovrappongono.

---

## 3. NSG e regola applicativa

È stato creato l'NSG:

```text
nsg-data-f1baa6
```

con la regola:

```text
Name: Allow-Web-Postgres
Priority: 300
Direction: Inbound
Access: Allow
Protocol: TCP
Source: 10.50.10.0/24
Destination port: 5432
```

Verifica:

```json
{
  "Access": "Allow",
  "Direction": "Inbound",
  "Port": "5432",
  "Priority": 300,
  "Protocol": "TCP",
  "Source": "10.50.10.0/24"
}
```

**Nota:**

La regola consente il flusso previsto dal livello web verso PostgreSQL sulla porta TCP 5432.

---

## 4. Associazione NSG alla subnet dati

La subnet `snet-data` è stata associata all'NSG.

Evidenza anonimizzata:

```json
{
  "Nsg": "/subscriptions/<omitted>/resourceGroups/rg-cea-network-f1baa6/providers/Microsoft.Network/networkSecurityGroups/nsg-data-f1baa6",
  "Prefix": "10.50.20.0/24"
}
```

**Risposta:**

L'NSG è applicato alla subnet `snet-data`, quindi le sue regole costituiscono una baseline di sicurezza per le NIC presenti nella subnet.

---

## 5. NIC della subnet dati

È stata creata dal portale la NIC:

```text
nic-data-01
```

Configurazione finale:

```json
{
  "PrivateIp": "10.50.20.4",
  "PublicIp": null,
  "Subnet": "/subscriptions/<omitted>/resourceGroups/rg-cea-network-f1baa6/providers/Microsoft.Network/virtualNetworks/vnet-cea-f1baa6/subnets/snet-data"
}
```

**Nota:**

La prima creazione era stata effettuata nella VNet errata; la NIC è stata eliminata e ricreata correttamente in `snet-data`.

---

## 6. NIC della subnet web

È stata creata da CLI la NIC:

```text
nic-web-01
```

Configurazione:

```json
{
  "PrivateIp": "10.50.10.4",
  "PublicIp": null,
  "Subnet": "/subscriptions/<omitted>/resourceGroups/rg-cea-network-f1baa6/providers/Microsoft.Network/virtualNetworks/vnet-cea-f1baa6/subnets/snet-web"
}
```

**Nota:**

Entrambe le NIC sono private e non dispongono di public IP.

---

## 7. Verifica della configurazione di rete

La topologia finale è:

```text
vnet-cea-f1baa6
10.50.0.0/16

├── snet-web
│   ├── 10.50.10.0/24
│   └── nic-web-01
│       └── 10.50.10.4
│
└── snet-data
    ├── 10.50.20.0/24
    ├── nic-data-01
    │   └── 10.50.20.4
    └── nsg-data-f1baa6
        └── Allow-Web-Postgres
            ├── Priority 300
            ├── Source 10.50.10.0/24
            └── TCP 5432
```

Il flusso previsto è:

```text
10.50.10.0/24:any -> 10.50.20.0/24:5432 TCP
```

---

## 8. Effective NSG ed effective routes

È stato eseguito:

```bash
az network nic list-effective-nsg   --resource-group "$LAB_RG"   --name nic-data-01   --output jsonc
```

Esito:

```text
NicMustBeAttachedToRunningVmToGetEffectiveSecurityGroups
```

È stato inoltre eseguito:

```bash
az network nic show-effective-route-table   --resource-group "$LAB_RG"   --name nic-data-01   --output table
```

Esito:

```text
NicMustBeAttachedToRunningVmToGetEffectiveRoutes
```

**Risposta:**

Azure richiede che la NIC sia collegata a una VM in esecuzione per calcolare gli NSG effettivi e le route effettive.

**Nota:**

In UD05 sono state verificate configurazione, associazioni e regole. Non è stata dichiarata connettività end-to-end, perché non è presente una VM/workload su cui eseguire IP Flow Verify o una connessione TCP reale.

---

## 9. Guasto intenzionale sulle priorità NSG

È stata aggiunta temporaneamente la regola:

```text
Name: Deny-Web-Postgres
Priority: 200
Direction: Inbound
Access: Deny
Protocol: TCP
Source: 10.50.10.0/24
Destination port: 5432
```

L'ordine risultava:

```text
Priority  Name                Access  Source          Port
200       Deny-Web-Postgres   Deny    10.50.10.0/24  5432
300       Allow-Web-Postgres  Allow   10.50.10.0/24  5432
```

**Risposta:**

La regola `Deny-Web-Postgres` sarebbe stata valutata prima, perché la priorità 200 è superiore alla priorità 300 in termini di ordine di elaborazione.

```text
200 < 300
-> Deny valutata prima
-> traffico negato
```

La regola temporanea è stata poi eliminata.

Configurazione finale:

```text
Priority  Name                Access  Source          Port
300       Allow-Web-Postgres  Allow   10.50.10.0/24  5432
```

---

## 10. Esito

**Risposta:**

La configurazione di rete e le regole NSG sono state verificate correttamente.

Sono state confermate:
- VNet e subnet con prefissi coerenti;
- NSG associato a `snet-data`;
- regola `Allow-Web-Postgres` con priorità 300;
- NIC private nelle subnet corrette;
- comportamento delle priorità NSG tramite un conflitto temporaneo.

**Nota:**

Le effective security rules, le effective routes e la connettività reale non sono verificabili senza una NIC collegata a una VM in esecuzione. Il test end-to-end sarà possibile quando verrà introdotto il workload previsto.
