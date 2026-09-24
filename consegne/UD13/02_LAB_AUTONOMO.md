# UD13 — Consegna LAB autonomo
 
- branch: fix/ud13-pipeline-path
- path errato: infra/bicep/delivery-NON-ESISTE.bicep
- stage: Validate IaC
- job: Validate Terraform and Bicep
- step: Bicep lint
- messaggio: `ERROR: An error occurred reading file. Could not find file '/home/giorg/azdo-agent/_work/1/s/infra/bicep/delivery-NON-ESISTE.bicep'.` (Bash exited with code '1')
- classe del problema: errore di filesystem/path del repository (non Azure, non pool/agent, non autenticazione GitHub, non service connection)
- correzione: ripristinato il path corretto `infra/bicep/delivery.bicep` in `pipelines/azure-pipelines-iac.yml` (commit `fix: restore Bicep pipeline path`, 058d729)
- run finale: Validate IaC → succeeded, Deploy delivery infrastructure → succeeded
- PR/merge: PR "Fix/ud13 pipeline path" da fix/ud13-pipeline-path verso main, completata con squash and merge
 