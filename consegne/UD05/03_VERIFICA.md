# Consegna UD05 — Verifica

## Parte A — Scelta singola

Per le domande 1–8 riporta risposta e motivazione.

1. B. Un prefisso più corto (numero più basso) equivale ad una subnet più grande: /24 ha 256 indirizzi, /26 ne ha 64.
2. B. Due VNet con lo stesso spazio 10.0.0.0/16 non possono essere collegate (peering) né avere routing coerente finché gli intervalli si sovrappongono.
3. B. In Azure la priorità numerica più bassa viene valutata per prima (precedenza maggiore), indipendentemente dal nome o dall'ordine di creazione. 
4. A. Un NSG può essere associato sia a una subnet sia a una singola interfaccia di rete (anche a entrambe contemporaneamente).
5. A. Se sia la subnet sia la NIC hanno un NSG, il traffico deve superare le regole di entrambi i livelli per essere permesso.
6. A. Il DNS risolve nomi host/dominio in indirizzi IP; non c'entra con filtro porte, ruoli o routing.
7. B. È lo strumento diagnostico che, data una VM, indica quale regola (allow/deny) determina l'esito per un flusso specifico.
8. B. Un public IP da solo non basta: serve anche un servizio effettivamente in ascolto sulla porta e regole NSG/routing coerenti lungo tutto il percorso.

## Parte B — Risposte brevi

9. **Spiega perché le subnet devono lasciare margine di crescita.**
Le subnet devono lasciare margine di crescita perché il ridimensionamento di una subnet già in uso (con risorse assegnate) è complesso o impossibile senza ricreare le risorse: un prefisso troppo stretto rischia di esaurire gli indirizzi disponibili man mano che si aggiungono NIC, e ampliarlo richiede spesso di eliminare e ricreare la subnet.
10. **Distingui NSG, route e DNS.**
- NSG: filtra il traffico (allow/deny) in base a regole di sicurezza (protocollo, porta, origine/destinazione), a livello di subnet e/o NIC;
- Route: determina il percorso che un pacchetto segue verso la destinazione (route di sistema o User Defined Route), indipendentemente dal fatto che sia permesso o meno;
- DNS: risolve nomi simbolici in indirizzi IP, a monte sia del routing sia del filtro NSG. Un problema DNS impedisce di raggiungere una destinazione anche se rete e regole sono corrette.
11. **Spiega la statefulness di un NSG.**
Un NSG è stateful se: quando una regola inbound permette l'ingresso di un flusso, il traffico di ritorno per quella stessa connessione viene automaticamente consentito, senza bisogno di una regola outbound corrispondente esplicita (e viceversa). Il tracking avviene per connessione (5-tuple), non regola per regola indipendente in ogni direzione.
12. **Perché una baseline NSG sulla subnet può essere più semplice da governare?**
Una baseline NSG a livello di subnet è più semplice da governare perché centralizza le regole comuni in un unico punto che si applica automaticamente a tutte le NIC della subnet, evitando di dover replicare/mantenere le stesse regole su ogni singola interfaccia di rete e riducendo il rischio di configurazioni incoerenti tra risorse dello stesso segmento.
13. **Quali verifiche sono possibili su una NIC senza VM e quale prova manca?**
Su una NIC senza VM associata è possibile verificare la configurazione statica: l'elenco delle regole NSG associate (subnet e/o NIC) ordinate per priorità, l'associazione NSG↔subnet, l'indirizzo IP privato/pubblico assegnato. Non è invece possibile ottenere gli NSG/route effettivi (list-effective-nsg, show-effective-route-table) né eseguire un vero IP Flow Verify, perché questi richiedono una VM in esecuzione: manca quindi la prova che il traffico reale attraversi correttamente lo stack di rete.

## Parte C — Caso situazionale

14. **Spiega perché il cambio di nome non modifica l'esito.**
Il cambio di nome non modifica l'esito perché l'ordine di valutazione delle regole NSG dipende esclusivamente dalla priorità numerica, non dal nome. Deny-Web (priorità 150) ha una priorità numericamente più bassa (quindi precedenza maggiore) rispetto a AAA-Allow-Web (priorità 400, invariata), anche se alfabeticamente "AAA" precede "Deny". Il traffico resta quindi negato.
15. **Proponi una correzione minima senza aprire Internet.**
Si potrebbe modificare la priorità di AAA-Allow-Web assegnandole un valore numerico più basso di 150 (es. 100), lasciando invariati protocollo, porta (443) e origine (10.60.10.0/24), così la regola Allow viene valutata prima della Deny, senza allargare l'origine consentita né esporre la risorsa a */Internet.
16. **Elenca i controlli successivi se, dopo la correzione, l'applicazione resta irraggiungibile.**
Se dopo la correzione l'applicazione resta comunque irraggiungibile, i controlli successivi da fare in ordine sono:
- verificare se esiste anche un NSG a livello di NIC (oltre a quello di subnet) che potrebbe bloccare lo stesso traffico;
- controllare le route effettive (eventuali UDR che deviano il traffico altrove);
- verificare che il servizio applicativo sia effettivamente in ascolto sulla porta 443 sulla VM di destinazione;
- controllare la risoluzione DNS del nome usato per raggiungere la destinazione;
- eseguire un IP Flow Verify reale (richiede VM in esecuzione) per confermare quale regola/livello sta ancora bloccando il flusso.