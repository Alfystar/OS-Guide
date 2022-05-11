# QtCreator e Cmake

Per importare un progetto basato su cmake è sufficiente parire il top-level `CMakeLists.txt`.

Eseguita questa operazione, inizierà l'esecuzione del cmake, che se arriverà a completamento porterà all'apertura con successo del progetto in cmake.

## Problema con "conanbuildinfo.cmake"

In vari progetti di Elettronica, è presente il gestore di pacchetti `conan` ([vedi guida](../conan/README.md)), esso impone delle scelte che concludono con la generazione del file `conanbuildinfo.cmake`, indispensabile ai progetti che lo usano.

Se il progetto in questione lo necessita è sufficiente seguire i seguenti passi per risolvere il problema:

1. Andare su `Projects`

2. Selezionare la configurazione di compilazione che si vuole usare

3. Modificare la `Build directory` in un path di una directory nota (si consiglia di creare una cartella `build` al livello del root del progetto e aggiungere la cartella al `.gitignore` se non presente precedentemente)

4. Andare dentro la cartella `build` ed eseguire il comando `conan install <path/conanfile.py>` ([vedi guida](../conan/README.md)).

5. Tornare dentro QtCreator e rilanciare la compilazione con cmake.

**NOTA BENE**: è necessario cambiare il *build path* per tutte le configurazioni (Debug/Relase/...) e tutti i compilatori

## Interface Libray

Per le librerie composte da solo header, l'editor posiziona questi file tutti dentro la cartella `<Headers>`.

---

[GO ---> BACK](../README.md)
