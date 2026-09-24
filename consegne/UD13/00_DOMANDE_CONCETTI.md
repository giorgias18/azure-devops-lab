# UD13 — Domande concetti

## 1. Perché lo state locale è accettabile in UD12 ma problematico in un team?
**In UD12 ogni partecipante lavorava da solo, le risorse erano temporanee e non c'erano esecuzioni concorrenti. In un team, uno state salvato solo sul computer di una persona crea rischi: non è chiaro chi possiede lo state corretto, due persone potrebbero applicare modifiche contemporaneamente e sovrascriverlo, e non è protetto.**

## 2. A che cosa serve un backend remoto?
**Permette di conservare lo state Terraform in una posizione condivisa e protetta (es. Azure Storage), con meccanismi di locking, invece che solo sul disco di un singolo utente.**

## 3. Distingui stage, job e step.
**La gerarchia è: Pipeline, Stage, Job, Step. Lo Stage raggruppa una fase logica (es. Validate, Deploy, Test); il Job è un insieme di step eseguito da un agent; lo Step è una singola operazione (checkout, script, task).**

## 4. Che cosa significa `checkout: self`?
**Significa "recupera il repository che contiene la pipeline". Il repository viene collocato nell'area di lavoro dell'agent, così i comandi pipeline possono usare percorsi relativi al repository.** 

## 5. Perché una pipeline self-hosted non deve assumere di trovarsi in `~/workspace/...`?
**Perché quel percorso è personale e legato al lavoro interattivo del singolo partecipante, non all'esecuzione della pipeline. La pipeline deve usare variabili predefinite come $(Build.SourcesDirectory), che riflettono dove Azure Pipelines ha effettivamente fatto il checkout, per essere portabile e non dipendere da configurazioni locali.**

## 6. A che cosa serve una service connection?
**Rappresenta l'identità autorizzata con cui la pipeline opera su Azure (scope Azure), evitando che la pipeline riutilizzi automaticamente il login personale del partecipante.**

## 7. Perché preferire Workload Identity Federation a un client secret?
**Con WIF, Azure DevOps ottiene un token federato tramite Microsoft Entra senza dover memorizzare password, client secret o PAT nella pipeline, quindi elimina un segreto statico da gestire e proteggere.**

## 8. Perché limitiamo la service connection a un Resource Group?
**Per applicare il principio del minimo privilegio: la service connection può operare solo sul Resource Group dedicato (`rg-ud13-15-delivery`), non sull'intera subscription.**

## 9. Perché Terraform viene validato ma non applicato dalla pipeline UD13?
**Perché introdurre un backend remoto completo (storage, accessi, autenticazione, locking, bootstrap) richiede tempo e attenzione che non è ancora oggetto di UD13. Il primo deployment IaC automatizzato reale è affidato a Bicep; Terraform in pipeline resta a livello di `fmt`, init -backend=false e `validate`.**

## 10. Perché l'ACR creato in UD13 non deve essere eliminato?
**Perché verrà riusato: in UD14 (CI) si farà build e push dell'immagine nello stesso ACR, e in UD15 (CD) Container Apps farà il pull di quella stessa immagine. Eliminarlo romperebbe la progressione tra le tre UD.**

## 11. A che cosa serve `trigger: none`?
**Serve a impedire che la pipeline parta automaticamente (ad es. a ogni push); permette di eseguirla solo manualmente, così da poterla prima comprendere.**

## 12. Qual è un ordine razionale per diagnosticare una pipeline che non parte correttamente?
**La pipeline è partita? → il job ha trovato un agent? → il checkout è riuscito? → il path è corretto? → il tool è disponibile? → la service connection è autorizzata? → i permessi Azure sono sufficienti? Questo ordine evita di confondere errori di tipo diverso (YAML, agent, file, autenticazione, Azure).**

## 13. Perché l'ACR UD13–UD15 fissa esplicitamente `LegacyRegistryPermissions`?
**Perché ACR supporta anche il modello RBAC+ABAC, in cui i ruoli legacy come AcrPull/AcrPush non funzionano allo stesso modo. Fissare esplicitamente il modello classico evita che le attività successive dipendano da un valore predefinito del servizio che potrebbe cambiare nel tempo.**

## 14. Perché UD13 usa deliberatamente il self-hosted Agent come prima pipeline?
**Per rendere visibile il collegamento diretto tra il tool installato nel WSL2, le capability/PATH dell'Agent, e il comando eseguito dal Job, un legame che con un agent Microsoft-hosted sarebbe meno evidente.**

## 15. A che cosa serve `workspace: clean: all` su un self-hosted Agent?
**Chiede all'Agent di pulire l'intero Pipeline.Workspace prima del Job, evitando che vecchie directory, cache o checkout precedenti facciano sembrare corretta una pipeline che in realtà dipende da residui locali.**

## 16. Distingui `workspace: clean: all` e `checkout: self, clean: true`.
**workspace: clean: all pulisce l'area di lavoro del Job sull'Agent (tutto lo spazio); checkout: self, clean: true pulisce specificamente la copia Git del repository prima del fetch. Sono due pulizie a livelli diversi.**

## 17. Perché la pipeline non deve usare percorsi come `~/workspace/azure-devops-lab`?
**Perché è un percorso personale legato al lavoro interattivo del partecipante, mentre la pipeline deve basarsi sul repository come effettivamente collocato dal checkout ($(Build.SourcesDirectory)), per non dipendere da configurazioni di una macchina specifica.**

## 18. Che cosa cambierebbe passando da `pool-ud09-wsl` a `vmImage: ubuntu-latest`?
**Cambierebbe l'ambiente di esecuzione: con vmImage: ubuntu-latest (Microsoft-hosted) Azure Pipelines assegnerebbe una VM nuova e temporanea a ogni Job, mentre con il self-hosted (pool-ud09-wsl) tool e filesystem persistono sulla macchina del partecipante. La logica della pipeline (checkout → terraform → Bicep → Azure) resterebbe simile.**

## 19. Perché la prima pipeline GitHub usa la Azure Pipelines GitHub App invece di creare un PAT manuale?
**Perché evita di basare la CI sull'identità personale tramite OAuth o su un PAT manuale, e consente di limitare l'accesso al solo repository realmente necessario.**

## 20. In produzione, che cosa rappresenterebbe `pool-ud09-wsl` rispetto a un vero Agent Pool aziendale?
**Rappresenterebbe uno dei nodi di un pool più ampio (es. pool-linux-build con più agent come build-agent-01, -02, -03): il WSL2 personale simula uno dei nodi del Pool, mentre in produzione il Pool identifica un gruppo di esecutori ammessi, non un singolo computer.**

# 21. Perché un Pool con tre Agent non garantisce da solo tre Job paralleli?
**Perché ogni Agent esegue un Job alla volta, quindi tre Agent consentono potenzialmente più Job contemporanei, ma il parallelismo reale dipende anche dalla capacità di Parallel Jobs effettivamente disponibile per l'organizzazione.**
