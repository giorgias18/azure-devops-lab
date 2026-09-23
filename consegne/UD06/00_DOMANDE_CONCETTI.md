# UD06 — Risposte alle domande sui concetti

## 1. Quali risorse e componenti principali costituiscono una VM Azure?
Una VM è composta da: Image (il sistema operativo/base software da cui parte la macchina), Size (le risorse computazionali assegnate: vCPU, RAM, I/O), OS Disk (il disco da cui la VM si avvia e che contiene il sistema operativo), Data Disk opzionale (disco aggiuntivo per dati/applicazioni, separato dal disco di sistema) e NIC (collega la VM alla rete virtuale, fornendo IP privato, eventuale IP pubblico, subnet e NSG).

## 2. Perché Public IP e raggiungibilità non sono sinonimi?
Perché il Public IP permette solo di indirizzare il traffico verso la risorsa, ma non equivale ad autorizzarlo né garantisce che l'applicazione risponda. Serve che il traffico segua un percorso valido, che l'NSG consenta protocollo/porta/sorgente, che il firewall del sistema operativo non blocchi la connessione, che il servizio sia avviato e in ascolto sulla porta corretta. Basta che uno di questi livelli fallisca perché il servizio resti irraggiungibile.

## 3. Distingui Availability Zone, Availability Set e VM Scale Set.
- Availability Zone: zone fisicamente separate nella stessa regione; distribuendo le istanze tra zone diverse si riduce l'impatto di un guasto limitato a una singola zona;
- Availability Set: distribuisce più VM tra Fault Domain (riduce la dipendenza dagli stessi componenti fisici) e Update Domain (evita che la manutenzione pianificata coinvolga tutte le VM insieme);
- VM Scale Set: gestisce in modo coordinato un insieme di istanze VM, permettendo di aumentarle o ridurle come capacità elastica, anche con autoscaling.

## 4. Qual è la differenza tra scale up e scale out?
Scale up (verticale) significa assegnare più risorse alla stessa istanza (es. da 2 a 4 vCPU); la macchina resta unica ma più potente, con limiti fisici massimi. Scale out (orizzontale) significa aumentare il numero di istanze che collaborano al workload, distribuendo il carico; si presta meglio a elasticità e disponibilità, ma richiede un'applicazione progettata per funzionare su più istanze.

## 5. Come può Azure Monitor Autoscale modificare un VM Scale Set?
Autoscale usa regole basate su metriche (es. CPU), soglie, un intervallo temporale in cui la condizione deve persistere, e un'azione di incremento/riduzione (anche con schedule). L'intervallo temporale evita reazioni a picchi momentanei. I limiti minimum e maximum sono importanti perché il minimo garantisce la capacità di base sempre disponibile, mentre il massimo impedisce una crescita incontrollata, sia per motivi tecnici sia economici (più istanze = più costo).

## 6. Qual è la differenza tra App Service Plan e Web App?
L'App Service Plan definisce l'ambiente di capacità (regione, sistema operativo, tier, capacità, funzionalità) su cui girano le applicazioni. La Web App è l'applicazione ospitata, con la sua configurazione e il deployment. Più Web App possono condividere lo stesso App Service Plan.

## 7. Qual è la differenza concettuale tra Azure Monitor Autoscale e App Service Automatic Scaling?
Sono due modelli distinti: Azure Monitor Autoscale modifica il numero di istanze in base a regole esplicite (metriche, soglie, finestre temporali, schedule), analogamente a quanto avviene per i VM Scale Set; Automatic Scaling è una modalità diversa, più orientata al traffico, che usa parametri specifici della piattaforma come Always ready, Maximum burst, Maximum scale limit, Prewarmed instances.

## 8. Distingui Metrics e Logs/Log Analytics.
Le Metrics sono valori numerici associati al tempo (es. CPU, traffico di rete, richieste), utili per trend, soglie, grafici e alert. I Logs, raccolti in un Log Analytics workspace e interrogabili con KQL, sono record interrogabili utili per ricerca, correlazione e analisi dettagliata — servono a capire il "perché" di un evento, non solo il "quanto".

## 9. Qual è il ruolo di Recovery Services vault, backup policy e recovery point?
Il Recovery Services vault è la risorsa che organizza e gestisce la protezione e i recovery point dei workload. La Backup Policy stabilisce come eseguire la protezione (frequenza, schedule, retention). Il Recovery Point è lo stato recuperabile prodotto dal processo di backup, quello che si sceglie effettivamente per il ripristino. Il flusso è: VM, Azure Backup, vault, policy e recovery point.

## 10. Distingui High Availability, Backup e Disaster Recovery.
- High Availability: risponde a "cosa succede se una singola istanza smette di funzionare?" esempio: un VM Scale Set che sposta il carico su un'altra istanza;
- Backup: risponde a "come recupero dati cancellati o corrotti?" esempio: ripristinare un disco da un recovery point;
- Disaster Recovery: risponde a "cosa succede se l'intero ambiente principale non è più utilizzabile?" esempio: failover verso una regione secondaria con Azure Site Recovery.

## 11. Che cosa indicano RPO e RTO?
RPO (Recovery Point Objective) indica quanta perdita di dati è accettabile, in termini di tempo (es. RPO = 1 ora significa poter perdere al massimo i dati dell'ultima ora). RTO (Recovery Time Objective) indica quanto tempo può passare prima che il servizio torni operativo dopo un'interruzione. Sono diversi perché rispondono a domande differenti: RPO riguarda i dati persi, RTO riguarda il tempo di inattività.

## 12. Perché una VM deallocated può continuare a generare costi?
Perché la deallocazione rilascia solo la capacità compute, ma non elimina le altre risorse associate: dischi, backup, indirizzi IP (secondo configurazione/SKU) e altri servizi collegati possono continuare a esistere e quindi a generare costi. La VM resta come configurazione, pronta per essere riavviata, ma non è una risorsa completamente eliminata.
