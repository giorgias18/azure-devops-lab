# UD11 — Consegna laboratorio guidato

## Preflight

- Azure: login effettuato, subscription "Azure subscription 1" selezionata
- Docker: Docker Desktop 4.91.0, engine 29.8.0, funzionante da WSL2
- containerapp extension: installata, versione 1.3.0b5 (preview)
- Microsoft.App: Registered
- Microsoft.OperationalInsights: Registered

## ACR

- nome: acr1789983563ud11
- SKU: Basic
- login server: acr1789983563ud11.azurecr.io
- repository: catalog-backend
- tag v1: v1
- tag v2: v2
- admin user abilitato: NO

## v1

- local test: /health status: ok, version: v1
- push: eseguito su ACR (docker push)
- ACR tag: v1 presente e verificato
- ACA environment: acaenv-ud11 (North Europe)
- Container App: catalog-api-ud11
- managed identity: SystemAssigned
- registry identity: system (nessuna password/username, autenticazione via managed identity)
- ingress: external
- target port: 8000
- FQDN: catalog-api-ud11.purplesmoke-843b79b8.northeurope.azurecontainerapps.io
- health: status: ok, version: v1
- version: v1
- revision: catalog-api-ud11--v1 (Active: True, Health: Healthy)
- logs: "Catalog backend v1 listening on http://0.0.0.0:8000 threshold=5"; richieste /health e /api/products con esito 200

## v2

- Dockerfile version: APP_VERSION modificata da v1 a v2
- local test: /health status: ok, version: v2
- push: eseguito su ACR (docker push)
- ACR tag: v2 presente insieme a v1
- update: az containerapp update con nuova image v2 e revision-suffix v2
- revision: catalog-api-ud11--v2 (Active: True, Health: Healthy) sostituisce v1 in modalità Single
- health: status: ok, version: v2
- version: v2 (confermato anche su /api/products, count: 4)

## Scaling

- minReplicas: 0
- maxReplicas: 1
- scale-to-zero possibile: sì (configurazione verificata, non testata attivamente per non attendere il timer di cooldown)