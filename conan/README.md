# Conan Vademecum

Guida completa reperibile sul sito [ufficiale](https://docs.conan.io/), da cui una [Cheetsheet è reperibile qui](https://docs.conan.io/en/lates/cheetsheet.html).
Indice:

1. [Installazione Conan](conan_installazione.md)
2. [Creazione Ricetta](conan_scrivereRicette.md)
3. [Installare le dipendenze per Consumer (lavoro locale)](#dip-Install-Consumer)
4. [Creare pacchetti Conan](#pack-create)
5. [Ripulire l'enviroment](#envClean)

Va precisato che in conan esistono 2 paradigmi principali:

- `Consumer`
- `Creator`
  I primi usano conan per reperire i binari mancanti e successivamente si procede senza altro con la compilazione dei pacchetti.
  I secondi generano i pacchetti per i primi.

La distinzione non è così netta e possono esistere progetti `Creator` che usano dei pacchetti, risalendo la catena delle dipendenze, e che quindi, prima di essere uplodati si comportano come Consumer.

## <a name="dip-Install-Consumer"> Installare le dipendenze per Consumer (lavoro locale) </a>

Il processo di installazione delle dipendenze prevede che `conan` legga la ricetta (`conanfile.txt`/`conanfile.py`) e provveda a scaricare in locale ( “cache conan”, path `$HOME/.conan/data`) dal remote `conan` le librerie (specificate nella sezione **`[requires]`**) allineate alle versioni riportate nella ricetta e relative alla configurazione del sistema in uso.

Se non presenti in remoto, provvederà prima alla loro compilazione utilizzando l’environment locale ed in seguito a caricare nella cache locale il relativo pacchetto compilato.
Coerentemente a quanto specificato nella sezione **`[generators]`**, verrà generato un file `conanbuildinfo.pri`/`conanbuildinfo.cmake` che definisce le variabili del qmake/cmake e permetterà ai generatori di conoscere i path e i nomi delle librerie che si useranno nella compilazione.

<details>
<summary> <i> Procetura per qmake (Vecchia e sconsigliabile) </i>  </summary>

  Operativamente occorre percorrere i seguenti step, posizionandosi dentro `extResources`  (e avendo precedentemente popolato il `extResources.pri` come visto nel [capitolo della scrittura delle ricette per qmake](conan_scrivereRicette.md))

```bash
conanLogin # se precedentemente non chiamato
conan install ..
```

**L'uso di questa modalità è adatta a un uso `Consumer` e rende problematico usare il codice per creare pacchetti `Creator`.**

</details>

<details>
<summary> <b> Procetura per cmake (CONSIGLIATA) </b>  </summary>

  Per usare questa procedura il/i cmake che richiedono l'uso di conan si DEVONO ASPETTARE che il `conanbuildinfo.cmake` si trovi dentro la cartella di compilazione, a codice l'include deve essere:

```cmake
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)    # Conan setting recuperato dinamicamente
```

  Con la presenza di questo path dinamico, si rende il CMakefile in grado di essere usato con successo tanto per le fasi del progetto in cui il pacchetto è `Consumer` che nella fase in cui diventi eventualmente `Creator`.

  Facendo così, è possibile posizionare la roba di `conan` dentro la directory in cui si vuole compilare con cmake, decisa da noi nel caso `Consumer` e scelta da `conan` all'interno della sua cache della fase `Creator`.

```bash
cd <path_CMakeLists.txt_toplevel>
mkdir -p build
cd build
conan install <path/conanfile.py>
cmake ..
# ...
```

</details>

### Output di conan

Nella cartella di install di conan verranno generati i seguenti files:

1. `conanbuildinfo.txt` – file contenente le istruzioni necessarie a conan per il build delle
   librerie
2. `conaninfo.txt` – file contenente le specifiche ed i settings del nostro environment di
   compilazione
3. `graph_info.json` – file contenente il grafico delle dipendense di versione del nostro
   applicativo
4. `conanbuildinfo.pri/cmake` – file contenente le istruzioni necessarie al ‘generator’, per poter compilare le librerie richieste
5. `conan.lock` – file ad uso esclusivo di conan per l’utilizzo sincronizzato dei file sopra citati

A questo punto siamo pronti a eseguire il qmake/cmake in base al  progetto corrente.

> **Qt Creator steps** : eseguire il `run Qmake` per linkare il `conanbuildinfo.pri` e successuvamente buildare.

> **Terminal steps qmake** : posizionarsi al livello del `*.pro` e lanciare prima `qmake` e poi `make`

> **Terminal steps Cmake** : posizionarsi al livello del top-cmakelists e procedere come dice la [guida di cmake](../cmake/README.md)

## <a name="pack-create"> Creare pacchetti Conan </a>

La creazione di pacchetti di `conan` coinsiste nel creare all'interno della propria cache i sorgenti, compilarli (tipicamente con cmake) e successivamente, se configurato caricarli dentro un repository online.

In elettronica la parte di *upload* è eseguita solo dentro i doker, dove la compilazione è eseguita dentro delle macchine virtuali.

Per poter essere sicuri che però la compilazione proceda senza problemi nelle configurazioni (è quindi un test preliminare) si può provare a creare un pacchetto in locale, il quale è comunque visibile ai progetti presenti sul proprio computer.

Per compilare un pacchetto è suffuciente posizionarsi nella home del progetto e lanciare a terminale:

```bash
conan create <path/conanfile.py>
```

In caso vi siano delle aree di codice che non si vogliono raggiungere durante la creazione di pacchetti (test/esempi, etc...) è consigliabile passare dal `conanfile.py` le impostazioni al cmake, e dentro il cmake tramite if tenerne conto ([vedi creazione ricette attraverso dal `conanfile.py` per maggiori info](conan_scrivereRicette.md)).

## <a name="envClean"> Ripulire l'enviroment </a>

Per pulire dopo una compilazione si può fare una pulizia rapida cancellando i file creati da conan per la compilazione, che contengono tutti i link per il corretto linking

```bash
cd <extResources>
rm conan* graph_info.json Find*
```

O si può optare per una pulizzia più profonda andando a cancellare i compulati presenti dentro la propria cartella di `.conan`

```bash
cd ~/.conan/data
# rm del pacchetto/i che si vogliono eliminare dalla cache
```

---

Scritto da:

- Alessandro Della Seta
- Alfystar (Emanuele Alfano)

[`GO ---> HOME`](../README.md)
