# Consegna UD02 — Verifica

## Parte A — Scelte operative

Per le domande 1–8 riporta risposta e motivazione.

1. B. Una macchina virtuale offre il controllo del sistema operativo e permette di installare componenti non supportati da servizi gestiti. Questo comporta però una maggiore responsabilità operativa rispetto a un servizio PaaS.

2. C. Con App Service Azure gestisce gran parte dell'infrastruttura e della piattaforma sottostante, mentre il cliente rimane responsabile dell'applicazione, della sua configurazione, degli accessi e dei dati.

3. C. Il resource group permette di raggruppare risorse correlate e di gestirle secondo un ciclo di vita comune.

4. B. Il resource group non obbliga tutte le risorse contenute a utilizzare la stessa region.

5. B. Il comando permette di verificare il contesto Azure CLI attivo, inclusa la sottoscrizione, prima di creare risorse. L'output deve permettere di confermare che la sottoscrizione e lo stato dell'account siano quelli attesi.

6. C. Il nome di uno storage account Azure deve essere globalmente univoco e può contenere soltanto lettere minuscole e numeri, senza spazi, trattini o underscore.

7. B. Un tag è un metadato e non causa automaticamente la cancellazione della risorsa.

8. C. Il risultato atteso è `false`, 

## Parte B — Risposte brevi

9. Un'azienda può utilizzare un cloud pubblico per ospitare il proprio sito web e i servizi rivolti ai clienti, però può mantenere anche alcuni sistemi che richiedono infrastruttura dedicata in un cloud privato. Collegando i due ambienti e permettendo lo scambio controllato di dati e servizi si ottiene un modello cloud ibrido.

10. Il tenant Microsoft Entra rappresenta il contesto delle identità dell'organizzazione; la sottoscrizione Azure contiene le risorse e gestisce aspetti amministrativi, economici e di accesso; il resource group è un contenitore logico all'interno della sottoscrizione e raggruppa risorse che possono condividere ciclo di vita e governance e la risorsa è l'elemento Azure effettivamente creato, ad esempio una VNet o uno storage account.

11. Una region è un'area geografica Azure che contiene uno o più datacenter, mentre una availability zone è una zona fisicamente separata all'interno di una stessa region. Le availability zone servono a migliorare la resilienza contro guasti locali, mentre le region rappresentano una posizione geografica più ampia.

12. Azure Portal offre un'interfaccia grafica utile per esplorare le risorse e visualizzarne le proprietà. Azure CLI permette invece di eseguire controlli e operazioni tramite comandi ripetibili e automatizzabili, rendendola particolarmente utile per script e pipeline.

13. I tag del resource group non devono essere considerati automaticamente presenti sulle singole risorse: se una risorsa deve avere determinati metadati per classificazione, gestione dei costi o automazione, i tag devono essere applicati anche alla risorsa oppure una policy deve garantirne l'applicazione.


## Parte C — Interpretazione tecnica

1. Nell'ID `/subscriptions/<omitted>/resourceGroups/rg-cea-test/providers/Microsoft.Storage/storageAccounts/stceatest01` si riconoscono la sottoscrizione tramite `/subscriptions/<omitted>`, il resource group tramite `/resourceGroups/rg-cea-test`, il provider tramite `Microsoft.Storage`, il tipo tramite `storageAccounts` e il nome della risorsa tramite `stceatest01`.

2. La risorsa si trova nella località `italynorth`. I tag presenti sono `environment=lab` e `unit=UD02`.

3. Per verificare che la risorsa appartenga realmente al resource group userei:

   `az resource show --resource-group rg-cea-test --name stceatest01 --resource-type Microsoft.Storage/storageAccounts`

   Se la risorsa appartiene al resource group, il comando restituisce i dettagli della risorsa; in caso contrario restituisce un errore.

4. Se il resource group contiene soltanto risorse del laboratorio, userei:

   `az group delete --name rg-cea-test --yes --no-wait`

   Il comando avvia l'eliminazione dell'intero resource group e delle risorse che contiene.

5. Dopo l'eliminazione userei:

   `az group wait --name rg-cea-test --deleted`

   e successivamente:

   `az group exists --name rg-cea-test`

   Il risultato atteso dell'ultimo comando è `false`, che conferma che il resource group non esiste più.

