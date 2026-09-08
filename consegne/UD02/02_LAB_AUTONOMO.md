# Consegna UD02 — Laboratorio autonomo

## Requisito e piano

- requisito interpretato: creare un ambiente Azure temporaneo e isolato (resource group dedicato, VNet con subnet, storage account vuoto) per provare un componente di sviluppo, verificarlo tramite portale e CLI, documentare le decisioni ed eliminarlo completamente come unità;
- risorse previste: resource group, rete virtuale con subnet dedicata, storage account;
- nomi e tag scelti: `rg-cea-ud02-auto-421fe3b8`, `vnet-cea-auto` (subnet `snet-workload`, 10.30.10.0/24), `stceaauto421fe3b8`; tag `course=cloud-engineer-academy`, `unit=UD02`, `environment=dev`, `scenario=autonomous`, `deleteAfter=2026-09-10`;
- verifiche preliminari: `az account show --query "{Name:name,State:state,IsDefault:isDefault}"` → sottoscrizione `Enabled` e di default confermata prima di procedere con qualsiasi comando.

## Svolgimento

Il resource group `rg-cea-ud02-auto-421fe3b8` nasce come test isolato e temporaneo, quindi tutte le sue risorse nascono insieme e devono poter essere eliminate insieme con un solo comando (`az group delete`). La località scelta è `italynorth`, la stessa già verificata in precedenza tramite `az account list-locations`.

`VNet` e `storage account` sono concettualmente diversi: la `VNet` definisce la rete isolata (10.30.0.0/16) su cui si affaccia il componente di sviluppo, con la `subnet snet-workload` (10.30.10.0/24) dedicata al carico di lavoro; lo storage account è invece uno spazio di archiviazione indipendente dalla rete, protetto da HTTPS obbligatorio, TLS minimo 1.2 e accesso `Blob` pubblico disabilitato.

I tag (`course`, `unit`, `environment`, `scenario`, `deleteAfter`) rendono tracciabili scopo, contesto e scadenza della risorsa anche a distanza di tempo, così da poter identificare cosa va eliminato anche se il cleanup manuale venisse dimenticato.

Verifica VNet/subnet:

    az network vnet show --resource-group "$AUTO_RG" --name "$AUTO_VNET" --query "{Address:addressSpace.addressPrefixes,Subnets:subnets[].{Name:name,Prefix:addressPrefix}}" --output jsonc

Output:

    {
      "Address": ["10.30.0.0/16"],
      "Subnets": [{"Name": "snet-workload", "Prefix": "10.30.10.0/24"}]
    }

Verifica proprietà di sicurezza dello storage account:

    az storage account show --resource-group "$AUTO_RG" --name "$AUTO_STORAGE" --query "{TLS:minimumTlsVersion,HTTPS:enableHttpsTrafficOnly,PublicBlob:allowBlobPublicAccess,Tags:tags}" --output jsonc

Output:

    {
      "HTTPS": true,
      "PublicBlob": false,
      "TLS": "TLS1_2",
      "Tags": {"course": "cloud-engineer-academy", "deleteAfter": "2026-09-10", "environment": "dev", "scenario": "autonomous", "unit": "UD02"}
    }

Confronto CLI vs Portale: `az resource list --resource-group "$AUTO_RG" --query "[].{Name:name,Type:type}" --output table` ha restituito `vnet-cea-auto (Microsoft.Network/virtualNetworks)` e `stceaauto421fe3b8 (Microsoft.Storage/storageAccounts)`. Il portale, nel resource group `rg-cea-ud02-auto-421fe3b8`, ha mostrato lo stesso identico insieme di due risorse, entrambe in Italy North: il match tra i due strumenti è confermato.

Esempio di ID confrontato (anonimizzato):
/subscriptions/<omitted>/resourceGroups/rg-cea-ud02-auto-421fe3b8/providers/Microsoft.Storage/storageAccounts/stceaauto421fe3b8

## Diagnosi

Non si sono verificate anomalie durante l'esecuzione: la sottoscrizione è risultata corretta al primo controllo, i nomi generati con il suffisso casuale (`421fe3b8`) non hanno avuto collisioni, e tutte le proprietà create hanno corrisposto ai requisiti richiesti fin dal primo tentativo.

## Cleanup e consegna

- risorse eliminate: resource group `rg-cea-ud02-auto-421fe3b8` (con VNet e storage account al suo interno), eliminato come unità;
- controllo finale: `az group delete --name "$AUTO_RG" --yes --no-wait`, seguito da `az group wait --name "$AUTO_RG" --deleted` (completato in circa 1 minuto) e `az group exists --name "$AUTO_RG"` → `false`;
- hash abbreviato e messaggio del commit: `58f42c2` "Completa lo scenario Azure autonomo".