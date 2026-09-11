# Verifica — Reti virtuali e connettività

## Parte A — Scelta singola

### 1. Quale subnet è più grande?

**Risposta corretta: B — `/24`**

**Motivazione:**  
Una rete `/24` contiene più indirizzi di una `/26`: il prefisso più corto lascia più bit disponibili per gli host.

**Nota:**

```text
/24 -> 256 indirizzi totali
/26 -> 64 indirizzi totali
```

---

### 2. Due VNet da collegare hanno entrambe `10.0.0.0/16`. Il problema principale è:

**Risposta corretta: B — sovrapposizione degli indirizzi**

**Motivazione:**  
Due reti con lo stesso spazio di indirizzamento producono ambiguità di routing e non possono essere collegate direttamente come reti non sovrapposte.

---

### 3. Tra regole NSG con priorità 200 e 300 viene valutata prima:

**Risposta corretta: B — 200**

**Motivazione:**  
Negli NSG Azure un numero di priorità più basso indica una priorità più alta.

**Nota:**

```text
200 -> valutata prima
300 -> valutata dopo
```

---

### 4. Un NSG può essere associato a:

**Risposta corretta: A — subnet e NIC**

**Motivazione:**  
Un Network Security Group può essere associato a una subnet, a una NIC o a entrambe.

---

### 5. Se subnet e NIC hanno NSG, il traffico deve essere:

**Risposta corretta: A — consentito da entrambi**

**Motivazione:**  
Quando sono presenti NSG sia sulla subnet sia sulla NIC, il traffico deve superare entrambi i livelli di controllo.

---

### 6. DNS serve principalmente a:

**Risposta corretta: A — tradurre nomi in indirizzi**

**Motivazione:**  
Il DNS risolve un nome in un indirizzo IP, permettendo poi agli altri componenti di rete di instradare e filtrare il traffico.

---

### 7. Quale strumento indica la regola che consente o nega un flusso su una VM?

**Risposta corretta: B — IP Flow Verify**

**Motivazione:**  
IP Flow Verify permette di verificare se un flusso verso o da una VM è consentito o negato e identifica la regola NSG responsabile.

---

### 8. Un public IP senza servizio in ascolto e regole coerenti:

**Risposta corretta: B — non garantisce raggiungibilità**

**Motivazione:**  
Un indirizzo IP pubblico identifica una destinazione, ma la raggiungibilità richiede anche routing corretto, regole di sicurezza coerenti e un servizio realmente in ascolto.

---

## Parte B — Risposte brevi

### 9. Spiega perché le subnet devono lasciare margine di crescita.

**Risposta:**  
Le subnet devono essere dimensionate lasciando spazio per nuove NIC, VM e servizi futuri. Una subnet troppo piccola può esaurire rapidamente gli indirizzi disponibili e costringere a modifiche più complesse dell'architettura.

**Nota:**  
Il dimensionamento deve considerare sia il fabbisogno attuale sia la crescita prevista.

---

### 10. Distingui NSG, route e DNS.

**Risposta:**

```text
DNS   -> traduce un nome in un indirizzo IP
Route -> determina il percorso verso la destinazione
NSG   -> consente o nega il traffico
```

**Nota:**  
I tre componenti svolgono funzioni diverse e un problema in uno di essi può impedire la comunicazione anche se gli altri sono configurati correttamente.

---

### 11. Spiega la statefulness di un NSG.

**Risposta:**  
Gli NSG Azure sono stateful: quando un flusso viene consentito, il traffico di risposta associato alla stessa connessione viene riconosciuto automaticamente e non richiede una regola speculare per il ritorno.

**Nota:**  
La statefulness riguarda le connessioni già autorizzate e non sostituisce la corretta configurazione delle regole iniziali.

---

### 12. Perché una baseline NSG sulla subnet può essere più semplice da governare?

**Risposta:**  
Applicare una baseline NSG alla subnet consente di definire regole comuni per tutte le NIC presenti in quella subnet, riducendo configurazioni duplicate e il rischio di differenze non intenzionali tra singole risorse.

**Nota:**  
Le regole specifiche possono poi essere aggiunte dove necessario, mantenendo separata la baseline comune.

---

### 13. Quali verifiche sono possibili su una NIC senza VM e quale prova manca?

**Risposta:**  
Senza VM è possibile verificare la configurazione della NIC, il suo indirizzo IP, la subnet di appartenenza, l'eventuale public IP, l'associazione dell'NSG alla subnet e le regole configurate.

Non è invece possibile ottenere le effective security rules e le effective routes della NIC, né eseguire un test completo con IP Flow Verify, perché Azure richiede che la NIC sia collegata a una VM in esecuzione.

**Nota:**  
È quindi possibile diagnosticare la configurazione, ma non dimostrare la connettività end-to-end.

---

## Parte C — Caso situazionale

Scenario:

```text
Allow-Web
Priority: 400
Action: Allow
Protocol: TCP
Port: 443
Source: 10.60.10.0/24

Deny-Web
Priority: 150
Action: Deny
Protocol: TCP
Port: 443
Source: 10.60.10.0/24
```

---

### 14. Spiega perché il cambio di nome non modifica l'esito.

**Risposta:**  
Il nome della regola non partecipa alla valutazione del traffico. Azure valuta le regole in base alla priorità e ai criteri di corrispondenza.

`Deny-Web` con priorità 150 viene quindi valutata prima di `AAA-Allow-Web` con priorità 400.

**Nota:**

```text
150 < 400
-> Deny valutata prima
-> traffico negato
```

Il cambio di nome non modifica né la priorità né l'azione della regola.

---

### 15. Proponi una correzione minima senza aprire Internet.

**Risposta:**  
La correzione minima consiste nel modificare o rimuovere la regola `Deny-Web` che confligge con il flusso previsto, oppure assegnare alla regola Allow una priorità numericamente inferiore a 150.

L'origine deve restare limitata a:

```text
10.60.10.0/24
```

e la porta a:

```text
TCP 443
```

**Nota:**  
Non è necessario usare `0.0.0.0/0`: il principio del minimo privilegio richiede di mantenere l'origine limitata alla subnet realmente autorizzata.

---

### 16. Elenca i controlli successivi se, dopo la correzione, l'applicazione resta irraggiungibile.

**Risposta:**  
Dopo aver corretto il conflitto NSG verificherei, nell'ordine:

1. associazione della NIC alla subnet corretta;
2. NSG associati a subnet e NIC e relative regole;
3. effective security rules e IP Flow Verify, se la VM è in esecuzione;
4. effective routes e raggiungibilità della destinazione;
5. risoluzione DNS, se viene utilizzato un nome;
6. presenza del servizio in ascolto su TCP 443;
7. firewall del sistema operativo o altre policy locali.

**Nota:**

```text
DNS
-> routing
-> NSG
-> VM / servizio
```

Una configurazione di rete corretta non garantisce da sola che l'applicazione sia effettivamente disponibile.
