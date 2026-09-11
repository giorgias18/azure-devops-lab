# Consegna UD05 — Domande sui concetti

Per ciascuna domanda 1–8 riporta una risposta motivata.

1. **Perché due VNet da collegare non devono avere CIDR sovrapposti?**
Se collegate (peering o VPN) due VNet i cui spazi di indirizzi si sovrappongono, il routing diventa ambiguo: un pacchetto destinato a un certo indirizzo potrebbe corrispondere a risorse in entrambe le reti, e Azure non saprebbe quale percorso scegliere. L'unicità dei prefissi è quindi un prerequisito per una connettività funzionante.
2. **Quale rete è più grande, /24 o /26, e perché?**
/24 è più grande. Il prefisso indica quanti bit sono fissi per la rete: più basso è il numero, più bit restano liberi per gli host; quindi, un /24 lascia 8 bit liberi (256 indirizzi), un /26 ne lascia solo 6 (64 indirizzi). 
3. **Perché un public IP non garantisce raggiungibilità?**
Un IP pubblico crea solo la possibilità di un endpoint esposto su Internet; affinché sia davvero raggiungibile serve anche che:
- la risorsa associata sia in ascolto su quella porta;
- l'NSG permetta il traffico in entrata;
- il sistema operativo/applicazione non blocchi la connessione e le route siano corrette. 
4. **Come viene scelta una regola NSG tra più corrispondenti?** 
Le regole NSG hanno una priorità numerica da 100 a 4096: viene valutata per prima quella con numero più basso, e alla prima corrispondenza la valutazione si ferma (le regole successive non vengono più considerate). Le regole di default hanno priorità più bassa (numeri alti) e possono essere superate da regole personalizzate con priorità più alta (numero più basso).
5. **Che cosa significa che un NSG è stateful?**
Significa che se un flusso viene permesso in una direzione, il traffico di risposta appartenente a quello stesso flusso viene automaticamente consentito, senza bisogno di una regola speculare nella direzione opposta. Questo vale solo per il traffico di risposta dello stesso flusso, non autorizza una nuova connessione indipendente iniziata dall'altro lato.
6. **Perché un Allow sulla NIC non supera un Deny applicabile sulla subnet?**
Quando un NSG è associato sia alla subnet sia alla NIC, il traffico deve essere permesso da entrambi i livelli: sono filtri concorrenti, non gerarchici. Un Allow a livello NIC non "vince" su un Deny a livello subnet (e viceversa), basta un blocco su uno dei due livelli perché il traffico venga negato.
7. **Qual è la differenza tra DNS, routing e NSG?**
Sono tre funzioni distinte nella catena di connettività:
- DNS: traduce un nome in un indirizzo IP senza autorizzare nulla, si limita alla risoluzione;
- Routing: determina il percorso (next hop) che il traffico deve seguire per raggiungere una destinazione;
- NSG: decide se il traffico, una volta instradato, è permesso o negato in base a origine, destinazione, porta e protocollo.
8. **Perché IP Flow Verify verrà completato dopo la creazione della VM?**
Perché è uno strumento di Network Watcher che verifica se un flusso di traffico specifico è permesso o negato per una NIC reale: richiede quindi che esista effettivamente una VM (con la sua NIC) su cui eseguire il test. 

