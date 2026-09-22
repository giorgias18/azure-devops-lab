# UD12 — Consegna LAB guidato

## Preparazione

- materiali UD12: trovati in ~/workspace/corso-azure-devops/UD12/partecipanti
- repository personale: ~/workspace/azure-devops-lab
- directory `infra/bicep`: creata, contiene main.bicep
- directory `infra/terraform`: creata, contiene main.tf, outputs.tf, providers.tf, variables.tf, versions.tf
- directory consegne: ~/workspace/azure-devops-lab/consegne/UD12

## Azure

- subscription verificata: Azure subscription 1
- identità verificata: sì

## Bicep

- Bicep version: 0.47.16 (installata durante il lab, non presente in precedenza)
- lint: nessun errore
- Resource Group: rg-ud12-bicep (westeurope) | nota: Il Resource Group Bicep in westeurope ha dato errore `RequestDisallowedByAzure` (regione non accetta nuovi clienti su questa subscription); è stato risolto passando a northeurope.
- Storage Account: stud12b90070266
- What-If change type: Create (1 to create) — Microsoft.Storage/storageAccounts
- deployment: completato con successo (ud12-bicep-deploy)
- output `storageAccountName`: stud12b90070266
- output `blobEndpoint`: https://stud12b90070266.blob.core.windows.net/
- verifica CLI: confermata — SKU Standard_LRS, TLS1_2, PublicBlob False
- Resource Group Bicep eliminato: sì
- `az group exists`: false

## Terraform

- Terraform version: 1.16.3 (installata durante il lab, non presente in precedenza)
- provider AzureRM: 5.4 (da versions.tf)
- `terraform init`: completato, .terraform/ e .terraform.lock.hcl creati
- `terraform fmt -check`: nessuna differenza (file già formattati)
- `terraform validate`: Success!
- plan add: 2
- plan change: 0
- plan destroy: 0
- apply: completato — Apply complete! Resources: 2 added, 0 changed, 0 destroyed
- output Resource Group: rg-ud12-tf
- output Storage: stud12t90071609
- verifica CLI: confermata — SKU Standard_LRS, location northeurope
- `terraform state list`: azurerm_resource_group.lab, azurerm_storage_account.lab

## Fine LAB guidato

- risorse Terraform mantenute per LAB autonomo: sì