# UD05 — Domande di controllo

## 1. Perché due VNet da collegare non devono avere CIDR sovrapposti?

**Risposta:**  
Perché il routing deve poter distinguere in modo univoco quale rete contiene l'indirizzo di destinazione. Se due VNet usano intervalli CIDR sovrapposti, le route diventano ambigue e il collegamento, ad esempio tramite peering, non può funzionare correttamente.

**Nota:**

```text
CIDR sovrapposti
→ destinazione ambigua
→ routing non valido
```

---

## 2. Quale rete è più grande, `/24` o `/26`, e perché?

**Risposta:**  
La rete `/24` è più grande della `/26` perché usa meno bit per identificare la parte di rete e lascia più bit disponibili per gli host.

Una `/24` contiene 256 indirizzi totali, mentre una `/26` ne contiene 64.

**Nota:**

```text
prefisso più piccolo
→ rete più grande

/24 > /26
```

---

## 3. Perché un public IP non garantisce raggiungibilità?

**Risposta:**  
Perché il public IP rende disponibile un indirizzo pubblico, ma il traffico deve comunque essere consentito dagli altri controlli di rete. Devono risultare corretti routing, associazioni, NSG e configurazione del servizio in ascolto.

**Nota:**  
Avere un indirizzo pubblico non significa automaticamente che la porta o il servizio siano accessibili.

---

## 4. Come viene scelta una regola NSG tra più corrispondenti?

**Risposta:**  
Le regole NSG vengono valutate in ordine di **priorità crescente**. La regola con numero di priorità più basso viene valutata prima e, quando una regola corrisponde al traffico, la valutazione si interrompe.

**Nota:**

```text
priorità 100
prima di
priorità 200
```

---

## 5. Che cosa significa che un NSG è stateful?

**Risposta:**  
Significa che Azure tiene traccia dello stato di una connessione autorizzata. Se un flusso viene consentito in una direzione, il traffico di risposta della stessa connessione non richiede una regola separata simmetrica.

**Nota:**  
Stateful non significa che ogni nuovo flusso opposto sia automaticamente consentito: vale per il traffico di risposta della connessione già autorizzata.

---

## 6. Perché un `Allow` sulla NIC non supera un `Deny` applicabile sulla subnet?

**Risposta:**  
Perché quando una NIC e la subnet hanno entrambi un NSG, il traffico deve essere consentito da **entrambi** i livelli. Un `Allow` su uno dei due non annulla un `Deny` applicabile sull'altro.

**Nota:**

```text
Subnet NSG: Deny
+
NIC NSG: Allow
=
traffico negato
```

---

## 7. Qual è la differenza tra DNS, routing e NSG?

**Risposta:**  
Il **DNS** traduce nomi in indirizzi IP. Il **routing** decide verso quale destinazione o next hop deve essere inoltrato il traffico. Un **NSG** decide invece se quel traffico è consentito o negato in base a regole di sicurezza.

**Nota:**

```text
DNS     → quale IP?
Routing → quale percorso?
NSG     → consentito o negato?
```

---

## 8. Perché IP Flow Verify verrà completato dopo la creazione della VM?

**Risposta:**  
Perché IP Flow Verify analizza un flusso associato a una specifica interfaccia di rete e richiede quindi una VM/NIC reale su cui valutare il traffico. Prima della creazione della VM manca il contesto necessario per verificare il flusso effettivo.

**Nota:**  
IP Flow Verify serve a capire se un flusso viene consentito o negato e quale regola NSG determina il risultato.
