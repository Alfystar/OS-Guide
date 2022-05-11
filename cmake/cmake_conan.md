# Cmake - Conan

In caso si voglia usare la libreria come pacchetto compilato è consigliabile usare il tool `conan` per il recupero, compilazione e rintracciamento dei pacchetti (vedi guida `conan` per scoprire come funziona).

Tra le possibili strade da seguire, in elettronica abbiamo optato per far chiamare da `cmake` il comando `conan install <path_ricetta>`, così facendo è possibile astrarre il processo di recupero dei pacchetti al programmatore e rendere il progetto in grado di essere compilato sia come `consimatore` che come `creatore`. I file di conan verranno creati da Cmake  all'interno della directory di `build`.

Per realizzare questo work flow dentro i `CMakeLists.txt` che necessitano di importare dei binari per compilare le librerie è sufficiente inserire:

*Pre requisito: Devono essere include le macro di conan e di elettronica, presenti dentro ELT_COMMON_SRC/eltCmake*

```cmake
include(01_subModuleLink/eltCmake/elt.cmake) # tra le altre cose eseguie anche la ConanLogin 
# ...
# ...
# ...

# Conan linking block
conanlogin()    # Macro incluided by elt.cmake
conan_cmake_install(
        PATH_OR_REFERENCE ${CMAKE_CURRENT_SOURCE_DIR}/<conanfile.py/txt relative path>
        BUILD missing               # Build only missing package
    )
include(${CMAKE_CURRENT_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup(TARGETS)

#...

target_link_libraries(${libName} PUBLIC CONAN_PKG::<PLG_Name1>) # Il nome del PKG sono le "XXXX" di XXXX/tag@repoPath/stable
target_link_libraries(${libName} PUBLIC CONAN_PKG::<PLG_Name2>)
# ...
```

La presenza di `CMAKE_CURRENT_SOURCE_DIR` permette di lavorare dentro la cartella corrente, ma nella struttura copiata dentro la build-directory. Questo trucco permette di rendere universale il cmake rispetto alla `conan install ...` (presente al suo interno) e il `conan create ...` chiamato dal doker per il deploy.

### Spiegazioni:

Le macro di conan qui usate, oltre alle nostre, vengono caricate attraverso `elt.cmake` e fanno riferimento al progetto [GitHub - conan-io/cmake-conan: CMake wrapper for conan C and C++ package manager](https://github.com/conan-io/cmake-conan), per maggiori informazioni e aggiornamenti guardare lì.

Dello questo snipet di codice propsto vediamo di spiegare cosa sono e cosa fanno in breve:

1. `conanlogin()` := è una macro importata attraverso `elt.cmake` e permette di far loggare conan con il remote, per funzionare è importante che il prorio sistema abbia impostate `CONAN_LOGIN_USERNAME` e `CONAN_PASSWORD` come descritto nella [guida di installazione di conan](../conan/conan_instalazzione.md), queste variabili sono definite anche dentro il doker e **DEVONO AVERE LO STESSO NOME**.
2. `conan_cmake_install(...)` := è  un wrapper di `conan install` e permette di richiamare il comando sfuttando tutte le sue features (come il filelock) direttamente da dentro Cmake, automatizzandone la risoluzione, abbiamo scelto di far compilare in automatico i pacchetti mancanti nella cash locale, ma l'obiettivo nel docker è cercare di recuperarli dalle altre pipeline.
Il Cmake al termine di questa compilazione potrebbe riportare gli errori che aveva riscontrato, ma è un falso allarme, infatti rieseguendo il cmake (senza ovviamente cancellare la cache di conan) il cmake non darà nessun errore e tutto procederà liscio.
3. `include(${CMAKE_CURRENT_BINARY_DIR}/conanbuildinfo.cmake)`  := include in cmake i setting che conan genera per poter finalmente linkare le librerie
4. `conan_basic_setup(TARGETS)` := dentro il wrapper cmake di conan vengono "caricate" nel namespace di Cmake i target che sono stati recuperati (attraverso download o build di missing libs) e permette di raggiungerli e farli linkare ai target di nostro interesse.

Tra le funzionalità non in uso, ma di cui sarebbe bene sapere abbiamo:

1. `conan_cmake_autodetect(settings)` := essa, posizionata tra la login e la install, permette al cmake di recuperare tutte le impostazioni della macchina corrente reale e di salvarle dentro una variabile (settings per l'appunto), sarebbe possibile usare questa variabile per comunicare all'install la configurazione da installare, ma questa macro è per noi **indesiderata** poichè il nostro obiettivo è avere i profili di default impostati dentro `~/.conan/profiles` allineati alla macchina reale. In caso allineare manualmente il file piuttosto che usare questa macro.

2. `conan_cmake_configure(..)` := esso è un wrapper che permette di ricreare tutto ciò che è contenuto in [conanfile.txt](https://docs.conan.io/en/latest/reference/conanfile_txt.html) direttamente all'interno di Cmake. Per progetti in cui la ricetta serve solo per scaricare requisiti o altre cose, potrebbe essere sensato avere tutto raccolto nel cmake, da analizzare all'occorrenza. 

---

## Impostare versione di compilazione

È possibile impostare la creazione di pacchetti in modalità diverse dalla release, digitando la `conan create` con il comando:

```cmake
conan create <args ...> -s build_type=<Type>
```

---

[GO ---> BACK](README.md)
