# Consegna UD04 — Verifica

## Parte A — Scelta singola

Per le domande 1–8 riporta risposta e motivazione.

A. Immagini accessibili come oggetti via HTTP è il caso d'uso principale di Blob Storage.
B. Le condivisioni SMB gestite sono la caratteristica distintiva di Azure Files.
B. Contributor è un ruolo di gestione (management plane), non dà accesso ai dati Blob.
B. Replica su zone di disponibilità diverse nella stessa regione, protegge da guasto zonale.
B. Token con permessi granulari e scadenza, per accesso delegato temporaneo.
C. Un budget Azure è solo un avviso, non un limite tecnico.
B. Il prefisso è selettivo.
A. Forza l'autenticazione via Entra ID invece che via chiave.

## Parte B — Risposte brevi

9. **Distingui ridondanza e backup**
La ridondanza (LRS, ZRS, GRS...) protegge dalla perdita dati per guasti hardware/infrastrutturali (disco, zona, datacenter), mantenendo copie sincrone identiche in tempo reale; il backup è invece una copia puntuale nel tempo, pensata per il ripristino da errori logici (cancellazione accidentale, corruzione, ransomware): la ridondanza non protegge da questi ultimi, perché un errore umano viene replicato istantaneamente su tutte le copie.
10. **Distingui management plane e data plane con un comando per ciascuno**
Il management plane riguarda la gestione della risorsa stessa (creazione, configurazione, permessi RBAC di gestione) — es. az storage account create. Il data plane riguarda le operazioni sui dati contenuti nella risorsa — es. az storage blob upload --auth-mode login. Un ruolo di management (es. Contributor) non concede automaticamente permessi sul data plane, come verificato nel lab.
11. **Elenca quattro proprietà di una SAS a minimo privilegio**
- Permessi minimi necessari (es. solo r per sola lettura, non rwdl);
- Scadenza breve (validità limitata nel tempo, es. 15-30 minuti);
- Scope ristretto (su un singolo blob/container, non sull'intero account);
- Firma con identità Entra (--as-user, user delegation SAS) invece che con Shared Key, per tracciabilità e revocabilità.
12. **Perché Archive non è appropriato per dati da recuperare immediatamente**
Il tier Archive è offline: i blob non sono accessibili direttamente, serve prima una rehydration (riattivazione) che può richiedere da alcune ore fino a un giorno, a seconda della priorità scelta. È pensato per dati raramente acceduti (compliance, archiviazione a lungo termine), non per accessi con requisiti di bassa latenza.
13. **Perché non bisogna salvare account key o SAS nel repository**
Sono credenziali con accesso diretto ai dati: chiunque acceda al repository (anche in futuro) potrebbe usarle per leggere/scrivere/eliminare dati senza autenticazione aggiuntiva, senza che l'accesso sia tracciabile a un'identità specifica. Una volta esposte in un commit, vanno considerate compromesse e rigenerate, anche se poi rimosse da commit successivi (restano nella storia Git).

## Parte C — Caso situazionale

14. **Individua almeno tre problemi**
- Contributor è un ruolo di management, non di data plane, quindi non dovrebbe essere usato per concedere accesso ai dati; serve un ruolo dati specifico (es. Storage Blob Data Reader/Contributor) con scope minimo;
- Avere l'account key condivisa concede accesso pieno e non tracciabile a tutto l'account, senza legare le operazioni a un'identità; è l'equivalente di condividere una password amministrativa;
- SAS con permessi completi e senza scadenza breve viola il principio di minimo privilegio su entrambi gli assi (permessi troppo ampi + validità troppo lunga), aumentando enormemente la finestra di rischio in caso di esposizione del token.
15. **Proponi autorizzazione e scope più appropriati**
- Sostituire Contributor con un ruolo data plane mirato (es. Storage Blob Data Reader se serve solo lettura, Contributor dati se serve anche scrittura), con scope limitato al singolo storage account o, se possibile, al singolo container;
- Usare Entra ID come metodo primario per le operazioni interattive/amministrative;
Per condivisioni verso applicazioni esterne o accessi temporanei, generare una user delegation SAS con permessi minimi (es. solo r) e scadenza breve (minuti/ore, non giorni);
-Evitare la distribuzione di account key.
16. **Come verificheresti accesso e cleanup senza pubblicare segreti**
- Verificherei i permessi effettivi con az role assignment list sullo scope corretto, controllando ruolo e scope senza mai stampare chiavi o token;
- Testerei l'accesso con --auth-mode login (es. blob list/blob show) per confermare che l'identità Entra funzioni senza bisogno di chiavi;
- Per il cleanup, verificherei la rimozione delle role assignment temporanee (az role assignment list deve tornare vuoto) e l'eliminazione delle risorse con az group exists (deve restituire false), documentando solo esiti (es. "container non più raggiungibile", "az group exists → False") senza mai riportare URL SAS, chiavi o token nei log/consegne.

