# Conan - Installazione

Per installare `conan` i passi da seguire sono i seguenti:

```bash
sudo dnf install python3             # Installare python3 (se non presente sul sistema)
sudo pip3 install conan              # Installare conan
sudo pip3 install conan --upgrade    # Verifica aggiornamenti possibili
conan                                # Eseguire conan come verifica
```

Se non viene trovato è necessario aggiungere al path la directory:

```bash
export PATH="$(python3 -m site --user-base)/bin:$PATH"
```

e tentare nuovamente a digitare `conan`, se neccessario aggiungere questa stringa a `.bashrc` o al `.profile`.

### Custom Setting

Se tutto è andato a buon fine, dobbiamo adesso creare i settings di default che conan dovrà utilizzare.

`Alla prima installazione non saranno presenti`, sarà quindi necessario creare quelli di default che poi personalizzaremo:

```bash
conan config init
```

Creati i settings di default andiamo a creare dei setting particolari (vedi seguito) in uso dentro Elettronica.
Posizionarsi nella cartella di conan

```bash
cd ~/.conan
```

e aprire il file `~/.conan/settings.yml`(che si dovrebbe essere creato prima). Vi sono ora da fare delle modifiche:

1. Aggiungere le `Verisioni di OS`: (circa linea 22) sotto l'opzione `Linux` dentro `os` aggiungere la seguente lista:
   
   ```yml
   distro: [None, RHEL7.3, RHEL8]
   ```
   
   **Note**: RHEL sta per `R`ed `H`at `E`nterprise `L`inux, per motivi storici dello sviluppo `None = RHEL7.1`.

2. Andare alla fine del file, e aggiungere la seguente riga:
   
   ```yml
   qt_version: [5.9, 5.15, 6.2]
   ```

Salvare e chiudere il file, quindi aprire : `~/.conan/profiles/default` (contente le informazioni di default, se non specificate) e aggiungere in fondo al blocco `[settings]` la versione di qt da usare di default sul proprio sistema, usando la seguente stringa come reference:

```
os.distro=RHEL8
qt_version=5.15
```

**Note**: 

- La distro va messa in base all'OS installato sulla propria macchina, Centos8 ad esempio equivale a RHEL8. 

- In caso di più versioni di Qt installate contemporaneamente, fare riferimento alla [guida di installazione multi-Qt ](../00_utility/elt-system-tool-setup.md) per modificare dinamicamente anche la conan setting corrente e mantenerla allineata.

### Creazione di un Token

Per poter usare conan dentro i repository di Gitlab è necessario creare un `Token Gitlab`, in sola lettura, per permettere a conan di accedere al server gitlab di Elt e recuperare i prodotti richiesti (precedentemente deployati):

- Go to elt-gitlab -> User Settings -> Access Tokens -> Create a new access token
- Pick a token name,
- Check flag "read_api"
- Get/Copy <token value>

Salvare il token appena creato aggiungendolo al file `.profile` (o `.bash_profile`, dipende dall'OS) per garantirne una visibilità globale tra i programmi e terminali **<mark>(importante usare questi nomi, comuni a tutti i progetti)</mark>**:

```bash
export CONAN_LOGIN_USERNAME="<Your gitlab user>"
export CONAN_PASSWORD="<Your Password/token>"
```

e successivamente riavviare la macchina per rendere l'impostazione efficace.

Il token appena creato viene usato (ripetendo l'operazione ad ogni riavvio di terminale) durante la login di conan:

```bash
conan user <gitlab_username or deploy token username > -r gitlab -p <personal_access_token or deploy token value>
```

Per automatizzare e semplificare l'ultimo step deldi login, si consiglia di mettere alla fine del proprio `.bashrc` il seguente script:

```bash
# CONAN_USER && CONAN_PASSWORD da definire dentro '.profile' e raggiungibili dopo il riavvio
alias conanlogin="conan user $CONAN_LOGIN_USERNAME -r gitlab -p $CONAN_PASSWORD"
alias conanlogout="conan user -c"
```

Così da poter facilmente eseguire il login a necessità digitando semplicemente `conanlogin` sul terminale

### Https credential setup

Nel lavorare con conan abbiamo che la compilazione dei pacchetti mancati è desiderabile sia il più possibile simile al procedimento che avviene dentro le pipeline di GitLab. Questo procedimento recupera e ricompila le librerie (quando necessario) attraverso `https`.

I nostri computer, per avere un astrazione comoda, e avere che la clone automatica si comporta come la clone attraverso ssh, devono aggiungere dentro la loro cartella `$HOME` un file, chiamato `.git-credentials` che al suo interno contiene una riga scritta così:

```
https://<Gitlab-Username>:<Conan-Gitlab token>@elt-gitlab
```

dove. ovviamente abbiamo:

1. `Gitlab-Username`: Il nome utente con cui ci si collega a conan

2. `Conan-Gitlab token`: Il token generato dentro Gitlab per Conan, e (se salvato seguendo la guida) presente in `.bash_profile`

Se si è seguita la guida, è possibile eseguire questa riga:

```bash
cd $HOME
echo "https://$CONAN_LOGIN_USERNAME:$CONAN_PASSWORD@elt-gitlab" > .git-credentials
cat .git-credentials
git config --global credential.helper store
git config --global credential.helper
```

Per far funzionare questo metodo è però necessario avere installato correttamente sul proprio sistema, e non solo su citrix il certificato di **`elt-gitlab`**, come descritto [nel setup del priprio OS](../00_utility/elt-system-OS-setup.md).

### Connessione ai repository da gestire con conan

A questo punto è opportuno rimuovere il repository centrale di conan (per evitare che il codice da noi deployato possa finire,per sbaglio, fuori dal perimetro della rete di Elt)

```bash
# Rimuovere il repository centrale di conan per evitare che il codice da noi deployato possa finire,per sbaglio, fuori dal perimetro della rete di Elt
conan remote remove conancenter

# Aggiungere come repository conan il server gitlab di Elettronica:
conan remote add gitlab https://elt-gitlab/api/v4/packages/conan False
```

Per verificare la correttezza delle impostazioni basta eseguire:

```bash
conan remote list
```

con un output previsto **SOLO**:

```bash
gitlab: https://elt-gitlab/api/v4/packages/conan [Verify SSL: False]
```

### Verifica finale

Al termine di tutti questi passaggi, per controllare che conan abbia correttamente effettuato l’associazione del repository `elt-gitlab` e sia collegato al proprio utente,  proviamo a digitare i seguenti comandi (dopo aver preventivamente riavviato il terminale):

```bash
conanlogin
conan search EltSpectrogram -r gitlab
```

Con output previsto:

```
$ conanlogin 
  Changed user of remote 'gitlab' from 'None' (anonymous) to 'rina_ealfano'

$ conan search EltSpectrogram -r gitlab
  Existing package recipes:

  EltSpectrogram/2.0.2@HCI+ELT_SPECTROGRAM/stable
  EltSpectrogram/2.0.3@HCI+ELT_SPECTROGRAM/stable
  EltSpectrogram/2.0.4@HCI+ELT_SPECTROGRAM/stable
```

---

## Userful Macro

Altre macro e funzioni utili da inserire nel proprio `.bashrc` sono:

```bash
function conanCreate
{
  conan create $1 $(git config --get remote.origin.url | cut -d ":" -f 2 | sed "s/.git//" | sed "s/\//+/g")/stable
}
```

---

[GO ---> BACK](README.md)
