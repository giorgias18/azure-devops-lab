# Consegna UD03 — Verifica

## Parte A — Scelta singola

Per le domande 1–8 riporta risposta e motivazione.

1. B. L'utente è autenticato, ma non dispone di un ruolo Azure RBAC che gli permetta di leggere il Resource Group
2. D. Una role assignment Azure è composta da Principal + Role definition + Scope.
3. C. È il livello minimo per consentire la sola consultazione delle risorse.
4. B. Un ruolo assegnato a una subscription viene normalmente ereditato dalle risorse e Resource Group sottostanti.
5. B. Può gestire le risorse, ma normalmente non può assegnare ruoli Azure RBAC. Per quello serve tipicamente Owner o un ruolo specifico come User Access Administrator.
6. C. Un lock CanNotDelete impedisce le operazioni di eliminazione, senza impedire normalmente la lettura o la modifica.
7. B. Il raggiungimento del 100% del budget può attivare gli alert configurati, ma non arresta automaticamente le risorse.
8. B. La creazione e gestione degli utenti cloud avviene in Microsoft Entra ID, con le autorizzazioni/ruoli Entra necessari.

## Parte B — Risposte brevi

9. Assegnare un ruolo direttamente al gruppo semplifica la gestione degli accessi e permette di aggiungere/rimuovere utenti senza modificare ogni assegnazione individualmente.
10. Microsoft Entra gestisce identità e tenant, ad esempio User Administrator; Azure invece gestisce risorse Azure, ad esempio Contributor su un Resource Group.
11. Contributor effettivo: è ereditato dalla subscription e quindi si applica anche al Resource Group. Essendo più permissivo di Reader, prevale nell'accesso risultante.
12. 1 - identità, 2 - scope, 3 - role assignment dirette/ereditate, 4 - permessi del ruolo, 5 - eventuali deny/policy.
13. Il tag deleteAfter è per i casi con informazione usata eventualmente da automazioni. CanNotDelete: blocca l'eliminazione. Budget invece monitora la spesa e genera alert, ma non blocca automaticamente le risorse.

## Parte C — Caso situazionale

14. **Almeno due scelte/interpretazioni errate**
- Scope troppo ampio: al tecnico è stato dato Contributor sull'intera sottoscrizione per un compito che riguarda solo rg-network-prod. Questo viola il principio di minimo privilegio: il tecnico ottiene accesso in scrittura anche su tutti gli altri resource group della sottoscrizione, non solo su quello che deve consultare;
- Ruolo eccessivo rispetto al compito: il compito è "consultare" (sola lettura), ma è stato assegnato Contributor, che concede anche modifica ed eliminazione delle risorse. Il ruolo corretto per una consultazione sarebbe stato Reader.
15. **Ruolo e scope iniziali più appropriati**
Ruolo: Reader. Scope: il singolo resource group rg-network-prod (non la sottoscrizione)
16. **Perché fallisce ciascuna operazione**
- Non riesce ad assegnare Reader al collega: Contributor concede permessi su risorse (creare, modificare, eliminare), ma non sui permessi di autorizzazione stessa. Assegnare ruoli richiede l'azione Microsoft.Authorization/roleAssignments/write, presente in ruoli come Owner o User Access Administrator, non in Contributor. È un errore di autorizzazione RBAC: ha un ruolo, ma non quello giusto per quell'azione specifica (esattamente il Caso B visto nel laboratorio: AuthorizationFailed ... roleAssignments/write);
- Non riesce a fare cleanup (ScopeLocked): questo non è un problema di permessi RBAC, ma un lock di governance (probabilmente CanNotDelete) attivo sullo scope, indipendente dal ruolo posseduto. Anche con Owner pieno, un lock CanNotDelete impedirebbe comunque l'eliminazione: il lock agisce a un livello diverso e cumulativo rispetto a RBAC, esattamente come visto nel Caso C del laboratorio.
