# Setup New Project

In questa guida sono presenti alcuni suggerimenti sul come creare un repository di qualità all'inizio di un progetto.

### Setup .gitignore

Prima azione da fare è impostare correttamente il file `.gitignore`, [qui è possibile trovare un esempio abbastanza generale](./general-GitIgnore.txt).

### Submodule import

Dopo aver aggiunto un sotto modulo, nella ROOT directory o dentro una cartella chiamata `01_subModule` nella ROOT ditectory del repository, collegarla dove serve **ATTRAVERSO dei link simbolici** in cui non si punta solo il file di interesse, ma tutta la libreria, questo per evitare problemi di link simbolici relativi, contenuti a loro volta all'interno del sotto modulo.

# Note di porting da Qt5 a Qt6

È possibile trovare una lista di passi da fare per ottenere un porting dalla versione attuale, alla nuova versione di progetto usando Qt5/6 e Cmake+conan [qui](portingQt5-6Note.md)

Per avere un idea dei problemi riscontrabili nel cambio di versione da Qt5 a Qt6, consultare le note di porting per [Ipc-porting](Note_porting_da_qt5_a_qt6_di_IPC.md).

---

[GO ---> BACK](../README.md)
