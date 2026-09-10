# Consegna UD04 — Laboratorio autonomo

## Scelta del servizio e della ridondanza

Per il servizio di archiviazione ho scelto Blob Storage, escludendo le altre tre opzioni disponibili in Azure Storage. Azure Files è stato scartato perché pensato per condivisioni file system montabili via SMB o NFS da più macchine, utile per scenari di file server condiviso, ma non per un'applicazione che legge documenti via HTTP. Queue Storage serve a gestire code di messaggi tra componenti applicativi, non a conservare file consultabili dagli utenti, quindi non è pertinente al requisito. Table Storage, infine, è pensato per dati strutturati in forma chiave-valore, non per contenere documenti. Blob Storage invece è nato proprio per contenuti non strutturati acceduti via API HTTP/REST, ed è l'unico dei quattro servizi che offre nativamente tier di accesso, regole di lifecycle e controllo granulare dei permessi sui dati, tutte caratteristiche richieste dallo scenario.

Sul fronte della ridondanza ho confrontato LRS e ZRS. LRS mantiene tre copie sincrone dei dati all'interno dello stesso datacenter: è l'opzione più economica, ma non offre alcuna protezione se l'intera zona di disponibilità in cui si trova il datacenter subisce un guasto. ZRS distribuisce invece le tre copie su zone di disponibilità diverse all'interno della stessa regione, garantendo che un guasto localizzato a una singola zona non comprometta la disponibilità dei dati, a fronte di un costo leggermente superiore. Per questo motivo, in un ambiente di produzione dove il requisito esplicito è la resilienza a un guasto zonale, la scelta corretta sarebbe ZRS. Nel laboratorio invece ho usato LRS, non perché sia la scelta ottimale in assoluto, ma perché si tratta di una risorsa temporanea e di test, priva di reali requisiti di alta disponibilità, per cui LRS permette di minimizzare i costi senza compromettere l'obiettivo didattico dell'esercitazione.

## Operazioni e verifica

Il container archive è stato creato con accesso privato, usando esclusivamente l'identità Entra ID (--auth-mode login), senza alcuna account key coinvolta. La verifica con container show ha confermato PublicAccess: None, in linea con il requisito di nessun accesso pubblico anonimo.
È stata caricata una copia del documento come current/documento.txt, ereditando il tier Hot di default dell'account. Il comando blob show ha confermato nome, tier e dimensione (49 byte), coerenti con il file originale.
Per l'accesso in sola lettura è stata generata una user delegation SAS (--as-user), con permesso limitato a r e durata massima di 15 minuti come richiesto. Il download tramite curl è andato a buon fine e il confronto con cmp non ha rilevato differenze rispetto all'originale, confermando l'integrità del file. Il token SAS non è stato in alcun momento stampato, salvato o riportato nella consegna.

## Diagnosi

1. AuthorizationPermissionMismatch

**Piano interessato**: piano dati (Blob data plane)
**Sintomo**: la chiamata restituisce errore di autorizzazione anche se l'utente ha un ruolo di gestione (es. Contributor) sulla risorsa.
**Causa**: manca (o non si è ancora propagata) una role assignment sul piano dati, es. Storage Blob Data Contributor/Reader; i ruoli di gestione non danno accesso ai dati.
**Controllo**: verificare in IAM che sia assegnato un ruolo dati corretto, con lo scope giusto, e attendere la propagazione.
**Correzione**: assegnare il ruolo dati mancante sullo scope corretto (storage account o container), senza allargare a Owner/Contributor generico.

2. ResourceNotFound: The specified container does not exist

**Piano interessato**: piano dati, ma è un errore di risorsa, non di permessi.
**Sintomo**: il comando fallisce indicando che il container non esiste.
**Causa**: nome del container errato/typo, container non ancora creato, oppure si sta puntando allo storage account sbagliato.
**Controllo**: az storage container list --account-name ... --auth-mode login per vedere i container realmente esistenti.
**Correzione**: correggere il nome nel comando, oppure creare il container se manca davvero.

3. curl: (22) The requested URL returned error: 403

**Piano interessato**: piano dati, accesso via URL/SAS.
**Sintomo**: il download HTTP diretto (via URL, tipicamente SAS) viene rifiutato con 403 Forbidden.
**Causa**: possibili cause, come SAS scaduta, permessi della SAS insufficienti (es. solo lista ma non lettura), SAS generata per un blob/container diverso, o firma non più valida.
**Controllo**: rigenerare la SAS con --expiry valido e --permissions r corretto, verificando nome blob/container esatti.
**Correzione**: generare una nuova SAS con parametri corretti, senza riutilizzare token scaduti o mal configurati.

## Lifecycle, costi e cleanup

La lifecycle rule delete-temporary filtra i blob tramite prefixMatch impostato su documents/temporary/, cioè applica l'azione solo ai blob il cui nome completo (container incluso) inizia esattamente con quel prefisso. Il blob archive/current/documento.txt si trova in un container diverso (archive, non documents) e sotto un percorso diverso (current/, non temporary/), quindi non soddisfa il filtro su nessuno dei due livelli. La regola non lo considera e il blob non viene eliminato, comportamento coerente con lo scenario, dove i documenti "current" devono restare disponibili per 30 giorni, a differenza dei file temporanei.

## Risultato finale

- nessun segreto pubblicato: conferma
- hash abbreviato e messaggio del commit: 4179baa "Completa lab guidato UD04 - Storage account e Blob"