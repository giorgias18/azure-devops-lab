# UD12 — Domande sui concetti

Rispondere dopo avere studiato `00_CONCETTI.md`. Non limitarsi a definizioni di una riga: quando possibile aggiungere un piccolo esempio.

## 1. Quale problema risolve l'Infrastructure as Code rispetto a una configurazione esclusivamente manuale? 
**Il fatto che ripetere in modo identico una configurazione fatta manualmente (Portale/CLI) diventa sempre più difficile man mano che l'infrastruttura cresce, perché bisogna ricordare a memoria opzioni, valori e ordine delle operazioni.**

## 2. Spiega con parole semplici la differenza tra approccio imperativo e dichiarativo. 
**Imperativo = descrivo le operazioni da eseguire ("crea RG, poi crea Storage..."); dichiarativo = descrivo lo stato finale desiderato, e lo strumento decide quali operazioni servono per raggiungerlo.**

## 3. Perché continuiamo a usare Azure CLI anche se introduciamo IaC? 
**Serve ancora per verificare le risorse, fare troubleshooting, preparare prerequisiti e controllare in modo indipendente ciò che Bicep/Terraform hanno creato.**

## 4. Che cos'è Bicep? 
**Un linguaggio dichiarativo progettato specificamente per descrivere risorse Azure, file con estensione `.bicep.`**

## 5. Bicep sostituisce Azure Resource Manager? 
**No, è un modo più leggibile e compatto per descrivere ciò che Azure Resource Manager distribuirà.**

## 6. Che cosa significa `param location string`? 
**Dichiara un parametro di nome location, di tipo testuale, che il file Bicep può ricevere dall'esterno (qui con default = regione del Resource Group).**

## 7. A cosa serve un parametro?
**Rende il file riutilizzabile: lo stesso template può essere usato con valori diversi (es. regioni diverse) senza riscrivere la risorsa.**

## 8. A cosa servono `@minLength` e `@maxLength`?
**Sono decorator che impongono vincoli sulla lunghezza minima e massima del valore del parametro.**

## 9. Nella riga `resource storage 'Microsoft.Storage/storageAccounts@2023-05-01'`, che cosa significa `storage`?
**È il nome simbolico usato solo internamente nel file Bicep per riferirsi alla risorsa; non è necessariamente il nome reale in Azure.**

## 10. Che cosa significa `Microsoft.Storage/storageAccounts`?
**Identifica il tipo di risorsa Azure: Microsoft.Storage è il Resource Provider, storageAccounts è il tipo di risorsa gestito da quel provider.**

## 11. Che cosa significa `@2023-05-01`? 
**È la versione dell'API Azure usata per interpretare le proprietà di quel tipo di risorsa.**

## 12. È la data di creazione dello Storage Account?
**No, non è né la data di creazione né la data del file.**

## 13. Qual è la differenza tra nome simbolico Bicep e nome reale Azure?
**Il nome simbolico (`storage`) serve solo al codice per riferirsi alla risorsa; il nome reale (dal parametro `storageName`) è quello che vedremo effettivamente nel Portale/CLI.**

## 14. A cosa servono gli output Bicep?
**Restituiscono valori del deployment (es. nome risorsa, endpoint) che possono essere letti dopo l'esecuzione o usati come input in una fase successiva di pipeline.**

## 15. Che cosa fa `az bicep lint`?
**Controlla il file Bicep per individuare problemi prima del deployment (un lint riuscito non garantisce comunque che Azure accetterà ogni operazione).**

## 16. Che cosa fa What-If?
**Mostra le modifiche previste confrontando ciò che esiste già con ciò che il template descrive.**

## 17. What-If crea realmente le risorse?
**No, come suggerisce il nome del comando, mostra solo una previsione.**

## 18. Qual è la differenza tra What-If e deployment `create`?
**What-If mostra appunto cosa succederebbe; create esegue realmente il deployment.**

## 19. Perché Bicep non richiede un file equivalente a `terraform.tfstate`?
**Perché lo stato reale delle risorse è mantenuto da Azure Resource Manager stesso, non serve una componente locale aggiuntiva da amministrare.**

## 20. Che cos'è Terraform?
**Uno strumento IaC dichiarativo, non nato solo per Azure, che lavora con molte piattaforme tramite provider; file con estensione `.tf.`**

## 21. Perché Terraform usa provider?
**Perché Terraform Core non contiene la logica di tutte le piattaforme: il provider (es. AzureRM) funge da adattatore specializzato che sa come gestire le risorse di una specifica piattaforma.**

## 22. Che ruolo ha AzureRM?
**È il provider che conosce i tipi di risorsa Azure (es. azurerm_resource_group, azurerm_storage_account); senza di esso Terraform non saprebbe gestirle.**

## 23. Che cosa contiene `versions.tf`?
**Dichiara la versione minima di Terraform richiesta e il provider necessario (source e vincolo di versione).**

## 24. A cosa serve `providers.tf`?
**Configura/attiva il provider AzureRM per la configurazione (es. blocco `features {}`).**

## 25. Perché credenziali e codice IaC devono restare separati?
**Perché non si vogliono inserire nel repository password, secret o token; le credenziali arrivano dall'ambiente di esecuzione, non dal codice.**

## 26. Che cosa contiene `variables.tf`?
**Dichiara valori configurabili (variabili), con tipo e valore predefinito opzionale.**

## 27. Che differenza c'è tra `azurerm_resource_group`, `lab`, `var.resource_group_name` e `rg-ud12-tf`?
**`azurerm_resource_group` = tipo di risorsa Terraform; `lab` = nome logico interno usato solo nel codice; `var.resource_group_name` = variabile da cui si prende il nome; `rg-ud12-tf` = il valore reale/nome effettivo creato in Azure.**

## 28. Nella riga `resource "azurerm_resource_group" "lab"`, che cos'è `lab`?
**È il nome logico interno usato da Terraform per riferirsi a quella risorsa nel codice.**

## 29. Il Resource Group reale si chiamerà `lab`?
**No, `lab` è solo un riferimento interno, non il nome reale.**

## 30. Da dove arriva il nome reale `rg-ud12-tf`?
**Dal valore predefinito della variabile `resource_group_name` in `variables.tf`.**

## 31. Che cosa significa `azurerm_resource_group.lab.name`?
**Significa "prendi la risorsa Terraform `azurerm_resource_group.lab` e leggine la proprietà `name`"; produce `rg-ud12-tf`.**

## 32. Perché lo Storage Account usa `azurerm_resource_group.lab.name`? 
**Per usare, come nome del Resource Group in cui va creato, esattamente il nome reale del RG dichiarato sopra, mantenendo la relazione tra le due risorse.**

## 33. Che cosa significa `azurerm_resource_group.lab.location`?
**Legge la proprietà `location` del Resource Group lab, cioè `westeurope`.**

## 34. Perché Terraform può dedurre la dipendenza fra Resource Group e Storage Account?
**Perché lo Storage Account fa riferimento direttamente alle proprietà del Resource Group (`.name`, `.location`); da questo riferimento Terraform capisce che il RG deve esistere prima.**

## 35. A cosa serve la validazione di `storage_account_name`?
**Controlla che il nome sia lungo tra 3 e 24 caratteri e contenga solo lettere minuscole e numeri, prima di usarlo.**

## 36. A cosa servono gli output Terraform?
**Espongono valori utili (es. nomi reali delle risorse create) dopo l'esecuzione.**

## 37. Che cosa fa `terraform init`?
**Prepara la directory locale: legge i provider richiesti, scarica AzureRM se necessario, crea `.terraform/` e `.terraform.lock.hcl`. Non crea ancora risorse.**

## 38. Che differenza c'è tra `.terraform/` e `.terraform.lock.hcl`?
**`.terraform/` è materiale locale di lavoro scaricato; `.terraform.lock.hcl` è il lock file che registra le versioni dei provider effettivamente selezionate.**

## 39. Che cosa fa `terraform fmt`?
**Formatta i file HCL secondo lo stile standard; non crea risorse Azure.**

## 40. Che cosa fa `terraform validate`?
**Controlla che la configurazione sia sintatticamente valida e internamente coerente (non garantisce che le operazioni su Azure riusciranno).**

## 41. Che cosa fa `terraform plan`?
**Confronta configurazione desiderata, state e informazioni dal provider, determinando cosa creare/modificare/eliminare.**

## 42. Perché il piano va letto prima dell'apply?
**Per verificare le modifiche previste prima di applicarle realmente, invece di agire "alla cieca".**

## 43. Perché nel LAB salviamo il piano in `ud12.tfplan`?
**Per poter applicare esattamente quel piano già controllato, rendendo netto il passaggio tra previsione e azione reale.**

## 44. Cosa fa `terraform apply`? 
**Esegue realmente le modifiche su Azure secondo il piano.**

## 45. Cos'è lo state Terraform? 
**Un meccanismo (es. file `terraform.tfstate`) che mantiene la relazione tra gli oggetti dichiarati nel codice e le risorse reali gestite su Azure.**

## 46. Quale relazione mantiene lo state? 
**Il collegamento tra l'oggetto dichiarato nel codice (es. `azurerm_storage_account.lab`) e la risorsa reale corrispondente in Azure.**

## 47. Perché `terraform.tfstate` non va trattato come normale codice sorgente?
**Perché può contenere identificativi, proprietà, metadati e anche valori sensibili; va protetto e gestito con attenzione, non semplicemente committato.**

## 48. Che cosa mostra `terraform state list`?
**Gli oggetti che quella specifica configurazione Terraform sta gestendo tramite il proprio state.**

## 49. `terraform state list` mostra tutte le risorse della Subscription? 
**No, solo quelle gestite da quella configurazione/state.**

## 50. `terraform destroy` elimina anche i file `.tf`? 
**No, elimina solo le risorse gestite; il codice IaC resta nel repository.**

## 51. Qual è la differenza principale nel percorso Bicep→Azure rispetto a Terraform→Azure?
**Bicep passa attraverso Azure Resource Manager direttamente; Terraform passa attraverso il provider AzureRM, e in più gestisce uno state locale.**

## 52. In che cosa What-If e Plan sono simili?
**Entrambi mostrano le modifiche previste prima di applicarle, rafforzando l'abitudine di controllare prima di agire.**

## 53. Perché non sono lo stesso meccanismo?
**What-If appartiene ad Azure Resource Manager; Plan appartiene al modello Terraform e lavora con configurazione, state e provider insieme.**

## 54. In quale tipo di organizzazione Bicep può essere particolarmente naturale?
**In organizzazioni quasi interamente su Azure, perché è molto vicino al modello ARM.**

## 55. In quale tipo di organizzazione Terraform può essere particolarmente naturale?
**In organizzazioni con più piattaforme (Azure, AWS, GitHub, Cloudflare, VMware...) grazie al modello a provider comune.**

## 56. Perché non ha senso dire in assoluto che uno dei due è sempre migliore?
**La scelta corretta dipende dagli standard, dalle piattaforme usate e dalle competenze dell'organizzazione, non da una superiorità assoluta di uno strumento.**

## 57. Perché i file IaC devono rimanere nel repository?
**Perché non sono materiale usa-e-getta: fanno parte del progetto, verranno estesi nelle UD successive e seguono un processo simile al codice applicativo (branch, PR, pipeline).**

## 58. Le directory `infra/bicep/` e `infra/terraform/` verranno ricreate da zero in UD13?
**No, verranno estese con nuovi file, non ricreate.**

## 59. Perché installare Terraform nel WSL2 è utile per le UD successive?
**Perché simula la preparazione della toolchain di un vero build host aziendale, necessaria per eseguire i comandi nelle pipeline delle UD successive.**

## 60. Su quale componente vengono realmente eseguiti i comandi di una pipeline?
**Sull'Agent (che a sua volta gira su una macchina, es. il WSL2 nel nostro caso).**

## 61. Che cosa rappresenta il WSL2 del partecipante nel modello self-hosted?
**Rappresenta la macchina/Agent self-hosted, cioè infrastruttura gestita dall'organizzazione (non un semplice PC personale dello sviluppatore, anche se nel corso coincide con la macchina del partecipante).**

## 62. Qual è la differenza principale fra persistenza self-hosted e ambiente Microsoft-hosted?
**Il self-hosted è persistente (gli strumenti installati restano finché non vengono rimossi), il Microsoft-hosted parte sempre da una macchina temporanea nuova per ogni Job, quindi non si può fare affidamento su installazioni precedenti.**

## 63. Perché eliminare le risorse Azure non significa eliminare il codice IaC? 
**Perché risorse Azure e codice IaC hanno lifecycle differenti: le risorse sono temporanee ed eliminabili, il codice resta nel repository e viene riusato in futuro.**