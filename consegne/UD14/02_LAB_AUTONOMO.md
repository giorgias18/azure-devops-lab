# UD14 — Consegna LAB autonomo

- branch: feature/ud14-ci-v2
- modifica app: APP_VERSION in server.py cambiata da "ci-v1" a "ci-v2"
- test fallito: test_health_version (AssertionError: 'ci-v2' != 'ci-v1')
- causa: il test verificava ancora il valore precedente ("ci-v1"), non più coerente con la nuova versione dell'app
- modifica test: in test_backend.py, assertEqual aggiornato da "ci-v1" a "ci-v2"
- test finale: OK (3 test eseguiti, tutti passati)
- PR: aperta da feature/ud14-ci-v2 verso main, con i due commit (bump versione app + allineamento test)
- merge: completato su main
- run CI: eseguita automaticamente dopo il merge (Test → BuildPush, entrambi su Microsoft-hosted, UD14_AGENT_MODE=MICROSOFT_HOSTED)
- nuovo tag ACR: 9
- cleanup eseguito: NO