# Consegna UD03 — Laboratorio autonomo

## Analisi dell'accesso

| Principal anonimizzato | Ruolo | Scope | Origine | Accesso effettivo |
|---|---|---|---|---|
| grp-cea-readers-fd36b9 (Group, team FinOps simulato) | Reader | resource group rg-cea-identity-fd36b9 | Diretta (assegnata dal portale in questo laboratorio) | Sola lettura su risorse e costi del resource group; nessuna possibilità di modifica o gestione accessi |
| account operatore (User) | Owner | sottoscrizione | Ereditata | Accesso pieno derivante da assegnazione preesistente a livello di sottoscrizione, non creata da questo laboratorio |

Per il team FinOps ho scelto **Reader** come ruolo, sul **resource group** come scope (non l'intera sottoscrizione). Lo scope è il resource group perché rappresenta il singolo ambiente applicativo che il team FinOps deve consultare: la sottoscrizione includerebbe anche altri ambienti fuori dal loro perimetro di competenza, e assegnare lì il ruolo violerebbe il principio di minimo privilegio anche a parità di permesso (Reader). Contributor e Owner sono esclusi perché concederebbero rispettivamente la possibilità di modificare risorse e di gestire accessi/permessi e privilegi non necessari per un team che deve solo consultare costi e inventario. Il Reader diretto sul gruppo non riduce eventuali privilegi più ampi già posseduti da altri principal: le assegnazioni RBAC si sommano, non si sostituiscono a scope diversi.

## Diagnosi dei casi

**Caso A** — `Please run 'az login' to setup account.`
- Sintomo: qualsiasi comando az fallisce prima di eseguire l'operazione richiesta;
- Causa probabile: sessione CLI non autenticata o token scaduto;
- Verifica: `az account show --output table`;
- Correzione: `az login` (o `az login --use-device-code` se il browser non è disponibile);
- Risultato atteso: `az account show` restituisce nome sottoscrizione, stato Enabled e utente corretto;

**Caso B** — `AuthorizationFailed ... Microsoft.Authorization/roleAssignments/write ...`
- Sintomo: la creazione di una role assignment fallisce nonostante il login sia valido;
- Causa probabile: il principal autenticato non ha un ruolo con permesso `roleAssignments/write` sullo scope target (manca es. Owner o User Access Administrator);
- Verifica: `az role assignment list --scope "$RG_SCOPE" --include-inherited --output table` sul proprio account, per controllare i ruoli posseduti;
- Correzione: richiedere l'assegnazione del ruolo necessario a chi ha i permessi, oppure usare Check access per individuare un'assegnazione già esistente;
- Risultato atteso: la role assignment compare nell'elenco solo dopo che il principal corretto la crea.

**Caso C** — `ScopeLocked: The scope is locked and can't be deleted.`
- Sintomo: `az group delete` fallisce con questo errore;
- Causa probabile: un lock di tipo CanNotDelete è attivo sullo scope;
- Verifica: `az lock list --resource-group "$LAB_RG" --output table`;
- Correzione: rimuovere il lock (solo se autorizzati e se il rimuoverlo è la scelta corretta) prima di ritentare la delete
- Risultato atteso: dopo la rimozione del lock, `az group delete` procede; letture come `az group show` restano possibili anche a lock attivo.

## Budget, lock e cleanup

- **Budget** (`budget-cea-fd36b9`, mensile, 10€, soglia 80%): funzione di monitoraggio e alert al superamento soglia; limite: non blocca la spesa né impedisce la creazione git add consegne/UD03/02_LAB_AUTONOMO.mddi risorse, è solo notifica;
- **Lock** (`lock-cea-delete`, CanNotDelete): protegge da eliminazioni accidentali dello scope; limite: blocca solo operazioni di delete, non impedisce lettura né altre modifiche, e può ostacolare cleanup/automazioni se non rimosso in tempo;
- **RBAC Reader**: limita l'accesso in scrittura del principal sullo scope, ma non ha effetto su budget o lock — sono controlli indipendenti e cumulativi;

Ordine seguito nel cleanup finale:
1. Rimosso il budget dal portale (dopo aver prima rimosso il lock, che bloccava anche la delete del budget — vedi nota sotto)
2. Rimossa la role assignment Reader dal gruppo, verificata con `az role assignment list --output table` (restava solo Owner ereditato)
3. Rimosso il lock via CLI con `az lock delete`, verificato con `az lock list` → tabella vuota
4. Rimosso l'utente dal gruppo, poi eliminati gruppo e utente dal portale
5. Eliminato il resource group con `az group delete`, atteso con `az group wait --deleted`, confermato con `az group exists` → `false`

Nota: durante l'esecuzione è emerso che il lock blocca la delete di *qualsiasi* risorsa al suo interno, non solo del resource group, infatti il tentativo di eliminare il budget mentre il lock era ancora attivo ha dato lo stesso errore `ScopeLocked` visto nel Caso C. Per questo il lock è stato rimosso per primo, prima del budget.

## Risultato finale

- output anonimizzati utilizzati: `az role assignment list --scope "$RG_SCOPE" --include-inherited --output jsonc` con subscription ID mascherato via `sed`;
- cleanup verificato: **eseguito**, rimossi budget, lock, role assignment Reader, utente e gruppo; resource group `rg-cea-identity-fd36b9` eliminato e confermato con `az group exists` → `false`
- hash abbreviato e messaggio del commit: 7215a65 "completamento ultima parte lab autonomo" 

