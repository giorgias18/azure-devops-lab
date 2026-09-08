# Consegna UD02 — Laboratorio guidato

## Contesto verificato

- Azure Portal accessibile: sì, accesso confermato
- Azure CLI autenticata: sì, verificata con `az account show`
- sottoscrizione corretta verificata senza pubblicarne l'ID: sì, tramite `az account show --query "{Name:name,State:state,IsDefault:isDefault}"`
- località scelta e motivo: italynorth, disponibile nella sottoscrizione e verificata con `az account list-locations` (regione più vicina)

## Ambiente creato

| Elemento | Nome tecnico | Tipo | Località | Scopo |
|---|---|---|---|---|
| Resource group | rg-cea-ud02-5a3d394d | Microsoft.Resources/resourceGroups | italynorth | contenitore del ciclo di vita delle risorse del laboratorio |
| Rete virtuale | vnet-cea-ud02 | Microsoft.Network/virtualNetworks | italynorth | rete logica di prova con subnet snet-app (10.20.1.0/24) |
| Storage account | stcea5a3d394d | Microsoft.Storage/storageAccounts | italynorth | account storage vuoto per osservare configurazione e sicurezza |

## Decisioni e verifiche

Resource group, VNet e storage account sono stati creati nello stesso resource group perché condividono lo stesso ciclo di vita: essendo nati insieme per questo laboratorio, devono anche essere eliminati insieme, e il resource group permette di farlo con un solo comando (`az group delete`) invece di rimuovere ogni risorsa singolarmente. In un modello di responsabilità condivisa come questo, Azure si occupa dell'infrastruttura fisica, della disponibilità della piattaforma e della sicurezza dei data center, mentre io come cliente resto responsabile della configurazione delle singole risorse (ad esempio aver disabilitato l'accesso blob pubblico, impostato TLS 1.2 e scelto la ridondanza LRS) oltre che della gestione degli accessi e degli eventuali dati caricati. Anche la scelta di nomi e tag riflette questa responsabilità: i nomi seguono una convenzione prevedibile (`rg-`, `vnet-`, `st` + prefisso corso/unità + suffisso univoco) che evita collisioni e rende subito riconoscibile lo scopo di ogni risorsa, mentre i tag (`course`, `unit`, `environment`, `deleteAfter`) permettono di filtrare le risorse, attribuirne il costo e segnalare quando andrebbero eliminate, anche nel caso in cui il cleanup manuale venga dimenticato.

Ho confrontato la risorsa `stcea5a3d394d` (storage account) tramite Azure Portal e tramite `az storage account show`. I valori principali (nome, location, SKU, kind, provisioning state, TLS minimo, tag) coincidono perfettamente tra i due strumenti, confermando che rappresentano lo stesso stato reale della risorsa.

La JSON View del portale mostra automaticamente tutte le proprietà della risorsa, inclusi campi che non avevo richiesto, come `networkAcls`, `encryption`, `keyCreationTime` e i sei primaryEndpoints (`blob`, `file`, `queue`, `table`, `web`, `dfs`). Lo reputo utile per esplorare una risorsa e scoprire proprietà che non conoscevo ancora; la CLI, al contrario, restituisce solo i campi specificati nella `--query`: nessun rumore, nessuna proprietà superflua. Questo la rende più adatta a controlli ripetibili e mirati, specialmente in script o pipeline, dove serve un output prevedibile e filtrabile (ad esempio con `--output tsv` per estrarre un singolo valore). Un'altra differenza che ho notato è che nel portale il Subscription ID compare sempre in chiaro nell'interfaccia (es. nel pannello Essentials) e sono io a dover decidere cosa non copiare nell'evidenza. Da CLI, invece, ho potuto anonimizzarlo automaticamente con sed prima ancora di salvare l'output, riducendo il rischio di esporre dati sensibili per errore.

Concludo dicendo nessuno dei due strumenti è "migliore" in assoluto; il portale eccelle nell'esplorazione visiva e nella scoperta, la CLI nella precisione, ripetibilità e sicurezza dei controlli automatizzati.

Esempio di ID confrontato (anonimizzato):
`/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-5a3d394d/providers/Microsoft.Storage/storageAccounts/stcea5a3d394d`

## Cleanup

- operazione di eliminazione:
- controllo utilizzato:
- risultato finale:
- eventuale anomalia e soluzione:

## Rilevanza professionale

Spiega come inventario, tag e verifica del cleanup rendono una procedura ripetibile e controllabile.

