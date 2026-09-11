# Consegna UD05 — Laboratorio guidato

## Piano di indirizzamento

| Elemento | CIDR | Scopo | Sovrapposizioni |
|---|---|---|---|
| VNet | 10.50.0.0/16 | Spazio di indirizzamento complessivo per l'ambiente di segmentazione | — |
| subnet web | 10.50.10.0/24 | Ospita le risorse "front-end" (nic-web-01) | Nessuna con snet-data (terzo ottetto diverso: 10 vs 20) |
| subnet data | 10.50.20.0/24 | Ospita le risorse "back-end" protette da NSG (nic-data-01) | Nessuna con snet-web |

## NSG e associazioni

| NSG | Scope associato | Regola | Priorità | Origine | Porta | Esito |
|---|---|---|---:|---|---|---|
| nsg-data-ud05 | subnet snet-data | Allow-Web-Postgres | 300 | 10.50.10.0/24 | TCP 5432 | Allow |
| nsg-data-ud05 | subnet snet-data | Deny-Web-Postgres | 200 | 10.50.10.0/24 | TCP 5432 | Deny (prevaleva su priorità 300) |

## Verifica effettiva

- **NIC create e subnet**: nic-data-01 in snet-data (senza IP pubblico), nic-web-01 in snet-web (IP privato 10.50.10.4, senza IP pubblico);
- **NSG effettivi osservati**: non disponibili, `list-effective-nsg` richiede una NIC collegata a una VM in esecuzione;
- **route effettive osservate**: non disponibili per lo stesso motivo;
- **ciò che è stato verificato**: configurazione statica corretta tramite `az network nsg rule list` (regola Allow-Web-Postgres presente e ben formata) e associazione NSG↔subnet confermata; comportamento delle priorità dimostrato creando e rimuovendo il guasto didattico (priorità numerica più bassa vince, a prescindere dall'ordine di creazione);
- **ciò che richiede ancora un workload**: la verifica di connettività end-to-end (effective security rules ed effective routes) sarà completata quando le NIC verranno collegate a VM in esecuzione.

## Costi e cleanup

Risorse create: VNet vnet-ud05 con subnet snet-web e snet-data, NSG nsg-data-ud05 con relativa regola, NIC nic-data-01 e nic-web-01. Nessuna VM né IP pubblico creati, quindi il costo attuale è marginale (le NIC senza VM associata e le NSG non generano costi diretti significativi). || Il cleanup (eliminazione di NIC, NSG e VNet) è rimandato a dopo il completamento del lab autonomo e della verifica, per evitare di dover ricreare le risorse.

## Rilevanza professionale

Progettare prima il piano di indirizzamento (CIDR di VNet e subnet, verifica di contenimento e assenza di sovrapposizioni) evita conflitti costosi da correggere una volta che le risorse sono già in uso da altri team o collegate a workload di produzione. Allo stesso modo, controllare le regole NSG e il loro ordine di priorità *prima* di collegare un workload permette di individuare errori di configurazione (come una regola di blocco più prioritaria di una regola di permesso) senza causare un'interruzione di servizio reale. In un contesto professionale, questa disciplina di verifica preventiva riduce il rischio di downtime e rende più semplice il troubleshooting, perché si parte da una base di rete già validata invece di dover diagnosticare contemporaneamente problemi di rete e problemi applicativi.

