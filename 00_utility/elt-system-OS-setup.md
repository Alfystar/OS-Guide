# OS setup Walkthrogh

Questa guida descrive come mettere in funzione un sistema Centos 8.

In oltre è possibile installare il desktop enviroment KDE

1. [Centos 8 installation](#Centos8installation)
2. [Installare KDE](#KDEinstall)
3. [Installazione certificato SSH (per HTTPS) su Centos](#certInstall)

## <a name="Centos8installation"> Centos 8 installation </a>

Instructions may vary depending on what you want to do. The following is for installing centOS 8 in multi-boot using physical partitions

- download OS image from centos.org (CentOS-8.3.2011-x86_64-dvd1.iso)
- make a Centos 8 bootable usb drive, for example with Win32 Disk Imager under Windows, or with dd under linux ([HowTos/InstallFromUSBkey - CentOS Wiki](https://wiki.centos.org/HowTos/InstallFromUSBkey))
- create a new partition for Centos
- enable boot menu in BIOS, reboot from Centos USB drive, install OS on partition space
   (NB: you may need to configure a custom partition scheme, depending on your bootloader...)

## <a name="KDEinstall"> Installare KDE </a>

 Per installare kde, partendo da gnome, aprire il terminale e digitare:

```bash
sudo dnf install epel-release PowerTools
sudo dnf config-manager --set-enabled PowerTools
sudo dnf groupinstall "KDE Plasma Workspaces"
sudo dnf group install "base-x"
```

Procediamo ora ad abilitare di default come `login manager` quello di default di KDE: `sddm`.

```bash
sudo systemctl disable display-manager.service
sudo systemctl enable sddm.service
```

Terminata l'installazione possiamo riavviare la macchina o fare il <mark>log-out</mark> dall'utente per tornare alla schermata di login, se non si è avviata quella di KDE, selezionare il proprio utente e prima di mettere la password andare sulle impostazioni (ingranaggio) e scegliere `Plasma(X11)`.

Una volta loggati è possibile andare sulle impostazioni KDE e impostare di default Plasma andando:

1. Cercare `login screen`
2. In basso circa al centro andare su `Behaviur ...`
3. Impostare per il proprio utente come sessione di default `Plasma(X11)`.
   Arrivati qui avremo il nostro desktop enviroment pronto, per verificarne il corretto funzionamento riavviare la macchina.

### Installare i tools base di KDE

Per rendere la macchina più KDE ready è consigliato installare i seguenti tools:

```bash
sudo dnf install ark p7zip p7zip-plugins htop bzip2 tree
```

Nell'ultima versione `konsole` ha emulato con successo molti comportamenti di `terminator`, si suggerisce al lettore di provare per un periodo le novità prima di modificare il terminale di riferimeto.

### Terminal Setup

Il teminale di default non è molto utile, ed è privo della maggior parte dei comandi/alias/setting per rendersi produttivi, creare dentro `$HOME` un file denominato `.bash_aliases` e aggiungere alla fine di `.bashrc`:

```bash
# User specific aliases and functions
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
```

Popolare quindi il file `.bash_aliases` con i [suggerimenti reperibili qui](terminal-tips/README.md) così da avere un setup completo per iniziare a lavorare.

## <a name="certInstall"> Installazione certificato SSH (per HTTPS) su Centos  </a>

Sul prorio PC nuovo l'IT dovrebbe aver messo in `Download` l'ultimo certificato valido per accedere alla intranet di Elettronica, in caso contrario chiedere a un collega l'ultima versione.
Questo certificato è indispensabile per poter usare senza errori o warning le connessioni alla intranet, di cui un esempio è il clone dei repository attraverso HTTPS.

I comandi da eseguire sul proprio terminale sono i seguienti:
**PRE-REQUISITO**: Trovarsi dentro la cartella con l'ultimo certificato valido 

```bash
sudo cp CA_64.cer /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
```

Per testare, semplicemente clonare attraverso HTTPS un repository a piacere dal server GitLab.

---

[`GO ---> BACK`](README.md)
