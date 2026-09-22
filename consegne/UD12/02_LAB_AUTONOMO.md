# UD12 — Consegna LAB autonomo

## Baseline

- `terraform plan`: eseguito come primo comando, prima di qualsiasi modifica
- stato atteso: nessuna modifica, dato che l'infrastruttura era già stata creata 
- stato osservato: "No changes. Your infrastructure matches the configuration." confermato, lo stato Terraform corrisponde esattamente alle risorse reali (rg-ud12-tf, stud12t90071609)

## Modifica

- tag aggiunto: `Environment = "Training"` (agli altri tag già presenti: Course, UD, ManagedBy)
- plan add: 0
- plan change: 1
- plan destroy: 0
- motivazione change e non recreate: i tag sono un attributo modificabile in-place della risorsa (non fanno parte delle proprietà immutabili come nome, location o SKU), quindi Terraform aggiorna la risorsa esistente invece di distruggerla e ricrearla

## Apply e verifica

- apply: eseguito da piano salvato (`terraform apply "ud12-change.tfplan"`). "Apply complete! Resources: 0 added, 1 changed, 0 destroyed"
- tag verificato con Azure CLI: sì, `az storage account show --query tags` conferma tutti e 4 i tag, incluso `Environment: Training`

## Errore controllato

- riferimento errato: `azurerm_resource_group.training.location` al posto di `azurerm_resource_group.lab.location` (nel blocco `azurerm_storage_account.lab`)
- messaggio `terraform validate`: `Error: Reference to undeclared resource` "A managed resource 'azurerm_resource_group' 'training' has not been declared in the root module."
- causa: il resource group nel codice è dichiarato con il nome logico `lab`, non `training`; il riferimento puntava a una risorsa mai definita
- correzione: sostituito `azurerm_resource_group.training.location` con `azurerm_resource_group.lab.location`
- validate finale: "Success! The configuration is valid."

## Git

- `.gitignore` verificato: sì, contiene `.terraform/`, `*.tfplan`, `*.tfstate`, `*.tfstate.*`
- state non committato: confermato, il `git add` selettivo (.gitignore, infra/bicep, infra/terraform/*.tf, .terraform.lock.hcl) non include alcun file `.tfstate`
- lock file: `.terraform.lock.hcl`
- commit: `feat: add Bicep and Terraform infrastructure baseline` (hash `0abfdd4`, 8 file, 139 inserimenti)
- push/PR: push diretto su `main` riuscito (`b624cde..0abfdd4 main -> main`)

## Cleanup Terraform

- `terraform plan -destroy`: eseguito prima del destroy vero e proprio, per verificare le azioni previste
- risorse previste: `azurerm_resource_group.lab` e `azurerm_storage_account.lab` "Plan: 0 to add, 0 to change, 2 to destroy"
- `terraform destroy`: eseguito con conferma `yes`, "Destroy complete! Resources: 2 destroyed"
- `az group exists`: `false` (rg-ud12-tf non esiste più)
- `terraform state list` finale: output vuoto, nessuna risorsa residua nello stato

## Elementi conservati per UD13–UD15

- Bicep: `infra/bicep/main.bicep` committato e conservato nel repo (il codice resta, solo le risorse Azure sono state distrutte)
- Terraform: tutti i file `.tf` conservati (`main.tf`, `outputs.tf`, `providers.tf`, `variables.tf`, `versions.tf`) più `.terraform.lock.hcl`
- file IaC: l'intera baseline IaC (Bicep + Terraform) resta versionata su GitHub, pronta per essere riapplicata nei laboratori successivi
- agent configurato: non toccato in questo laboratorio, come da istruzioni del punto 12 (non rimosso, non riconfigurato, nessun nuovo PAT creato); resta registrato in `~/azdo-agent`, la verifica finale è rimandata a dopo la verifica individuale di UD12