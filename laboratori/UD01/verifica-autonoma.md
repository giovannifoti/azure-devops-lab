# Verifica autonoma dell'ambiente — UD01

La verifica è stata eseguita direttamente nel Terminale di macOS. La versione del sistema operativo è stata controllata con `sw_vers` e l'architettura con `uname -m`: la postazione utilizza macOS 26.6.2 su architettura Apple Silicon `arm64`. Il repository personale è conservato in `~/workspace/azure-devops-lab`; il comando `git rev-parse --show-toplevel` permette di verificare che il terminale si trovi effettivamente nella radice del repository corretto.

Il remote è stato controllato tramite `git remote -v` e punta al repository GitHub personale `https://github.com/giovannifoti/azure-devops-lab.git` tramite HTTPS. Nel workflow Git, il working tree contiene i file e le modifiche locali, mentre la staging area contiene soltanto le modifiche selezionate con `git add` per il prossimo commit. `git commit` registra queste modifiche nella cronologia locale e `git push` pubblica il commit sul repository remoto GitHub.

Azure CLI è stata verificata tramite `az version` e risulta installata nella versione 2.90.0. La sottoscrizione Azure è stata controllata tramite `az account show`, risultando attiva, abilitata e impostata come predefinita. L'invito al docente nel repository GitHub risulta accettato.

Un possibile errore di contesto consiste nell'eseguire i comandi Git dalla cartella sbagliata. Il comando `git rev-parse --show-toplevel` permette di riconoscere rapidamente il problema; se restituisce `not a git repository`, è possibile individuare i repository presenti nel workspace con `find ~/workspace -maxdepth 3 -type d -name .git -print` e quindi raggiungere la directory corretta.
