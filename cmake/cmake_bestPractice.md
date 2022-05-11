# Cmake Best Practice

Per scrivere un buon cmake bisogna avere ben chiaro cosa si sta facendo: [Libreria](#Cmake-Libreria), [Eseguibile](#Cmake-Eseguibile).

In questo file troverai 2 modelli per scrivere Cmake, **possibilmente da copiare e incollare**  e successivamente adattare.

## Cmake-Libreria

Per scrivere bene il Cmake di una libreria/sottosistema è importante tenere a mente queste linee guida:

- Una buona libreria è tale se contiene al suo interno tutto il necessario per essere aggiunta da fuori semplicemente linkandola.

- Altra caratteristica ben voluta è la il riconoscimento del nome della libreria con quello della directory che la contiene, vincolo non forzato da CMAKE, ma che proprio a causa di questo grado di libertà in eccesso causa la maggior parte delle volte casino.

- Un buon Cmake di libreria deve essere robusto alla possibilità di essere incluso più volte per errore (eventualità comunque da evitare ed evitata se si segue una buona struttura di progetto)

Queste linee guida possono essere trasformate in codice, in particolare con questo scheletro

```cmake
cmake_minimum_required(VERSION 3.17)

# Cattura il nome della cartella per allineare il nome della libreria a quello della cartella
get_filename_component(libName ${CMAKE_CURRENT_SOURCE_DIR} NAME)

# Verifica che il targhet non sia già presente (equivalente di #ifndef *****)
if (NOT TARGET ${libName})
    message(STATUS "[${libName}] library Start load")
    list(APPEND CMAKE_MESSAGE_INDENT "    ")    # Add indent space to better divide the import process

    # Crea il targhet che da fuori verrà "assorbito" con tutte le proprietà descritte in questo file
    add_library(${libName} [STATIC | SHARED])    

    # Elenco i sorgenti di cui è composta la libreria,
    # N.B in caso di librerie header-only vedere nota sotto
    target_sources(${libName} [INTERFACE|PUBLIC|PRIVATE] 
            file1.h file1.cpp
            file2.h file2.cpp
            #...
            )

    # Aggiungo dei Define per il compilatore con visibilità solo alla libreria
    # target_compile_definitions(${libName} [PUBLIC | PRIVATE] CMAKE_COMPILING=1)

    # Assegno al target tutte le directory dove andranno cercati i "*.h" 
    target_include_directories(${libName} [INTERFACE|PUBLIC|PRIVATE] 
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        #...
    )

    ### Include library from compiled src ###
    find_package(<lib>)
    target_link_libraries(${libName} [PUBLIC | PRIVATE] <lib>)

    #...

    ### Include library from code src ###
    add_subdirectory(<relPathToLib1>/<lib1DirName> ./<lib1DirName>)       # La cartella deve contenere un CMakeList.txt come questo
    target_link_libraries(${libName} [PUBLIC | PRIVATE] <lib1DirName>)    # Se come questo il nome della Directory è anche quello della libreria!

# Piccolo suggerimento per quando si cerca "pthread"
#    find_package(Threads REQUIRED)
#    target_link_libraries(${libName} PUBLIC ${CMAKE_THREAD_LIBS_INIT})

    message(STATUS "[${libName}] library succesfull load")
    list(POP_BACK CMAKE_MESSAGE_INDENT)    # Remove the indent for current library
else()
    # message(WARNING "<${libName}> library WAS JUST succesfull loaded, WARNING !!")
endif ()
```

Usando come base questo script, e potenziandolo con le fantastiche caratteristiche del cmake è possibile creare un solido albero, robusto a sviste e in grado di disaccoppiare tra di loro package diversi ma che dipendono l'uno dall'altro.

<detail>
<summary> <mark> <b> Definire libreria di soli header (*.h) </b> </mark> </summary>

  In caso ci si ritrovi nella situazione di dover generare una libreria composta da soli `headers` abbiamo varie strade che possono essere seguite.
  La prima e un pò "sporca" coinsiste nel creare un file `empty.cpp` vuoto e linkarlo dentro `target_sources(...)`, una simile soluzione anche se permette di superare l'empasse, non è pulita e non dovrebbe essere favorita.

  Al suo posto è possibile eliminare lo statement `target_source(...)` (per altro inesistente) e mettere:

```cmake
# Creare un interfaccia fa si che essa venga propagata a tutti gli altri target che la incontrino
add_library(${libName} INTERFACE)    

# Assegno al target tutte le directory dove andranno cercati i "*.h" 
target_include_directories(${libName} INTERFACE 
  CMAKE_CURRENT_SOURCE_DIR}/
  #...
)
```

</detail>

## Cmake-Eseguibile

Quando si vuole generare un eseguibile, è buona norma richiamare solo librerie senza generarne altre, si diventa quindi consumatori delle librerie definite nei sotto livelli.

```cmake
cmake_minimum_required(VERSION 3.17)

# Non è vincolante, ma se l'eseguibile è solo uno può essere una buona idea
get_filename_component(exeName ${CMAKE_CURRENT_SOURCE_DIR} NAME)
message(STATUS "[${exeName}] executable Start defining")
list(APPEND CMAKE_MESSAGE_INDENT "    ")    # Add indent space to better divide the import process

project(${exeName})

add_executable(${exeName} <File caratterizzanti ed esclusivi (di solito main.cpp)>)

### Include library from code src ###
add_subdirectory(<relPathToLib1>/<lib1DirName> ./<lib1DirName>)                    # La cartella deve contenere un CMakeList.txt come questo
target_link_libraries(${exeName} [PUBLIC | PRIVATE] <lib1DirName>)    # Se creato come sopra, il nome della Directory è anche quello della libreria!

message(STATUS "[${exeName}] executable successful defined")
list(POP_BACK CMAKE_MESSAGE_INDENT)    # Remove the indent for current executable
```

Se fosse necessario customizzare il Cmake, affinchè diventi parametrico, un comando utile è option:

```cmake
# option (OptionName "<Options description>" [ON/OFF]_defultValue )
option(WITH_EXAMPLE "Enable examples" ON)
if(WITH_EXAMPLE)
   # operations ...
   add_subdirectory(Examples)
endif()
```

Così facendo è possibile sul terminale modificare l'opzione usando:

```bash
cmake <Path/CMakeLists.txt> -D<OptionName>=ON/OFF # ...
```

L'opzione risulta particolarmente utile per l'integrazione con `conan` permettendo di far disattivare a `conan` delle opzioni indesiderate durante la creazione di pacchetti e lasciarle attive di default quando si compila in locale.

---

## Ricerca delle dipendenze

Quando si usa il `find_package` di dei pacchetti di sitema operativo, è buona norma seguire una sintassi di questo tipo:

```cmake
find_package(SQLite3)
if(NOT SQLITE3_FOUND)
    message(FATAL_ERROR "SQLite3 not found, try install with:\nsudo dnf install libsqlite3x-devel")
endif()
target_link_libraries(${libName} PUBLIC sqlite3)
```

Con questa struttura si hanno i seguienti benefici:

1. Essendo `FATAL_ERROR` il messaggio e terminale e si evita di andare avanti, rendendo la soluzione più immediata

2. Scrivendo il codice, o i suggerimenti necessari a risolverlo, si comunica indirettamente agli utilizzatori della libreria cosa fare solo quando serve

---

[GO ---> BACK](README.md)
