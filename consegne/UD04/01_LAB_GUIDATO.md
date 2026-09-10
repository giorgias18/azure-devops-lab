# Consegna UD04 — Laboratorio guidato

## Contesto anonimizzato

- resource group: rg-cea-storage-<suffix-random>
- storage account: stcea<suffix-random>
- region: italynorth
- tipo e ridondanza: StorageV2, Standard_LRS (ridondanza locale, tier Hot)

## Servizi e configurazione

| Elemento | Configurazione | Motivazione |
|---|---|---|
| Blob container | `documents`, accesso privato | Nessun bisogno di accesso pubblico anonimo per un documento didattico |
| access tier | Hot | File di piccole dimensioni, accesso frequente durante il lab |
| accesso pubblico | Disabilitato a livello di account (`allowBlobPublicAccess: false`) | Impedisce che un singolo container/blob venga reso pubblico per errore, indipendentemente dalla configurazione del container |
| trasferimento/TLS | HTTPS only, TLS minimo 1.2 | Cifratura del traffico obbligatoria, in linea con le best practice di sicurezza dati |

## Autorizzazione e lifecycle

Per l'accesso ai dati è stato assegnato il ruolo **Storage Blob Data Contributor** con scope limitato al singolo storage account (non alla subscription), seguendo lo stesso principio di minimo privilegio già applicato nel precedente UD03 per il piano di gestione.

Sono state confrontate tre modalità di autorizzazione:
- **Microsoft Entra ID** (`--auth-mode login`): le operazioni sono legate all'identità dell'utente, tracciabili e revocabili tramite role assignment;
- **Shared Key**: chiave dell'account che garantisce accesso ampio a tutto lo storage account, senza distinguere chi la usa. Non stampata o non salvata in chiaro, va rimossa dall'ambiente subito dopo l'uso;
- **User delegation SAS**: token temporaneo (30 minuti), firmato con l'identità Entra dell'utente tramite `--as-user`, con permessi granulari (in questo caso sola lettura su un singolo blob). Combina la sicurezza dell'identità Entra con la possibilità di condividere un accesso limitato nel tempo senza esporre credenziali permanenti.

È stata inoltre configurata una **regola di lifecycle management** (`delete-temporary`) che elimina automaticamente i blob sotto il prefisso `documents/temporary/` trascorso 1 giorno dall'ultima modifica, verificata sia da portale sia da CLI (`az storage account management-policy show`).

## Verifiche, costi e cleanup

Le operazioni di lettura/scrittura tramite identità Entra sono state verificate con `az storage container list` e `az storage blob list` (`--auth-mode login`), inizialmente falliti per propagazione del ruolo non ancora completata, poi risolti in autonomia senza allargare lo scope del ruolo. L'integrità dei file scaricati è stata verificata con `cmp` (via download diretto e via SAS), senza differenze.

Principali driver di costo: capacità dati occupata (tier Hot), transazioni di lettura/scrittura, ridondanza LRS scelta come opzione più economica per un ambiente di lab. Il cleanup finale prevede la rimozione della role assignment temporanea e l'eliminazione del resource group, verificata con `az group exists`.

## Rilevanza professionale

Per un caso reale tra Blob Storage e Azure Files: **Blob Storage** è la scelta corretta per contenuti non strutturati acceduti via API/HTTP (documenti, backup, log, asset statici), con autorizzazione basata su RBAC dati (Entra ID) come metodo primario e SAS per condivisioni temporanee verso terzi. **Azure Files** sarebbe preferibile quando serve un file share montabile via SMB/NFS da più macchine contemporaneamente (es. condivisione di rete legacy), dove l'autorizzazione tipicamente si appoggia su identità di dominio (Entra ID Kerberos o Active Directory) oltre che su Shared Key.