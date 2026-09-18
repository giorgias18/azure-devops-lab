# UD10 — Verifica

## Parte A

1. B. Un'immagine è un artefatto (template) usato per creare container, non un processo in esecuzione.
2. B. il punto finale in docker build -f ... . indica il build context (la cartella da cui il Dockerfile può leggere i file).
3. B. EXPOSE documenta la porta prevista dall'immagine, ma non pubblica nulla sull'host: serve -p/ports: per quello.
4. B. host:container, quindi 127.0.0.1:8080 (host) → 80 (container).
5. B. In una rete Compose, i servizi si raggiungono tramite il nome del servizio (DNS interno).
6. B. Dentro il container frontend, localhost è il container stesso, non l'host né altri servizi.
7. B. down -v rimuove anche i volumi named del progetto (oltre a container e rete).
8. B. Un container può essere running ma fallire l'healthcheck, quindi unhealthy.

## Parte B

9. **Image, container, registry:**
L'image è un artefatto immutabile (filesystem + metadati) che definisce come creare un'istanza. Il container è un'istanza in esecuzione (o pronta per esserlo) di un'immagine, con il proprio stato runtime. Il registry è il servizio dove le immagini vengono archiviate e distribuite (es. Docker Hub, ACR).
10. **Perché limitare il build context:**
Il build context viene interamente inviato al daemon Docker durante la build. Un context troppo ampio rallenta la build, rischia di includere file non necessari o sensibili (segreti, .git, node_modules) e va sempre filtrato con .dockerignore.
11. **Named volume vs bind mount:**
Il named volume è gestito da Docker stesso (posizione interna, portabile, ideale per dati persistenti dell'applicazione). Il bind mount collega un percorso specifico del filesystem host al container: utile per sviluppo (es. live-reload del codice), ma dipende dal path locale e non è portabile.
12. **Variabili d'ambiente e riuso dell'immagine:**
Le variabili d'ambiente permettono di configurare il comportamento del container a runtime senza modificare o ricostruire l'immagine: la stessa immagine può girare in ambienti diversi (dev, test, prod) semplicemente passando valori diversi.
13. **Perché il backend non deve pubblicare una porta sull'host:**
Perché comunica con altri servizi (es. il frontend) tramite la rete interna di Compose, usando il nome del servizio. Pubblicare la porta sull'host serve solo se un client esterno al progetto Compose deve raggiungerlo direttamente.
14. **Comandi per iniziare il troubleshooting di uno stack che non risponde:**

docker compose ps
docker compose logs --tail 50 < servizio>
docker compose config

Prima verifichi lo stato dei container, poi i log per l'errore, poi la configurazione risolta per escludere problemi di variabili/valori.

## Parte C

15. **Causa e modifica minima:**
La causa è la stessa vista nel lab: LOW_STOCK_THRESHOLD ha un valore non numerico ('abc'), il backend fallisce la validazione all'avvio e va in crash-loop (Restarting), il frontend riceve 502 perché il backend non è mai healthy. La modifica minima è correggere il valore della variabile nel compose.yaml (es. tornare a "5").
16. **Serve rebuild? Verifiche dopo il fix:**
No, non serve rebuild: è una variabile d'ambiente runtime, non parte dell'immagine. Dopo il fix: docker compose up -d, poi docker compose ps (verificare healthy), poi curl -i http://127.0.0.1:8080/health e test dell'endpoint applicativo (es. /api/products).
17. **Una futura pipeline gira su pool-ud09-wsl e lo step docker build fallisce con errore di connessione al Docker daemon. Quale componente dell'ambiente controlleresti per primo e perché?**
Il primo componente da controllare è il Docker daemon/servizio Docker sull'agente (verificare che sia avviato e raggiungibile, es. docker info o lo stato del servizio Docker su WSL), essendo la causa più comune quando la build non riesce nemmeno a partire per un errore di connessione, prima di guardare Dockerfile o rete.
18. **Distingui, rispetto alla disponibilità dei tool, un self-hosted Agent da un Microsoft-hosted Agent.**
Il self-hosted agent ha a disposizione esattamente i tool che tu installi e mantieni sulla macchina (più controllo, ma responsabilità di manutenzione tua). Il Microsoft-hosted agent è una VM effimera preconfigurata con un set di tool standard già installati da Microsoft, ricreata ad ogni run, comoda ma meno personalizzabile e priva di persistenza tra le esecuzioni.

## Cleanup finale

- `docker compose down`:
- volume presente dopo `down`:
- `docker compose down -v`:
- volume rimosso:
- immagini UD10 rimosse:
- prune globale usato: NO
