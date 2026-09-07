# Verifica autonoma — UD01

La distribuzione Ubuntu-24.04 richiesta per il corso risulta in esecuzione su WSL 2, come confermato dal comando `wsl --list --verbose` eseguito in PowerShell, che mostra il valore `2` nella colonna VERSION accanto al nome della distribuzione.

Il repository personale `azure-devops-lab` si trova nel filesystem Linux, al percorso `/home/giorg/workspace/azure-devops-lab`. Ho verificato che questo coincida con la radice del repository confrontando l'output di `pwd` con quello di `git rev-parse --show-toplevel`: entrambi restituiscono lo stesso percorso, confermando che non sto lavorando per errore in una sottocartella o in un percorso Windows montato (`/mnt/c`). Il remote configurato, verificato con `git remote -v`, punta a `https://github.com/giorgias18/azure-devops-lab.git`, sia per fetch che per push.

Il **working tree** è la copia dei file così come si trovano sul disco, con le modifiche non ancora segnalate a Git; la **staging area** (o index) contiene i file selezionati con `git add`, pronti per essere inclusi nel prossimo commit; il **commit locale** è uno snapshot permanente della staging area salvato nella cronologia del repository sul proprio computer; il **repository remoto** su GitHub riceve questi commit soltanto dopo un `git push` esplicito.

Il comando utilizzato per controllare la presenza e il funzionamento di Azure CLI è `az version`, che ha restituito la versione installata (2.90.0), mentre l'accesso effettivo al tenant è stato confermato con `az account show --output table`, il cui esito positivo indica che l'autenticazione con `az login --use-device-code` era andata a buon fine.

Lo stato dell'invito al docente come collaboratore del repository è stato verificato in **Settings → Collaborators**, dove risulta attualmente come collaboratore attivo (invito accettato).

Un possibile errore di contesto frequente consisterebbe nel lavorare, senza accorgersene, in una cartella del filesystem Windows montato in WSL (percorsi che iniziano con `/mnt/c/...`) invece che nella home Linux. Questo errore si riconosce eseguendo `pwd`: se il percorso restituito inizia con `/mnt/c`, si sta operando fuori dal filesystem Linux, con possibili rallentamenti e comportamenti anomali di Git; il rimedio è spostarsi con `cd ~/workspace/...` verso la cartella di lavoro corretta.


## Autovalutazione

| Capacità | Valutazione |
|---|---|
| Distinguo PowerShell dal terminale Ubuntu | `Completato` |
| Verifico che Ubuntu utilizzi WSL 2 | `Completato` |
| Riconosco la radice del repository Git | `Completato`|
| Verifico il remote prima del push | `Completato` |
| Distinguo file non tracciato e file in staging | `Completato` |
| Inserisco in staging soltanto il file richiesto | `Completato` |
| Verifico lo stesso commit in locale e su GitHub | `Completato` |
| Verifico lo stato dell'invito al docente | `Completato` |
| Riconosco ed escludo dati riservati | `Completato` |