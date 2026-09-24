# UD13 — Consegna LAB guidato

## Terraform
- network.tf: copiato da modelli corso, VNet vnet-ud13 (10.13.0.0/16) e subnet snet-app (10.13.1.0/24), dipendenze da Resource Group/VNet ricavate da Terraform tramite riferimenti impliciti
- plan: 4 to add, 0 to change, 0 to destroy (Resource Group, Storage Account, VNet, Subnet)
- apply: completato con successo su regione italynorth (regione westeurope inizialmente non disponibile: errore 403 RequestDisallowedByAzure "region currently not accepting new customers")
- VNet verificata: vnet-ud13, address space 10.13.0.0/16, confermata via `az network vnet show`
- subnet verificata: snet-app, prefix 10.13.1.0/24, confermata via `az network vnet subnet show` (campo `addressPrefixes`, array)
- destroy: eseguito con `terraform destroy` dopo verifica del piano (4 to destroy, solo risorse del laboratorio)
- RG Terraform eliminato: sì, confermato con `az group exists --name rg-ud12-tf` false

## Delivery persistente
- Resource Group: rg-ud13-15-delivery (regione italynorth, tag Course=AZ104 Purpose=FinalDelivery)
- service connection: sc-azure-ud13-15
- WIF: Workload Identity Federation, creazione automatica (App registration automatic), account con ruolo Owner sulla subscription
- scope: Subscription + Resource group rg-ud13-15-delivery
- accesso globale a tutte le pipeline: NO

## Pipeline
- YAML: /pipelines/azure-pipelines-iac.yml, trigger: none, variabile location corretta da westeurope a italynorth per coerenza con il Resource Group
- agent pool: pool-ud09-wsl (self-hosted, WSL2), verificato Online prima dell'esecuzione
- Validate: stage "Validate IaC" completato con successo (job ValidateIaC: terraform fmt -check, terraform init -backend=false, terraform validate, az bicep lint) — ~1m16s
- Deploy: stage "Deploy delivery infrastructure" completato con successo (job DeployIaC, dependsOn Validate, condition succeeded())
- What-If: eseguito via AzureCLI@2 con service connection sc-azure-ud13-15 prima del deployment reale
- deployment: az deployment group create eseguito con successo, nome ud13-delivery-< Build.BuildId>
- ACR: acrud1315fkclwokw5vatm, login server acrud1315fkclwokw5vatm.azurecr.io, roleAssignmentMode LegacyRegistryPermissions
- SKU: Basic
- admin user: disabilitato (adminUserEnabled: false)

## Fine UD13
- risorse delivery conservate: sì (rg-ud13-15-delivery, ACR, sc-azure-ud13-15, pipeline IaC, agent pool-ud09-wsl nessun cleanup)