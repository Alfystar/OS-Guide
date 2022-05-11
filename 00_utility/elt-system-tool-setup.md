# Walkthrough

Summary:

1. [vpn installation](#vpn_installation)
2. [Git Setup](#GitSetup)
3. [Developer tools](#devTools)
4. [QT installation Online](#QTinstallationOnline)
5. [Installazione rabbitmq](#rabbitmq_installation)

---

#### <a name="vpn_installation"> vpn installation </a>

```bash
sudo dnf install epel-release
sudo yum install openconnect NetworkManager-openconnect*
```

- open Settings -> Network -> VPN, click on '+', then under Identity tab, set:
  
  - vpn protocol: 'cisco anyconnect or openconnect'
  - gateway: vpn.elt.it

- then under the ipv4 tab:
  
  - set routes -> 'automatic' to OFF, then add below:
    
    ```
    address        netmask        gateway        metric
    172.16.0.3    255.255.0.0    172.16.0.0    1
    10.180.0.0    255.255.0.0    172.16.0.0    1
    ```
  
  - check option "use this connection only for resources on its network"
  
  - click Add To connect to the vpn, click on the top-right system trail, then VPN, then connect (select group VPN-ENG when logging into vpn)

#### <a name="GitSetup"> Git Setup </a>

```bash
sudo dnf install git git-gui gitk aspell-en
git config --global user.name "Mario Rossi"
git config --global user.email "mariorossi@acme.com"
ssh-keygen         # (this will generate both private and public key files)
```

Then copy and paste the public key (.pub file content) onto your git account settings webpage

- From https://elt-gitlab -> Settings -> SSHKeys
  
  - Paste public key, give it a title, leave blank "expires at" field for "never", click on add key

---

#### <a name="devTools"> Developer tools </a>

**OBBLIGATORI**

Tra i più comuni tools per lo sviluppo suggeriamo di installare:

```bash
sudo dnf groupinstall "Development tools" --with-optional
sudo dnf install mesa-libGL-devel kernel-devel dkms make perl
```

***Facoltativi***

Per poter operare sul file system è sviluppato un tool molto carino su github: [qdirstat: QDirStat - Qt-based directory statistics](https://github.com/shundhammer/qdirstat#building), se interessati clonare il repository compilarlo e installarlo (necessita di aver installato Qt5).

---

Having installed QT creator, you may want to add qmake to your PATH, in order to be able to invoke it from a terminal

- Add the following line to your .bashrc file (~/.bashrc)

- export PATH="[PATH TO QT BIN]:$PATH"    (usually [PATH TO QT BIN] is something like /home/[user]/Qt/5.15.2/gcc_64/bin)
  Then, run:
  
  ```
  source ~/.bashrc
  ```

---

#### <a name="QTinstallationOnline"> QT installation Online </a>

*Pre-requisito, aver installato i [Developer Tools](#devTools) obbligatori*

1. Creare un account Qt : https://login.qt.io/register

2. Scaricare l'installer da : https://www.qt.io/download-qt-installer

3. Rendere il file scaricato eseguibile (`sudo chmod +x <file name>`)

4. Avviare l'eseguibile

5. Loggarsi con le credenziali create prima

6. Scegliere i passi di interesse mettendo le licenze open source e facendo installare anche l'ide, NON far installare Cmake, ninja e altri builder, meglio usare i pacchetti presenti nella distribuzione e quindi globali
   
   **<mark>NOTA di SICUREZZA</mark>**: indipendentemente dalla posizione ove si installa Qt, si sugerisce di <mark>non</mark> scegliere una posizione che necessiti dei permessi di <mark>root</mark>, per evitare brecce di sicurezza, in quel caso creare a mano la cartella da terminale come sudo, e prima di avviare il download, eseguire un `sudo chown` e assegnare utente e gruppo della cartella a se stessi o ad un utente ad-hoc che non abbia i poteri di root.

7. ATTENDERE...

Finita l'installazione è necessario aggiungere al `$PATH` la directory dove trovare `qmake` e gli altri applicativi `Qt`, per fare ciò ci rechiamo dentro la cartella di installazione, di default :`~/Qt/<qt_version>/gcc_64/bin`, e bisogna aggiungere questo `PATH`, si consiglia alla fine del `.bashrc`.

<u>**Nell'eventualtà in cui siano stati installati sulla stessa macchina più versioni**</u> si consiglia invece un approccio dinamico:
faremo passare il `PATH` attraverso una cartella simbolica, posizionata dentro `.local` e aggiungeremo nel `.bashrc` dei comandi (tramite alias) per spostare il link simbolico alla versione di interesse.
A titolo di esempio ecco i comandi cosa aggiungere al `.bashrc` se si è installato: `Qt5.9`, `Qt5.15`, `Qt6.2`:

```bash
###############################
### Variation of qt version ###
###############################
alias qtEnable5_9='unlink ~/.local/qtVersion 2>/dev/null ; ln -s  ~/Qt/5.9.9/  ~/.local/qtVersion ; conan profile update settings.qt_version=5.9 default'
alias qtEnable5_15='unlink ~/.local/qtVersion 2>/dev/null ; ln -s  ~/Qt/5.15.2/  ~/.local/qtVersion ; conan profile update settings.qt_version=5.15 default'
alias qtEnable6_2='unlink ~/.local/qtVersion 2>/dev/null ; ln -s  ~/Qt/6.2.2/  ~/.local/qtVersion ; conan profile update settings.qt_version=6.2 default'
alias qtVersionSelect='readlink ~/.local/qtVersion'
PATH="$HOME/.local/qtVersion/gcc_64/bin:$PATH"
```

Avremo così un impostazione permanente nel tempo, non cambierà fino alla prossima modifica, e se faremo riferimento per tutti i path da aggiungere a questo file simbolico, potremmo mantenere allineati i terminali e il sistema. 

---

#### <a name="rabbitmq_installation"> Installazione rabbitmq </a>

**Nota 15/12/21**: a causa della versione limitata di `erlang` presente nei repository di centos 8, la guida descrive come installare `rabbitmq 3.8.8` per essere allineato a `erlang 22.0.7`,  verificare in futuro sul [sito di Rabbit le nuove compatibilità](https://www.rabbitmq.com/which-erlang.html) in base alla versione ultima presente sui repository.

Guida di riferimento : [How To Install RabbitMQ on RHEL 8 / CentOS 8 | ComputingForGeeks](https://computingforgeeks.com/how-to-install-rabbitmq-on-rhel-8/)

```bash
sudo dnf install epel-release erlang-22.0.7-1.el8
curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
sudo dnf makecache -y --disablerepo='*' --enablerepo='rabbitmq_rabbitmq-server'
sudo dnf install rabbitmq-server-3.8.8-1.el8
```

Avendo fissato una versione antecedente per rabbit `dnf` ci ricorderà costantemente che non può aggioranre il pacchetto per colpa di erlang, per questo motivo possiamo rimuovere questo fastidioso messaggio andando ad aggiungere alla fine di `/etc/dnf/dnf.conf` : `exclude=rabbitmq-server-3.8.8-1.el8.noarch`. **FILE DA MODIFICARE COME SUDO, ATTENZIONE**

Siamo ora pronti per abilitare il server rabbitmq attraverso il suo servizio di sitema: 

```bash
sudo systemctl enable --now rabbitmq-server
# check if it's active...
sudo systemctl status rabbitmq-server
```

Installeremo ora `lvc plugin` dentro `/usr/lib/rabbitmq/lib/rabbitmq_server*/plugins/` (il pacchetto `rabbitmq_lvc_exchange-3.8.0.ez` è scaricato da https://github.com/rabbitmq/rabbitmq-lvc-exchange/releases/tag/v3.8.0 da terminale attraverso `wget`)

```bash
cd /usr/lib/rabbitmq/lib/rabbitmq_server*/plugins/ # capisce in automatico la versione installata
sudo wget https://github.com/rabbitmq/rabbitmq-lvc-exchange/releases/download/v3.8.0/rabbitmq_lvc_exchange-3.8.0.ez
sudo rabbitmq-plugins list    # Se tutto è andato bene dovremmo trovare rabbitmq_lvc_exchange nella lista
sudo rabbitmq-plugins enable rabbitmq_lvc_exchange
sudo rabbitmq-plugins enable rabbitmq_management # enable Web interface su http://localhost:15672/ 
sudo systemctl restart rabbitmq-server
```

L'interfacci web di `rabbitmq` è raggiungibile tramite browser e si trova a http://localhost:15672/ 

- Ip := `localhost`
- Porta := `15672`
- Utente := `guest`
- Pw := `guest`

---

Writed by: Giovanni Martino
Updated by: Alfystar (Emanuele Alfano)

[`GO ---> BACK`](README.md)
