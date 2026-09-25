# UD14 — Consegna LAB guidato

- ACR: acrud1315fkclwokw5vatm (rg-ud13-15-delivery, SKU Basic, roleAssignmentMode LegacyRegistryPermissions)
- admin user: false
- test locali: 3 test eseguiti con unittest, OK
- Docker build locale: catalog-backend:ud14-local, avviato su 127.0.0.1:18014, /health OK (status ok, service catalog-backend, version ci-v1)
- sc-acr-ud14: creata, tipo Docker Registry (Azure Container Registry)
- WIF: Workload Identity federation, nessun accesso automatico a tutte le pipeline
- YAML: /pipelines/azure-pipelines-ci.yml (UD14_AGENT_MODE=MICROSOFT_HOSTED)
- Stage Test: job PythonTests, Agent.Name=Azure Pipelines 1, Agent.OS=Linux, step "Run Python tests" → 3 test OK
- Stage BuildPush: job DockerBuild, Agent.Name=Azure Pipelines 1, Agent.OS=Linux, step "Build and push to ACR" (Docker@2) → build e push riusciti
- run CI: Success (Test → BuildPush, condition succeeded() rispettata)
- Build ID: 5
- tag ACR: 5 (digest sha256:df876bd0cfaa1b5844310227e8755d1a2c519224636ea6a943e9765302d46b08)