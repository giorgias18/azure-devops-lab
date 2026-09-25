# UD15 — Consegna LAB guidato

## Controlli iniziali
- ACR: acrud1315fkclwokw5vatm — verificato
- `LegacyRegistryPermissions`: verificato — admin user disabilitato (AdminUserEnabled: False)
- `sc-azure-ud13-15`: OK, login e operazioni riuscite in tutti gli stage
- `AcrPush`: confermato — assegnato al service principal della pipeline (c63a76c9-...)
- Storage state: sttf15... — presente e raggiungibile
- container state: tfstate — presente, contiene ud15.tfstate
- `Storage Blob Data Contributor`: mancante inizialmente → aggiunto manualmente durante il lab
- managed identity: presente, con AcrPull assegnato (4a677a7b-...)
- `AcrPull`: confermato — assegnato alla managed identity
- Agent: pool-ud09-wsl, agente Linux (WSL2)

## Terraform
- init: OK (locale con -backend=false, e in pipeline con backend-config dinamico)
- validate: OK — "Success! The configuration is valid."
- plan iniziale: OK, mostrato nei log dello stage IaC, non applicato
- apply: OK, eseguito nello stage Deploy
- state remoto: verificato — ud15.tfstate presente nel container tfstate

## Pipeline
- Build ID: 13
- IaC: Succeeded (dopo fix permessi Storage Blob Data Contributor)
- Test: Succeeded
- BuildPush: Succeeded
- Deploy: Succeeded
- Smoke: Succeeded (version = Build ID confermato)

## Deployment
- image tag: catalog-backend:13
- FQDN: catalog-api-ud15.jollyfield-4514129a.italynorth.azurecontainerapps.io
- APP_VERSION: 13
- revision: catalog-api-ud15--h1n9oxh (Active: True, Health: Healthy, Replicas: 1)
- health: {"status": "ok", "service": "catalog-backend", "version": "13"}