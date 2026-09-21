# UD11 — Verifica

## Parte A

1. B. Repository (il nome dell'immagine all'interno del registry; myacr.example è il registry/login server, v2 è il tag)
2. B. Leggerlo con az acr show --query loginServer (il formato < name>.azurecr.io non è sempre garantito, es. in cloud sovrani il suffisso cambia)
3. A. Il nome della risorsa ACR (basta il nome, non serve il login server completo)
4. B. Uno snapshot immutabile di una versione/configurazione dell'app
5. A. Una istanza in esecuzione di una revision
6. A. managed identity
7. B. Pull delle immagini dal registry nel modello RBAC appropriato
8. B. L'app può scalare a zero repliche quando non necessarie

## Parte B

9. **Tag vs digest**
Il tag (es. v2) è un'etichetta mutabile: può essere riassegnata a un'immagine diversa nel tempo. Il digest (es. sha256:abc123...) è un identificatore immutabile calcolato sul contenuto dell'immagine: identifica esattamente e in modo univoco quella singola build, indipendentemente da come viene taggata.
10. **Container Apps Environment, Container App, revision, replica**
-Environment: il confine di rete/isolamento condiviso (Log Analytics, networking) che ospita una o più Container App.
-Container App: la risorsa logica che rappresenta l'applicazione (nome, configurazione generale, ingress).
Revision: uno snapshot immutabile di una specifica versione/configurazione della Container App (immagine, variabili d'ambiente, risorse).
-Replica: un'istanza effettivamente in esecuzione (un pod/container reale) di una data revision, moltiplicabile per lo scaling.
11. **Perché managed identity invece di ACR admin credentials**
Le admin credentials sono una password condivisa a livello di intero registry, difficile da ruotare e da tracciare, e va gestita/salvata manualmente (rischio di leak). La managed identity elimina la necessità di gestire segreti: l'identità è legata alla risorsa Azure, l'accesso è controllato tramite RBAC granulare (es. AcrPull su un singolo registry), è auditabile e revocabile senza impattare altri consumer del registry.
12. **Perché una nuova image produce una nuova revision**
Perché la revision è uno snapshot immutabile della configurazione: cambiare l'immagine (o altri parametri "revision-scope") cambia la configurazione effettiva dell'app, quindi Container Apps crea una nuova revision per preservare la tracciabilità e permettere rollback/blue-green tra versioni.
13. **Cosa deve coincidere tra backend e ingress target port**
La porta su cui il processo applicativo nel container è effettivamente in ascolto (es. 0.0.0.0:8000) deve coincidere con il targetPort configurato sull'ingress, altrimenti il proxy non riesce a instradare il traffico verso il container (come visto nel lab, 8000 vs 9999 dà 503).
14. **Almeno cinque controlli per una Container App non raggiungibile**
-Stato dell'app (runningStatus) e stato/salute della revision attiva.
-Configurazione ingress: external, targetPort, FQDN.
-Log applicativi, per verificare su quale porta il backend è realmente in ascolto e se ci sono errori di avvio.
-Elenco revision: quale è attiva, se ce n'è più di una in conflitto.
-Permessi/identità: se il pull dell'immagine da ACR fallisce (managed identity, ruolo AcrPull).
-Stato del Resource Group/risorse di rete (se pertinente) o eventuali problemi DNS/certificato sul FQDN.
15. **Perché UD11 esegue manualmente build, push e deployment invece di partire da una pipeline**
Per far comprendere concettualmente ogni singolo passaggio del ciclo di vita di un'immagine e di un deployment (build, tag, push su registry, autenticazione, configurazione ingress, troubleshooting) prima di automatizzarlo. Solo capendo bene i singoli step manuali si può poi progettare, leggere e debuggare correttamente una pipeline CI/CD che li automatizza.
16. **Perché con RBAC+ABAC non puoi dare per scontato che AcrPull sia il ruolo corretto**
Con ABAC (Attribute-Based Access Control) abilitato su ACR, gli accessi possono essere ulteriormente condizionati da attributi/condizioni (es. limitati a specifici repository all'interno del registry). Quindi assegnare AcrPull a livello di registry non garantisce automaticamente l'accesso al repository specifico necessario: bisogna verificare se esistono condizioni ABAC che restringono l'ambito del ruolo.
17. **Come il lavoro manuale di UD11 prepara UD14 e UD15**
UD11 fa acquisire familiarità pratica con i concetti chiave (registry, tag/digest, revision, replica, ingress, managed identity, troubleshooting) che nelle unità successive (tipicamente pipeline CI/CD e IaC) vengono automatizzati. Sapere "cosa succede sotto" rende molto più facile capire, configurare e debuggare una pipeline che esegue automaticamente build/push/deploy.
18. **System-assigned vs user-assigned managed identity**
-System-assigned: legata al ciclo di vita della singola risorsa Azure che la usa; viene creata e distrutta insieme alla risorsa; non è condivisibile con altre risorse.
-User-assigned: risorsa Azure indipendente, con ciclo di vita proprio; può essere creata una volta e assegnata a più risorse contemporaneamente, e sopravvive anche se una risorsa che la usa viene eliminata.

## Parte C

19. **Causa più probabile e correzione minima**
Causa: mismatch tra la porta di ascolto del backend (8000, da log) e il targetPort dell'ingress (9000). Correzione minima: aggiornare l'ingress impostando targetPort=8000, senza toccare immagine, revision o identity, che risultano già corretti (revision Healthy, immagine catalog-backend:v2 presente in ACR).
20. **Verifiche post-correzione prima di dichiarare risolto**
- curl -fsS "https://$FQDN/health" | python3 -m json.tool → verificare status=ok e la versione attesa.
- Ricontrollare la configurazione ingress (az containerapp ingress show) per confermare targetPort=8000.
- Verificare che la revision attiva sia ancora Healthy dopo la modifica.
- Controllare i log applicativi per assicurarsi che non ci siano nuovi errori post-modifica.

## Cleanup finale

- resource list verificata: sì (ACR, Log Analytics workspace, Container Apps Environment, Container App catalog-api-ud11)
- Resource Group eliminato: sì (rg-ud11-containers)
- `az group exists` = false: sì
- immagini Docker locali UD11 rimosse: sì (catalog-backend:ud11-v1, ud11-v2 e i tag acr1789983563ud11.azurecr.io/catalog-backend:v1, v2)
- prune globale usato: NO
