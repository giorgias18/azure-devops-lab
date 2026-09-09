# Consegna UD03 — Laboratorio guidato

## Contesto anonimizzato

- sottoscrizione e tenant verificati: sottoscrizione "Azure subscription 1" (ID mascherato), tenant Default Directory verificato via `az account show`
- percorso Entra eseguito: A (New user e New group entrambi disponibili in Microsoft Entra ID)
- resource group temporaneo: rg-cea-identity-fd36b9, location italynorth, tag course/unit/environment/deleteAfter impostati alla creazione

## Identità e assegnazione RBAC

| Principal anonimizzato | Ruolo | Scope | Diretta/ereditata | Motivo |
|---|---|---|---|---|
| grp-cea-readers-fd36b9 (Group) | Reader | resource group rg-cea-identity-fd36b9 | Diretta | Assegnata dal portale in IAM → Add role assignment, per verificare accesso in sola lettura su uno scope isolato |
| account operatore (User) | Owner | sottoscrizione | Ereditata | Ruolo preesistente sulla sottoscrizione, non creato da questo laboratorio; osservato per contrasto con Reader diretto |

Ho creato un utente cloud temporaneo (alias non personale) e un gruppo di sicurezza con membership Assigned, aggiungendo l'utente come membro. Ho poi assegnato Reader al gruppo sullo scope del resource group. Con Check access ho verificato che il gruppo risulta solo Reader su quello scope, mentre il mio account mostra Owner per eredità dalla sottoscrizione — le due assegnazioni si combinano, non si sostituiscono, e questo mostra la differenza tra un ruolo assegnato direttamente e uno ereditato dall'alto.

## Governance e costi

- tag e significato: course=cloud-engineer-academy, unit=UD03, environment=lab, deleteAfter=<data> servono a identificare scopo, durata e responsabile del resource group temporaneo;
- lock e operazione impedita: lock-cea-delete di tipo CanNotDelete; il tentativo di `az group delete` è fallito con errore ScopeLocked, confermando che il lock protegge da eliminazioni accidentali;
- stato di Cost Analysis: costo effettivo pari a zero, coerente con l'assenza di risorse a consumo nel resource group e con i tempi di elaborazione dei dati di costo;
- budget creato o limitazione documentata: budget-cea-fd36b9, periodo mensile, importo 10€, soglia di alert all'80%;
- motivo per cui il budget non blocca la spesa: un budget è solo monitoraggio e notifica al superamento soglia — non impedisce la creazione di risorse né arresta servizi già attivi; per bloccare davvero servono policy o automazioni separate.

## Cleanup

Registra rimozione degli oggetti temporanei e verifica finale del resource group.

## Rilevanza professionale

Il laboratorio distingue tre livelli spesso confusi: l'**autenticazione** (login riuscito, `az account show` conferma "chi sono"), l'**autorizzazione RBAC** (Reader assegnato al gruppo definisce "cosa può fare quel principal su quello scope", e si combina con eventuali ruoli ereditati come l'Owner sulla sottoscrizione), e il **blocco di governance** (il lock CanNotDelete impedisce un'operazione indipendentemente da chi la richiede o da quali permessi RBAC possiede). Un utente può essere autenticato, avere permessi RBAC sufficienti, e comunque non riuscire a completare un'azione perché un lock la impedisce a livello di scope — sono meccanismi indipendenti e cumulativi.

