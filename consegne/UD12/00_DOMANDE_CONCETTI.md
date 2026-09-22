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

## 7. A cosa servono `@minLength` e `@maxLength`?
**Rende il file riutilizzabile: lo stesso template può essere usato con valori diversi (es. regioni diverse) senza riscrivere la risorsa.**

## 8. @minLength/@maxLength? 
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

## 21.
**Risposta:**

## 22.
**Risposta:**

## 23.
**Risposta:**

## 24.
**Risposta:**

## 25.
**Risposta:**

## 26.
**Risposta:**

## 27.
**Risposta:**

## 28.
**Risposta:**

## 29.
**Risposta:**

## 30.
**Risposta:**

## 31.
**Risposta:**

## 32.
**Risposta:**

## 33.
**Risposta:**

## 34.
**Risposta:**

## 35.
**Risposta:**

## 36.
**Risposta:**

## 37.
**Risposta:**

## 38.
**Risposta:**

## 39.
**Risposta:**

## 40.
**Risposta:**

## 41.
**Risposta:**

## 42.
**Risposta:**

