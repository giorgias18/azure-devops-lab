# UD11 — Consegna laboratorio autonomo

## Baseline

- status: ok
- version: v2
- target port: 8000

## Errore

- nuovo target port: 9999
- sintomo: endpoint /health irraggiungibile
- HTTP/timeout: HTTP 503 "upstream connect error or disconnect/reset before headers... delayed connect error: Connection refused"

## Evidenze

- running status: Running
- revision: catalog-api-ud11--v2 (Active)
- health: Healthy
- log listen port: 8000 ("Catalog backend v2 listening on http://0.0.0.0:8000")
- ingress target port: 9999

## Diagnosi

- Sintomo: la Container App risponde con HTTP 503 sull'endpoint /health invece del previsto status=ok, version=v2.
- Risultato atteso: {"status": "ok", "service": "catalog-backend", "version": "v2"} con HTTP 200.
- Evidenza: app "Running" e revision "Healthy" (container sano); log applicativi mostrano ascolto su porta 8000; configurazione ingress mostra TargetPort = 9999.
- Ipotesi: il proxy di ingress non riesce a raggiungere il container perché tenta la connessione sulla porta sbagliata.
- Causa: mismatch tra la porta di ascolto del backend (8000) e la target port configurata sull'ingress (9999). Non è un problema di immagine, ACR, managed identity o revision.
- Correzione minima: riportare il target port dell'ingress a 8000, senza modificare nient'altro.

## Verifica

- target port: 8000
- health: ok
- version: v2

## Domande

1. Non era necessario creare una nuova image: l'immagine e il codice del backend erano corretti fin dall'inizio.
2. Non era necessario fare push di v3: il problema non era nel codice applicativo.
3. Il problema era l'ingress (mismatch di target port: 9999 invece di 8000), non ACR né managed identity.
4. L'evidenza chiave è stata il confronto tra i log applicativi ("listening on http://0.0.0.0:8000") e la configurazione ingress (TargetPort = 9999).
5. Modificare più impostazioni contemporaneamente sarebbe stato un errore metodologico perché avrebbe reso impossibile isolare quale modifica avesse effettivamente risolto il problema, oscurando la vera causa.


## Passaggio alla verifica

- target port ripristinato a 8000: sì
- health v2 nuovamente OK: sì
- Resource Group mantenuto disponibile: sì