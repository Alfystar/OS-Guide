# Cmake-QT

Qui è possibilie trovare qualche nota importante per includere correttamente QT all'interno di un progetto cmake:

- [Cmake sets variable](#qtVariableSet)
- [Cmake compatibile con QT5/6](#qt5_6_compatibility)
- [QTest e Cmake](#qt_testSuite)
- [Integrazione al CMAKE_PATH per QT Installato da installer (online/offline)](#qt-cmakePathFind)

## <a name="qtVariableSet"> Cmake sets variable </a>

Ogni progetto/libreria che usa QT, richiede siuramente di impostare le seguenti variabili di ambiente che permettono la compilazione dei *moc* necessari a QT per compilare. **DEVONO ESSERE POSIZIONATI PRIMA DI add_library(...)**:

```cmake
# Qt Compiler enable, MUST be befor "add_library"    
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
```

Queste variabili abilitano cmake a chiamare i compiler che creeranno i file `.moc`, `.rcc` e `.ui` indispensabili al funzionamento dell'applicativo.

<mark>**ATTENZIONE**:</mark> Questi compiler però sono eseguiti solo sui file che lo richiedono specificati dentro `target_sources(..)`.

#### Qmake-Cmake equivalent

In aggiunta a ciò, è comodo sapere che, come in qmake si scrive:

```qmake
QT += <moduleName1> <moduleName2>
```

Il suo equivalente in cmake è:

```cmake
find_package(Qt5 COMPONENTS <moduleName1> <moduleName2> REQUIRED)
message(STATUS "Found Qt" ${QT_VERSION_MAJOR} " (${QT_VERSION})")
# ...
target_link_libraries(${libName} [PUBLIC | PRIVATE | INTERFACE] <moduleName> )
```

**NOTA BENE:** Una qualunque classe che discende da QObject, impone l'uso del modulo `Core` e l'abilitazione di `set(CMAKE_AUTOMOC ON)` per permettere la compilazione, altrimenti si otterranno errori di `undefined reference to "vtable for MyReceiver"`!!!

---

## <a name="qt5_6_compatibility"> Cmake compatibile con QT5/6 </a>

Per rendere il cmake compatibile sia per Qt5 che per Qt6 il modo migliore di procedere è il seguente:

```cmake
## !! Attualmente presenti dentro eltCmake/eltCmake !!
# Dynamic discover for Qt Version
find_package(QT NAMES Qt6 Qt5 COMPONENTS Core REQUIRED)
message(STATUS "Found Qt" ${QT_VERSION_MAJOR} " (${QT_VERSION})")
```

I quali sono ra presenti dentro `eltCmake/elt.cmake`, grazie a questi comandi ci diventa possibile linkare i moduli di nostro interesse scriverdo:

```cmake
# Package find for current Qt version...
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS <moduleName1> <moduleName2> REQUIRED)

# Linking to adapt at configured Qt version...
target_link_libraries(${libName} [PUBLIC | PRIVATE | INTERFACE] Qt${QT_VERSION_MAJOR}::<moduleName1> Qt${QT_VERSION_MAJOR}::<moduleName2>)
```

Procedendo così avverrà, nell'ordine:

1. Verrà cercato `Qt6`, se trovato verranno impostate le variabili e si cercherà quel modulo

2. In caso di fallimento si procederà verso `Qt5` per impostare conseguentemente le variabili

Al termine del primo comando, se si avrà avuto successo le variabili di ambiente saranno state impostate e bisognerà solo cercare e linkare i moduli di proprio interesse.

*Se si fosse nella necessità di far scegliere la versione a riga di comando, attraverso un if è possibile selezionare la versione richiesta e si riscrive il primo comando togliendo la versione indesiderata.*

---

## <a name="qt_testSuite"> QTest e Cmake </a>

Per poter generare dei test usano Qt attraverso cmake, il cmake che contiene la definizione dell'eseguibile deve possedere poche caratteristiche extra (fare comunque riferimento alle [buone pratiche](cmake_bestPractice.md) nella sua scrittura di base.

```cmake
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Test REQUIRED)
enable_testing()    # Enable te qt test suite
add_executable(exeName <files>)  # normale sintassi per creare un eseguibile
add_test(NAME <Nome Da dare al test> COMMAND <comando da eseguire per il test, tipicamente il nome dell'eseguibile>)
target_link_libraries(exeName Qt${QT_VERSION_MAJOR}::Test)
```

Il comando `add_test(...)` fa parte della suite `ctest` di cmake, e permette di automatizzare i test e tenerne traccia.
Per poter eseguire il tutto, dopo aver compilato, entrare nella cartella che contiene i compilati per i test, al suo interneo sarà presente un file `CTestTestfile.cmake`, grazie ad esso sarà possibile da terminale eseguire i test.

```bash
ctest # Eseguire tutti i test, uno dopo l'altro.
ctest -R <name_of_your_test or experssion_to_execute>
```

Per avere un output più dettagliato dei singoli test consultare il log, o eseguirli singolarmente.

---

## <a name="qt-cmakePathFind"> Integrazione al CMAKE_PATH per QT Installato da installer (online/offline)  </a>

Se si è installato Qt attraverso l'installer, nessuno ha posizionanto dentro i path di `cmake` dove trovare i file  `*.cmake` di Qt .
Per aggiungerli manualmente è sufficiente aggiungere alla variabile di enviroment del terminale: `CMAKE_PREFIX_PATH` il path dove sono posizionati i `*.cmake`.
Per evitare di incorrere sempre in questo problema, si consiglia di aggiungere l'export corretto al termine del proprio `.bashrc`

**In caso si abbiano sulla propria macchina più versioni di Qt**, come [descritto nella guida di setup della propria macchina](../00_utility/elt-system-tool-setup.md), è preferibile aggiungere il al `CMAKE_PREFIX_PATH` il path passante per il link simbolico "mobile":

```bash
# cmake find path serch
export CMAKE_PREFIX_PATH=$HOME/.local/qtVersion/gcc_64/lib/cmake/:$CMAKE_PREFIX_PATH
```

---

[GO ---> BACK](README.md)
