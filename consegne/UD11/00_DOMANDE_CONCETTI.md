# UD11 — Risposte alle domande sui concetti

# 1. Registry, repository, tag, digest
In myregistry.azurecr.io/catalog-backend:v1: il registry/login server (myregistry.azurecr.io) è il servizio che ospita le immagini; il repository (catalog-backend) raggruppa logicamente le versioni della stessa image; il tag (v1) è un'etichetta leggibile associata a un contenuto, ma riassegnabile; il digest (sha256:...) identifica invece il contenuto in modo content-addressed, quindi immutabile.

# 2. Perché non basarsi solo su latest
latest è solo un tag come un altro: non garantisce che sia la versione più recente o approvata, perché può essere riassegnato a contenuti diversi nel tempo. Usare tag espliciti come v1/v2 permette di rispondere con certezza alla domanda operativa "quale image sta eseguendo questa revision?", cosa che latest non garantisce.

# 3. Perché leggere il login server dalla risorsa
Il nome della risorsa ACR (es. acr12345ud11) e il login server effettivo non coincidono necessariamente uno a uno nella forma usata dalle immagini. Costruirlo manualmente rischia di produrre un valore sbagliato; leggerlo con az acr show --query loginServer garantisce di usare il valore reale restituito da Azure.

# 4. Container Apps Environment vs Container App
L'Environment è un boundary condiviso infrastrutturale in cui possono vivere più Container App (non è l'applicazione stessa). La Container App è invece la descrizione dell'applicazione vera e propria: image, variabili d'ambiente, CPU/memoria, ingress, scaling, identità, modalità revision.

# 5. Revision vs replica
La revision è uno snapshot immutabile della configurazione (image, env var, CPU/memory, scale rule): cambiando questi elementi Azure può crearne una nuova. La replica è invece l'istanza runtime effettivamente in esecuzione di quella revision — una revision può avere più repliche, oppure zero se minReplicas=0.

# 6. A cosa serve ingress
Porta il traffico HTTP/HTTPS dall'esterno (Internet) verso il container, seguendo il percorso: Internet, FQDN HTTPS di Container Apps, Ingress, target port ed infine processo nel container.

# 7. Perché target port deve coincidere con la porta reale
Se il target port non corrisponde alla porta su cui l'applicazione ascolta realmente (es. impostare 9999 invece di 8000), l'ingress inoltra il traffico verso una porta su cui nessun processo è in ascolto: la Container App e la revision possono apparire presenti, ma il traffico non arriva mai all'app.

# 8. Perché managed identity per il pull da ACR
Perché evita di introdurre credenziali statiche (username/password del registry) nel codice o nella configurazione, permettendo invece di usare identità e RBAC nativi di Azure.

# 9. A cosa serve AcrPull
Concede il permesso minimo necessario: leggere (pull) le immagini dal registry. Serve alla Container App per scaricare l'image da eseguire — non deve poter pubblicare (AcrPush non è necessario, perché il push lo fa l'operatore umano durante il lab).

# 10. Cosa succede aggiornando l'image di una Container App
Viene creata una nuova revision (nuova configurazione immutabile associata alla nuova image); nel flusso del laboratorio, la revision precedente (v1) viene sostituita dalla nuova (v2) come revision attiva.

# 11. Perché in UD11 si lavora manualmente
Perché prima di automatizzare un processo bisogna capire quali operazioni si stanno automatizzando: l'automazione non significa saltare la comprensione del processo manuale, ma codificare e rendere ripetibile un processo già compreso. UD11 rende visibili ed esplicite tutte le responsabilità (chi fa push, chi fa pull, quale identità, quale porta) che in UD14–UD15 verranno delegate a una pipeline/Agent.

# 12. RBAC Registry Permissions vs RBAC+ABAC
Nel modello RBAC Registry Permissions si usano i ruoli legacy AcrPull/AcrPush/AcrDelete per l'accesso al registry. Nel modello RBAC + ABAC Repository Permissions, invece, questi ruoli legacy non vengono usati per l'accesso ai repository: si usano ruoli più granulari come Container Registry Repository Reader/Writer. Nel lab si sceglie deliberatamente RBAC Registry Permissions con AcrPull per concentrarsi sui concetti base, non perché RBAC+ABAC sia "sbagliato".

# 13. Perché il self-hosted Agent di UD09 non serve in UD11
In UD11 i comandi vengono eseguiti manualmente nel terminale WSL2 del partecipante, non tramite pipeline: quindi l'Agent non è necessario per svolgere il lab. È comunque collegato alla progressione del corso perché lo stesso ambiente WSL2/Docker sarà poi riutilizzato dall'Agent quando queste operazioni entreranno nelle pipeline (UD14–UD15).

# 14. System-assigned vs user-assigned managed identity
La system-assigned ha un lifecycle legato alla risorsa stessa: viene creata e rimossa insieme alla Container App. La user-assigned è invece una risorsa identità autonoma, che può essere preparata e riutilizzata separatamente da più risorse. In UD11 si usa la system-assigned per rendere visibile il legame diretto risorsa↔identità; nella pipeline finale si userà anche la user-assigned.

# 15. Scale-to-zero
Con minReplicas=0, la Container App può arrivare ad avere zero repliche quando non deve servire traffico, riducendo il compute inutilizzato. Non significa però che l'intero ambiente Azure sia gratuito: ACR e le altre risorse continuano a esistere, quindi per un lab temporaneo il cleanup reale è eliminare il Resource Group.

# 16. Controlli in caso di errore del FQDN dopo una nuova revision
Seguire il percorso della richiesta passo per passo, senza cambiare subito l'image:
FQDN, chiedersi se l'ingress esterno sia abilitato, se il target port è corretto, revision attiva/healthy, replica presente, log applicativi (az containerapp logs show), image/tag presente in ACR, managed identity/AcrPull configurati correttamente, environment variables corrette. Questa sequenza riduce le modifiche casuali e isola il punto di guasto.