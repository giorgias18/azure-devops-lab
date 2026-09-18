# UD10 — Risposte alle domande sui concetti

## 1. Differenza fra codice sorgente, image e container
Il codice sorgente (es. server.py) è solo un file di progetto. L'image è l'artefatto immutabile ottenuto combinando un'image base, il codice e le istruzioni di build (Dockerfile). Il container è l'istanza runtime in esecuzione creata a partire da un'image; la stessa image può generare più container.

## 2. Cosa fa e non fa docker build
Docker build costruisce l'image a partire dal Dockerfile e dal build context. Non avvia l'applicazione: non esegue il CMD, si limita a produrre l'artefatto da cui, in seguito, un docker run potrà creare un container.

## 3. Ruolo di Dockerfile, build context e .dockerignore
Il Dockerfile è la "ricetta" che descrive come costruire l'image. Il build context è l'insieme dei file (di solito la directory corrente) che il builder può usare, ad esempio per le istruzioni COPY. Il .dockerignore riduce ciò che entra nel context, escludendo file inutili o sensibili (.git, __pycache__, .env, ecc.), a differenza del .gitignore che riguarda invece cosa Git non versiona.

## 4. Cosa succede se l'image di FROM non è disponibile localmente
Docker verifica se l'image (es. python:3.13-slim) è già presente localmente; se sì la riusa, altrimenti la scarica (pull) da un registry configurato, normalmente Docker Hub.

## 5. Differenza fra RUN e CMD
RUN viene eseguito durante la build, per preparare il filesystem/l'utente dell'image (es. creare appuser, la cartella /runtime, assegnare permessi): le modifiche restano nell'image finale. CMD invece definisce il comando che verrà eseguito quando nasce il container, non durante la build.

## 6. Perché modificare server.py non modifica automaticamente l'image
Perché COPY inserisce una copia del file nel filesystem dell'image al momento della build. Il container esegue quella copia incorporata, non il file presente nella directory Git. Se il sorgente cambia dopo la build, l'image resta quella vecchia finché non si ricostruisce.

## 7. A cosa serve il tag catalog-backend:ud10
È un'etichetta che identifica l'image nel repository locale: catalog-backend è il nome, ud10 è il tag. Serve per riferirsi a quella specifica image; non rappresenta necessariamente una vera versione semantica (nemmeno latest lo è).

## 8. Cosa succede, in ordine, con docker run catalog-backend:ud10
Docker prende l'image catalog-backend:ud10, crea un nuovo container, applica la configurazione runtime (env, publish porte, ecc.) ed esegue il comando definito da CMD (python /app/server.py).

## 9. Differenza fra EXPOSE 8000 e --publish 127.0.0.1:8000:8000
EXPOSE 8000 è solo un'informazione documentale nell'image: dichiara su quale porta l'app ascolta, ma non collega nulla verso l'host. --publish 127.0.0.1:8000:8000 è la pubblicazione reale, effettuata al momento del run, che rende la porta del container raggiungibile dall'host.

## 10. Problema risolto da Docker Compose rispetto a molti docker run manuali
Gestire manualmente più container (reti, volumi, env, ordine di avvio, ecc.) con comandi separati è scomodo e soggetto a errori (parametri dimenticati, porte/reti sbagliate). Compose permette di descrivere in modo dichiarativo lo stato desiderato dell'intero stack in un unico file YAML e di realizzarlo con un solo comando.

## 11. Ruolo del Dockerfile vs compose.yaml
Il Dockerfile descrive come costruire una singola image. Il compose.yaml descrive come far funzionare insieme più servizi/container (reti, volumi, dipendenze, ecc.). Compose non sostituisce il Dockerfile: lo usa per costruire le image dei servizi.

## 12. Cosa crea/gestisce Compose nella UD10
Nel nostro compose.yaml: due servizi (backend e frontend), una rete bridge (catalog-net) e un volume nominato (catalog-runtime), oltre ai relativi container.

## 13. Perché docker compose up -d --build può costruire sia backend sia frontend
Perché entrambi i servizi nel compose.yaml hanno una sezione build: con il proprio Dockerfile (backend.Dockerfile e frontend.Dockerfile). Compose legge i servizi, trova quelli con build:, esegue le rispettive build e poi crea i container.

## 14. Perché il backend non è pubblicato direttamente sull'host nello stack Compose
Perché nell'architettura Compose il backend deve essere raggiunto solo dal frontend tramite la rete interna catalog-net (usa expose, non ports). Il browser entra dal frontend, e Nginx inoltra le richieste /api/* al backend, mantenendo la separazione fra livello web e API.

## 15. Perché Nginx usa http://backend:8000 e non http://localhost:8000
Perché localhost dentro il container frontend indica il frontend stesso, non il backend. backend è invece il nome del servizio Compose, risolvibile tramite il DNS interno della rete Docker (catalog-net), che permette ai container di comunicare senza conoscere gli IP.

## 16. Funzione di catalog-net
È la rete bridge privata che collega backend e frontend, permettendo la comunicazione tra i due tramite il nome del servizio (DNS interno), senza esporre il backend direttamente all'host.

## 17. Funzione di catalog-runtime
È il named volume montato su /runtime nel container backend: garantisce che i dati scritti lì persistano indipendentemente dal ciclo di vita del singolo container (rimuovere/ricreare il container non comporta necessariamente la perdita dei dati).

## 18. Perché running e healthy non sono la stessa cosa
running indica solo che il processo del container esiste ed è in esecuzione. healthy indica che il controllo applicativo definito dall'healthcheck (es. una richiesta a /health) ha avuto effettivamente successo. Sono due livelli di verifica diversi: uno di processo, uno funzionale.

## 19. Cosa fa depends_on: condition: service_healthy
Fa sì che il frontend venga avviato solo dopo che il backend ha raggiunto lo stato healthy (non solo running), rendendo esplicito il rapporto tra ordine di avvio e reale prontezza applicativa del servizio da cui si dipende.

## 20. Differenza fra rebuild e recreate
Rebuild è necessario quando cambia codice o Dockerfile (es. server.py, nginx.conf): bisogna ricostruire l'image (docker compose up -d --build). Recreate riguarda invece i casi in cui cambia solo una configurazione runtime (es. una variabile d'ambiente): la stessa image può essere riusata, ma il container va ricreato con la nuova configurazione, senza bisogno di rifare la build.

## 21. Ordine di diagnosi per uno stack Compose che non risponde
Seguendo il metodo di troubleshooting della UD: 1) docker compose ps, 2) docker compose logs < servizio>, 3) docker compose config, 4) verificare l'health, 5) verificare l'environment, 6) verificare le porte, 7) verificare rete e nomi DNS, 8) verificare mount/volume, 9) decidere se serve recreate o rebuild, 10) fare una modifica minima, 11) ripetere il test. Prima si osserva, poi si formula un'ipotesi, infine si modifica.

## 22. Collegamento fra ciò che facciamo manualmente in UD10 e le pipeline successive
Eseguiremo manualmente docker build, docker run, docker compose up, ecc. per capire davvero cosa succede. Nelle UD successive (UD11 push su Azure Container Registry, UD14 pipeline CI con docker build/push, UD15 pipeline CD) sarà un Agent (self-hosted o Microsoft-hosted) a eseguire automaticamente le stesse operazioni che oggi abbiamo imparato a fare e diagnosticare a mano.