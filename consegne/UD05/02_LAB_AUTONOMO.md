# Consegna UD05 — Laboratorio autonomo

## Inventario iniziale

- VNet: `vnet-ud05` (10.50.0.0/16), resource group `<resource-group>`, subscription `<subscription-id>`
- Subnet `snet-web`: 10.50.10.0/24 — NIC `nic-web-01`, IP privato 10.50.10.4, nessun IP pubblico
- Subnet `snet-data`: 10.50.20.0/24 — NIC `nic-data-01`, IP privato 10.50.20.4, nessun IP pubblico
- NSG associato a `snet-data`: `nsg-data-ud05`
- Regola presente prima del guasto: `Allow-Web-Postgres` — priorità 300, Inbound, Allow, Tcp, origine 10.50.10.0/24, destinazione *, porta 5432

## Guasto e diagnosi

- regola introdotta: `Deny-Web-Postgres-Auto` — priorità 250, Inbound, Deny, Tcp, origine 10.50.10.0/24, destinazione *, porta 5432
- ordine di priorità osservato: `Deny-Web-Postgres-Auto` (250) precede `Allow-Web-Postgres` (300); a parità di origine/porta, la priorità numerica più bassa vince
- sintomo: traffico TCP 5432 da `snet-web` verso `snet-data` risulterebbe bloccato
- ipotesi: la regola Deny con priorità inferiore intercetta il traffico prima che Azure valuti la regola Allow sottostante
- controllo: `az network nsg rule list` ordinato per priorità; tentativo di `list-effective-nsg` / `show-effective-route-table` su `nic-data-01` non eseguibile, errore `NicMustBeAttachedToRunningVmToGetEffectiveSecurityGroups/Routes` (NIC non collegata a una VM in esecuzione)
- correzione minima: rimuovere la regola `Deny-Web-Postgres-Auto` oppure assegnarle una priorità numerica più alta rispetto ad `Allow-Web-Postgres`
- verifica dopo la correzione: `az network nsg rule list` mostra unicamente `Allow-Web-Postgres` (300) come regola custom, tornata prima e unica regola corrispondente per quel traffico

## Casi ulteriori

Distinzione tra fatti osservati (deducibili dalla configurazione statica, senza VM) e prove non ancora eseguibili (richiedono una VM in esecuzione):

- **CIDR (`InvalidAddressPrefix`)**, livello: validazione ARM prima del deployment. Fatto osservato: non si è verificato nessun errore di questo tipo (i prefissi usati erano validi e contenuti nella VNet, verificato nel lab guidato). Correzione eventuale: correggere il prefisso CIDR nel parametro del comando. Prova non eseguibile qui: non è stato necessario ripetere un test di errore reale.
- **Regole NSG / porta filtrata (`SecurityRuleConflict`)**, livello: NSG. Fatto osservato: dedotto e confermato da `az network nsg rule list` ordinato per priorità (Deny 250 precede Allow 300). Prova non eseguibile: IP Flow Verify reale, che richiederebbe una VM in esecuzione su `nic-data-01`.
- **Routing**, livello: route di sistema/UDR. Fatto osservato: nessuno, comando `show-effective-route-table` fallito per lo stesso motivo (NIC non attaccata a VM). Prova non eseguibile: verifica delle route effettive.
- **DNS (nome non risolto)**, livello: risoluzione nomi, a monte del filtro di rete. Fatto osservato: non applicabile a questo scenario (non sono stati modificati DNS server della VNet in questo lab). Prova non eseguibile: `nslookup`/`dig` dall'interno di una VM reale.
- **Servizio non in ascolto**, livello: applicativo, sulla VM `nic-data-01` (es. Postgres non avviato o in ascolto su interfaccia/porta diversa). Fatto osservato: non verificabile, nessun servizio/VM in esecuzione in questo lab. Prova non eseguibile: `netstat`/`ss` sulla VM per confermare che Postgres sia effettivamente in ascolto sulla 5432.

## Cleanup e risultato finale

- regola autonoma rimossa: sì — `Deny-Web-Postgres-Auto` eliminata e verificata (solo `Allow-Web-Postgres` presente dopo la rimozione)
- cleanup verificato: cleanup delle risorse di rete UD05 completato (nic-data-01, nic-web-01, NSG nsg-data-ud05 e VNet vnet-ud05 con relative subnet eliminati; verificato con liste vuote su vnet/nsg/nic nel resource group). Cleanup dello storage account UD04 (stcea46663063) e del resource group rg-cea-storage-46663063 rimandato alla sezione finale
- hash abbreviato e messaggio del commit: c84e44b "Lab Guidato UD05 prima del clean up"
