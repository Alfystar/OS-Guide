# Cool Shell

Su linux la shell di comando è impostata dalla variabile di ambiente `PS1` , essa comunica al teminale cosa far vedere (e come) ogni nuova linea del terminale.

Per rendere il terminale TOTALMENTE universale (presente sia per user che per root) inserire tutti i file e gli script qui riportati dentro `/etc/` e `/etc/.bashrc`, questo perchè su centos, il `.bashrc` di default di ogni utente vede se è presente (e in tal caso lo include) un bash di default comune a tutti.

Se si preferisce però lasciare il root *vanilla* posizionarsi sulla propria `$home` e modificare solo quei file.

#### Passo preliminare

Copiare il file `.bash_setupConstans.sh` nella propria `$home`/`/etc/`, così da impostare le variabili usate dagli script proposti di seguito.

## Scrivere Nuovi terminali

Se si volessero aggiungere nuovi terminali, è necessario tenere a mente che tutti colori, all'interno della definizione di `PS1` devono essere racchiusi tra `\[` e `\]` per aiutare bash a riconoscere che non sono caratteri printabili e quindi mantenere il corretto calcolo della lunghezza di stringa.

`\[` e `\]` **<u>non devono</u>** al contrario essere usati se si sta usando una funzione (vedi `colorBranch` come esempio) per dare colore al testo, bensi è opportuno usare `printf` o `echo -e <stringa>`.

## Personalizzare il terminale

Se si vuole provare queste varianti posizionarsi in fondo al proprio .bash_aliases per sovrascrivere la `PS1` impostata (sarà così sempre possibile annullare le modifiche semplicemente commentando le righe).

A questo punto scegliete una di queste alternative, quella che più vi aggrada:

---

### {Utente}@{gruppo} {Path} {Color Branch} $

Un esempio di terminale è:

```
laboratorio@elt-16005596lx ~/Scrivania/eltProject/conan/00_utility (master)$ 
```

E si realizza mettendo in fondo a :

```bash
###################################################
############ TERMINAL INFORMATION SET #############
###################################################
# Shell form: {Utente}@{gruppo} {Path} {Branch} $
source /usr/share/git-core/contrib/completion/git-prompt.sh
source ~/.bash_setupConstans.sh
export PS1="\[$BGreen\]$User@$Group \[$BYellow\]$PathShort\[$BCyan\]\[\$(colorBranch)\]\$(__git_ps1)\[$Color_Off\] \$ "
```

---

### {System-time} {Color Branch} {Path} $

Questa modalità ha varie impostazioni per comunicare lo stato del priprio repositori.

Un esempio di terminale è:

```
12:44:52 {master} ~/Scrivania/eltProject/conan/00_utility$
```

E si realizza mettendo:

```bash
###################################################
############ TERMINAL INFORMATION SET #############
###################################################
# Shell form: {System-time} {Branch} {Path} $
source /usr/share/git-core/contrib/completion/git-prompt.sh
source ~/.bash_setupConstans.sh
export PS1="\[$BIBlue\]$Time12h \[$BYellow\]$PathShort\[\$(colorBranch)\]\$(__git_ps1)\[$Color_Off\] \$ "
```

---

### {System-time} {Path} {Color Branch} \n $

Questa modalità ha varie impostazioni per comunicare lo stato del priprio repositori.

Un esempio di terminale è:

```
04:20:20 ~/Documents/eltProjects/conan (master)
└─ $ ▶ 
```

E si realizza mettendo:

```bash
###################################################
############ TERMINAL INFORMATION SET #############
###################################################
# Shell form: {System-time} {Branch} {Path} $
source /usr/share/git-core/contrib/completion/git-prompt.sh
source ~/.bash_setupConstans.sh
PS1="\[$BIBlue\]$Time12h \[$BYellow\]$PathShort\[\$(colorBranch)\]\$(__git_ps1)\n\[$Green\]└─▶\[$Color_Off\] \$ "
```

---

Etc...

[GO ---> BACK](README.md)
