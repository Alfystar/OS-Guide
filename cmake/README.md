# Cmake - Guida per i progetti

In questa guida vedremo brevemente le cose più importanti da sapere per poter leggere e scrivere un cmake.

1. [Cmake - a cosa serve](#Cmake_a_cosa_serve)
2. Cmake - Breve guida introduttiva
   1. [Cmake - sintassi base](#Cmake_sintassi_base)
   2. [Note preliminari](#Note_preliminari)
   3. [Cmake - design del progetto](#Cmake_design_progetto)
3. [Cmake - Compilare un progetto da terminale](#Cmake_Compilare)

Per avere maggiori informazioni su come scrivere un buon `cmake` di base guardare:

1. [Modelli CMakelists.txt](cmake_bestPractice.md)
2. [Integrazione Cmake-Qt ](cmake_qt.md)
3. [Integrazione Cmake-Conan](cmake_conan.md)

In caso non si conosca la sintassi cmake, si consiglia di guardare le seguenti pagine:

- https://campana.vi.it/blog/introduzione-a-cmake/

- https://riptutorial.com/Download/cmake-it.pdf

- https://cliutils.gitlab.io/modern-cmake/

- https://leimao.github.io/blog/CMake-Public-Private-Interface/

# Cmake - Breve guida introduttiva

## <a name="Cmake_a_cosa_serve"> Cmake - a cosa serve </a>

Attraverso Cmake è possibile automatizzare la creazione dei `makefile` e quindi la compilazione dei progetti.

Il vantaggio di usare il cmake è che è estremamente flessibile e in continuo sviluppo, e il linguaggio con cui lo si può programmare (il cmake appunto) è molto flessibile, ai livelli di un linguaggio di programmazione.

## <a name="Note_preliminari"> Note preliminari </a>

Nell'approcciare il linguaggio di Cmake è utile tenere a mente che si compone solo di comandi, variabili, funzioni definite dall'utente e stringhe.

Le variabili possono contenere 1 o più valori e possono essere definite attraverso:

```cmake
set(<variable> <value1> <value2> ...)
```

Essistono modi più complessi per aggiungere a variabili esistenti altri valori, si prega per queste funzionalità avanzate di fare riferimento al manuale.

In ogni caso per leggere una variabile è sifficiente eseguire:

```cmake
${varName}
```

La quale, se settata, espanderà in una lista di stringhe, mentre altrimenti rimarrà vuota.

Ultima nota preliminare, per comunicare con l'utente, è possibile usare il comando

```cmake
message("message Text")
message("la var1=${var1}")
...
```

In base all'operazione che si vuole fare normalmente il sistema comunica impostando delle variabili con un nome pre-fissato o predicibile.

## <a name="Cmake_sintassi_base"> Cmake - sintassi base </a>

Un progetto gestito attraverso Cmake ha una forma ad albero, dove ogni ogni nodo è una cartella, e al `root-point` (normalmente in cima al repository) deve essere prente un file Cmake di partenza, esso si chiama generalmente `CMakeLists.txt` e ha un aspetto simile:

```cmake
# Start point cmake
cmake_minimum_required(VERSION 3.0) # Limita la versione sotto cui non deve essere permessa l'esecuzione dello script

project(Lib_name)                   # Definisce il nome del progetto e setta la variabile di ambiente "PROJECT_NAME"
                                    # se è il CMakeList.txt top-level imposta anche "CMAKE_" 

add_executable(EXE_name main.cpp main.h)    # Dichiara il nome dell'eseguibile generato equali file deve usare
```

Una simile sintassi, anche se sufficiente per compilare, risulta molto limitante e onerosa, imponendo a ogni eseguibile di scrivere tutta la lista dei file da includere per la compilazione.

Una prima soluzione (non consigliata) coinsiste nell'impostare 2 variabili:

```cmake
set(SOURCES file1.cpp file2.cpp ...)
set(HEADER_FILES file1.h file2.h ...)
add_executable(EXE_name_1 main1.cpp ${SOURCES} ${HEADER_FILES})
add_executable(EXE_name_2 main2.cpp ${SOURCES} ${HEADER_FILES})
```

Questa soluzione può aiutare per la gestione di 1 progetto, ma risulta fallimentare nella sua estensione quando parti di progetto (le librerie) risultano comuni a più eseguibili che però non devono vedere gli altri file.

Senza continuare con questa lista di migliorie parziali, vediamo un buon modo di organizzare il progetto e conseguentemente la struttura del cmake.

## <a name="Cmake_design_progetto"> Cmake - design del progetto </a>

Un buon progetto per cmake è organizzato ad albero, seguendo questa struttura:

```
.   # Root project
├── CMakeLists.txt        # <-- Root Cmake, genera gli eseguibili del nostro progetto
├── include
│   └── ...
├── src
│   └── ...
│  ## sottomoduli delle librerie di cui vogliamo i SORGENTI                                 
├── qamqp                   # Libreria esterna, sottomodulo git
│   ├── CMakeLists.txt      # CMakeLists dei sorgenti della Libreria da includere
│   └── ...
|  ## **Vedi Guida di conan**, contiene il generato di conan
├── extResources 
│   └── ...
│  ## File del repository                                 
├── readme
└── ...
```

**Per vedere come scrivere i cmake per librerie e per exeguibili, fare riferimento a [Best Pratic](cmake_bestPractice.md).**

---

## <a name="Cmake_Compilare"> Cmake - Compilare un progetto da terminale </a>

Poichè la creazione di un cmake generalmente sporca molto il file sistem si consiglia di procedere così:

```bash
cd <path root CMakefile.txt da compilare>
mkdir build; cd build
cmake ..
make
```

---

[GO ---> BACK](../README.md)
