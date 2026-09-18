# UD10 — Consegna laboratorio autonomo

## Baseline

- compose ps: backend "Up (healthy)", frontend "Up", porta 127.0.0.1:8080->80/tcp
- health: GET /health → 200 OK, {"status":"ok","service":"catalog-backend","version":"2.0"}

## Errore

- variabile: LOW_STOCK_THRESHOLD
- valore errato: "not-a-number"
- stato backend: Exited (1) — il container termina subito dopo l'avvio
- log/errore: ValueError in read_int_env (server.py): "LOW_STOCK_THRESHOLD deve essere un intero, ricevuto: 'not-a-number'"

## Diagnosi

- Sintomo: il container backend termina subito dopo l'avvio (Exited (1)); di conseguenza il frontend, che dipende da backend con condition: service_healthy, non riesce a partire ("dependency failed to start").
- Risultato atteso: il backend deve avviarsi, risultare "healthy" e rispondere 200 su /health.
- Evidenza: i log mostrano ValueError sollevato da read_int_env durante l'inizializzazione; docker compose config conferma che LOW_STOCK_THRESHOLD ha valore "not-a-number" nel servizio backend.
- Ipotesi: il crash avviene in fase di inizializzazione, prima che il server sia pronto a rispondere — probabile errore di validazione di una variabile d'ambiente, non un problema di rete, immagine o logica applicativa a runtime.
- Causa: LOW_STOCK_THRESHOLD nel compose.yaml impostata a un valore non numerico; il backend valida questa variabile all'avvio con int(raw), la conversione fallisce e l'eccezione non gestita termina il processo (exit code 1).

## Fix

- modifica minima: ripristinato LOW_STOCK_THRESHOLD: "5" nel compose.yaml
- rebuild necessario?: no
- motivazione: la variabile è passata a runtime tramite Compose, non è "cotta" nell'immagine in fase di build; codice e immagine non sono mai cambiati, quindi è bastato un docker compose up -d per far ripartire il container con la nuova configurazione.

## Test

- backend healthy: sì — Up (healthy)
- health: GET /health → 200 OK
- products: GET /api/products → 4 prodotti restituiti correttamente, con stock_status coerente con la soglia (LOW per stock 4 e 2, OK per stock 8 e 15)
- browser: verificato su http://127.0.0.1:8080/

## Modifica conservata

- APP_ENV: aggiunta APP_ENV: "local-docker" al blocco environment del servizio backend (variabile documentale, non richiesta dal backend per funzionare)
- compose config: confermata presenza di APP_ENV: local-docker nella configurazione risolta
- test: dopo l'aggiunta, backend risultato "healthy" e /health ancora 200 OK

## Git

- branch: fix/ud10-invalid-threshold
- commit: "fix: validate Docker runtime configuration"
- PR: https://github.com/giorgias18/azure-devops-lab/pull/1
- diff verificato: sì, tramite gh pr diff — modifica limitata al file compose.yaml (aggiunta di APP_ENV)
- merge: squash merge completato, branch locale e remoto eliminati

## Passaggio alla verifica

- stack mantenuto attivo: sì