# Evidenza — Verifica autonoma dell'ambiente UD01

## Contesto verificato

La verifica autonoma è stata eseguita direttamente nel Terminale di macOS. La postazione utilizza macOS 26.6.2, build 25G83, su architettura Apple Silicon `arm64`.

Il repository personale è conservato in `~/workspace/azure-devops-lab`. Il comando `git rev-parse --show-toplevel` è stato utilizzato per verificare che il terminale fosse posizionato all'interno della radice corretta del repository.

Il remote è stato controllato tramite `git remote -v` e punta al repository GitHub personale:

```text
https://github.com/giovannifoti/azure-devops-lab.git
```

L'invito al docente risulta accettato.

## Strumenti verificati

| Componente | Versione o stato | Controllo eseguito |
|---|---|---|
| macOS | 26.6.2 — build 25G83 | `sw_vers` |
| Architettura | arm64 | `uname -m` |
| Git | 2.50.1 (Apple Git-155) | `git --version` |
| Visual Studio Code | 1.134.0 — arm64 | `code --version` |
| GitHub CLI | 2.100.0 | `gh --version` |
| GitHub | autenticazione HTTPS attiva | `gh auth status` |
| Azure CLI | 2.90.0 | `az version` |
| Azure | sottoscrizione attiva, `Enabled`, predefinita | `az account show` |

## Verifica del flusso Git

Durante la prova è stata verificata la differenza tra i principali stati del lavoro in Git. Il working tree contiene i file e le modifiche presenti localmente. Con `git add` viene selezionato soltanto il file che deve entrare nel prossimo commit e la modifica passa nella staging area. `git diff --cached` permette di controllare il contenuto preparato prima del commit. `git commit` registra le modifiche nella cronologia locale, mentre `git push` invia il commit al repository remoto GitHub.

Per riconoscere un eventuale errore di contesto viene utilizzato:

```bash
git rev-parse --show-toplevel
```

Se il comando restituisce `not a git repository`, i repository presenti nel workspace possono essere individuati con:

```bash
find ~/workspace -maxdepth 3 -type d -name .git -print
```

## File prodotti

Per la verifica autonoma sono stati predisposti:

```text
laboratori/UD01/verifica-autonoma.md
evidenze/UD01-verifica-autonoma.md
```

## Controllo sicurezza

- [x] Non sono presenti password, token, chiavi o codici temporanei.
- [x] Non sono presenti e-mail personali non necessarie.
- [x] Non sono presenti Subscription ID o Tenant ID.
- [x] Il remote GitHub non contiene token o parametri riservati.
- [x] Gli output riportati sono limitati alle informazioni tecniche utili.

## Esito

La verifica autonoma ha confermato il corretto contesto di lavoro, il collegamento al repository personale, il funzionamento del flusso Git, l'accesso a GitHub e la disponibilità di Azure CLI con sottoscrizione attiva.
