# Consegna UD02 — Domande sui concetti

Per ciascuna domanda 1–8 riporta una risposta motivata.

1. Perché una macchina virtuale lascia al cliente più responsabilità operative rispetto ad App Service? 
Con una macchina virtuale (VM) il cliente è responsabile di una parte molto più ampia dell'ambiente: sistema operativo, aggiornamenti, patch, configurazione del sistema, software installato, sicurezza e applicazioni. Con Azure App Service, invece, Azure gestisce gran parte dell'infrastruttura e della piattaforma sottostante, mentre il cliente si concentra principalmente sull'applicazione e sulla sua configurazione. Quindi la VM offre maggiore controllo, ma comporta anche maggiori responsabilità operative.

2. Qual è la differenza tra tenant, sottoscrizione e resource group?
Sono tre livelli diversi:
Tenant: rappresenta l'ambiente di identità Microsoft Entra ID dell'organizzazione e contiene utenti, gruppi e identità;
Sottoscrizione: è il contenitore amministrativo ed economico delle risorse Azure; determina anche il contesto in cui vengono applicati permessi, quote e fatturazione;
Resource group: è un contenitore logico all'interno di una sottoscrizione che raggruppa risorse con un ciclo di vita comune.

3. Perché la località del resource group non obbliga tutte le risorse a usare la stessa region?
La location del resource group identifica principalmente dove vengono memorizzati i metadati del resource group e non determina automaticamente la posizione fisica delle risorse che contiene. Una sottoscrizione può quindi avere risorse distribuite in region diverse, anche se appartengono allo stesso resource group. Questo permette, ad esempio, di raggruppare logicamente risorse distribuite geograficamente.

4. Quale differenza esiste tra una region e un availability zone?
Una region è un'area geografica Azure che contiene uno o più data center, mentre invece una availability zone è invece una separazione fisica indipendente all'interno di una stessa region, progettata per aumentare la resilienza. Quindi, una region rappresenta una posizione geografica, mentre una zone rappresenta una zona fisicamente separata all'interno di quella region.

5. Perché az account show deve precedere la creazione di una risorsa?
Perché permette di verificare in quale contesto Azure CLI è autenticata e quale sottoscrizione è attiva. Prima di creare una risorsa è importante assicurarsi di operare sulla subscription corretta, altrimenti si rischia di creare risorse nell'ambiente sbagliato, con possibili conseguenze sui costi e sulla gestione. È quindi un controllo preventivo che riduce il rischio di errori.

6. Perché i tag non devono essere usati come meccanismo di sicurezza?
I tag sono metadati associati alle risorse e servono principalmente per classificazione, organizzazione, gestione dei costi e automazione, quindi non impediscono a un utente di accedere a una risorsa e non sostituiscono i meccanismi di autorizzazione come RBAC, le policy e i controlli di rete.

7. Quale vantaggio offre Azure CLI rispetto alla sola operazione nel portale? 
Azure CLI permette di trasformare operazioni manuali in comandi ripetibili e automatizzabili. I comandi possono essere salvati in script, riutilizzati, parametrizzati e inseriti in pipeline CI/CD. Il portale è molto utile per l'esplorazione visuale e per comprendere le proprietà delle risorse, mentre la CLI è particolarmente efficace quando servono ripetibilità, automazione, precisione e controlli standardizzati.

8. Perché l'esecuzione del comando di eliminazione non dimostra da sola che il cleanup sia concluso?
Perché il comando di eliminazione può avviare un'operazione asincrona. In particolare, usando --no-wait, Azure CLI restituisce immediatamente il controllo al terminale senza aspettare la conclusione effettiva dell'eliminazione. Per questo bisogna effettuare una verifica successiva.

