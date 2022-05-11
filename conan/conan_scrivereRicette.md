# Creazione Ricetta

Come descritto nella documentazione "Conan is a software package manager which is intended for C and C++ developers"
Per raggiungere questo scopo l'elemento base che permette la magia è la **Ricetta** conan.
Essa è l'insieme dei comandi che permette a conan di risalire le dipendenze per compilare il progetto corrente e generare gli eseguibili linkando i giusti file binari, e in mancanza di essi facendoli compilare localmente a conan in automatico.

Una **Ricetta** è esprimibile attraverso un basico file *.txt* contenente ciò di cui l’applicativo necessita per poter essere compilato (dipendenze, direttive ...) o mediante una più potente classe in *python*.

- [Creazione delle ricette](#ricette)
  - [Ricetta mediante `conanfile.txt`](#ricetta-txt)
  - [Ricetta mediante `conanfile.py`](#ricetta-py)
- [Includere Conan nei progetti](#conan-include)
  - [Qmake include code (*.pri)](#qmake-include)
  - [Cmake include - Procedura](#cmake-include)

## Creazione delle ricette

## <a name="ricette"> Creazione delle ricette</a>

### <a name="ricetta-txt"> Ricetta mediante `conanfile.txt`</a>

Per descrivere mediante questo metodo i passi da fare sono:

1. Creare un file conanfile.txt nella directory principale del progetto

2. Modificarlo inserendo le sezioni base di una ricetta
   
   - **`[requires]`** - individua le dipendenze del progetto, i.e. le librerie le librerie necessarie
     per la compilazione dell’applicativo, scritte secondo il formato
     
     ```
     <nome Libreria>/<release>@<percorso git>/stable
     ```
     
     dove:
- release riferisce la tag, nel caso riportato per la prima dipendenza (se la Tag è
  v2.4.0 si omette “v”)

- percorso git è il classico path git da cui vanno eliminati prefisso e postfisso e
  sostituito lo ‘/’ con il ‘+’
  e.g. : `git@elt-gitlab:HCI/hmi_core.git diventa HCI+hmi_core`
  Un modo pratico per vedere quali sono le dipendenze richieste è leggerle dal
  classico extResources.pri
  Per vedere quali versioni di una specifica libreria sono disponibili su gitlab, in
  modo tale da poterne includere una nel conan file, si può eseguire una ricerca
  conan con il seguente comando:
  `conan search –r gitlab <my lib name>`
  
  - **`[generators]`** - individua il comando utilizzato per generare i `makefile` di
    compilazione, e.g. `qmake`, `cmake`

Ecco un esempio di  `conanfile.txt`:

```
[requires]
HmiCommonCore/2.4.0@HCI+hmi_core/stable
ELTIpc/2..1.0@HCI+ELT_IPC/stable
[generators]
qmake
```

Maggiori dettagli sono reperibili nella guida ufficiale al link:

1. https://docs.conan.io/en/latest/using_packages/conanfile_txt.html

2. https://docs.conan.io/en/latest/reference/conanfile_txt.html

### <a name="ricetta-py"> Ricetta mediante `conanfile.py`</a>

Questo metodo più sofisticato, permette di ricreare tutto ciò che si può realizzare con il `conanfile.txt` con **l'ulteriore grado di libertà** di poter specificare, tramite programma python, il dettaglio degli step di building, packing, importing, etc...
Maggiori dettagli sono reperibili al link: 

1. [Tutorial conanfile.py](https://docs.conan.io/en/latest/mastering/conanfile_py.html)

2. [Reference conanfile.py](https://docs.conan.io/en/latest/reference/conanfile.html)

Per generare un `conanfile.py` di base posizionarsi nella directory del progetto e digitare:

```bash
conan new <libName>/1.0
```

Essa genererà un codice di base da customizzare, per renderlo adatta al nostro progetto.

Altrimenti un primo sketch base di un `conanfile.py` contentente solo metodi che overridano la classe base di conan (vedi reference) e già automatizzato in alcuni passaggi è il seguente :

```python
from conans import ConanFile, CMake, tools

class <libName>(ConanFile):
    #########################################################################################################
    # Impostazioni e info per creare e ritrovare la ricetta, distinguendola dalle altre presenti in GitLab  #
    #########################################################################################################
    license = "Alfystar"
    description = "<short description of the source>"
    url = "https:/<repository path>"
    name = "<libName>"    # Name of the Recepies
    default_user = "HCI+<Repository Name>"  # User of the recepies, of us is the gitlab group + repository name
    default_channel = "stable"    # Channel of the Recepies

    # Set the Recepies version equal to the current git checkout tag
    def set_version(self):
        git = tools.Git(folder=self.recipe_folder)
        self.version = git.run("describe --abbrev=0")
        if self.version[0] == "v" : 
            self.version = self.version[1:]

    # Permit to set env_info, user_info, cpp_info, that are variable userful for a rapid debug
    def package_info(self):
        self.cpp_info.libs = tools.collect_libs(self)

    #####################################################################################################
    #                          Parametri e dipendenze specifiche della libreria                         #
    #####################################################################################################
    # My requires library to find with conan
    # path rule (default): <libReq_name>/<libReq_tagVersion>@<repository path>/stable
    requires = "<require Conan path>"
    # List of library opiton for the consumer
    options = {"shared": [True, False]}     # The "shared" parameter is passed to cmake through the CMake class
    default_options = {"shared": True}
    generators = "cmake", "cmake_find_package"
    # Tell to conan which field is use to generate the unique ID (sha-hash), from the current profile set
    settings = "os", "compiler", "build_type", "arch", "qt_version", "distro"

    # scm is used by conan to upload the recepies to a remote server
    scm = {
        "type": "git",
        "url": "auto",
        "revision": "auto",
        "verify_ssl": False,
        "shallow": False,
        "submodule": "recursive"
    }

    #####################################################################################################
    #                     Metodi da usare nella fase di build ed export del risultato                   #
    #####################################################################################################
    # This method is used to build the source code of the recipe using the desired commands
    def build(self):
        cmake = CMake(self)
        # Passing parameter to the Cmake file;
        # es: cmake.definitions["WITH_EXAMPLE"]=self.options.build_example
        cmake.definitions["<Variable name>"] = <Value>
        # ...
        cmake.configure()
        cmake.build()

    # Called after success build, generate a list with absolute paths
    # of the files copied in the destination folder.
    # MUST be use self.copy(...)
    def package(self):
        self.copy("Find*.cmake", keep_path=False)    # in base ai casi
        self.copy("*.h", dst="include", src="<Source Path>", keep_path=False, excludes="private")
        self.copy("*.lib", dst="lib", keep_path=False)
        self.copy("*.dll", dst="lib", keep_path=False)
        self.copy("*.exp", dst="lib", keep_path=False)
        self.copy("*.dylib*", dst="lib", keep_path=False)
        self.copy("*.so", dst="lib", keep_path=False)
        self.copy("*.a", dst="lib", keep_path=False)
```

Le ricette `conan` vengono usate da cmake ed usano cmake nella fase di build, per scrivere un cmake consultare la quick guide [cmake guide](/cmake/cmake_guide.md).

#### Info varie

##### cmake.definition

Tramite **`cmake.definitions`** è possibile comunicare dei parametri al Cmake (**DEVONO essere impostati prima di `cmake.configure()`**), si consiglia di usare il comando `options(...)` dentro il `CMakeLists.txt` come descritto nel [Cmake Best Practice](../cmake/cmake_bestPractice.md_).

##### settings

La lista di patametri `settings` dichiarata nella ricetta è usata da `conan` per creare un HASH, i semi di questo hash sono presi dal profilo corrente (di default contenuto dentro `~/.conan/profiles/default`) e permette di risalire facilmente alla versione di pacchetto desiderata. Essa viene usata per la ricerca Online sui repository Gitlab, se la firma corrisponde i pacchetti vengono scaricati, altrimenti conan scarica i sorgenti e ricompila in locale dentro la sua cash i pacchetti usando i settings del profilo attuale.

<mark>Della lista complessiva proposta, lasciare solo quelli necessari a caratterizzare esaustivamente il binario che si vuole compilare</mark>.

**È importante** avere i settings della propria macchina allineati alla condizione reale, così da garantire la compatibilità tra lo scaricato e il compilato in loco.

## <a name="conan-include"> Includere Conan nei progetti </a>

<details>
<summary> <b> Qmake include code (*.pri) (Tendenzialmente in disuso) </b>  </summary>

  Per permettere a conan di creare/scaricare i pacchetti è necessario predisporre uno spazio nel repository locale (**Da non committare**):

```bash
cd <path_root_src>
mkdir -p extResources 
cd extResources
touch extResources.pri
```

Una volta creato il file `extResources.pri` che verrà incluso da qmake, è necessario popolarlo affinchè possa scegliere se prendere i binari attraverso `conan` o alla "vecchia maniera".
I binari di conan verranno creati/posizionati una volta chiamato `conan install` ([vedi installare dipendenze con conan](README.md)), e ciò viene pilotato attraverso la ricetta.

Per popolare file `extResources.pri` questo è lo sketch di base (da copiarci dentro e personalizzare):

```qmake
exists($$PWD/conanbuildinfo.pri)
{
    # Includere i settaggi riportati nel conanbuildinfo.pri
    CONFIG += conan_basic_setup
    include($$PWD/conanbuildinfo.pri)
    # Definire RPATHDIR
    QMAKE_RPATHDIR += \
        $${CONAN_<lib1>_ROOT}/lib \
        $${CONAN_<lib2>_ROOT}/lib
        # ...
}else{
    # Include ‘tradizionali’ quando conan non è installato
    include($$PWD/<lib1>.pri)
    include($$PWD/<lib2>.pri)
    # Definire RPATHDIR (usato in unix per il linking)
    QMAKE_RPATHDIR += \
        ../lib/<lib1>/lib \
        ../lib/<lib2>/lib
}
```

Con i file `<lib1>.pri` che descrivono come includere e compilare dentro il progetto corrente i sorgenti della libreria target, e abbiamo come obiettivo di generare le interfacce contenute nei `*.h` e binari nel `*.so`, se possibile, altrimenti inludere il `.pri` del repository (conan risolve questi casini)

```qmake
QT *= <moduli qt necessari>
INCLUDEPATH += $$PWD/include # The includes "*.h" generated after the building, change on necessity

# CONFIG(debug, debug|release) evaluates to true if CONFIG contains "debug" but not "release", or if it contains both "debug" and "release" but "release" doesn't appear after the last occurrence of "debug".
CONFIG(release, debug|release): LIBS += -L$$PWD/lib/ -l<libName senza il prefisso lib>
CONFIG(debug, debug|release): LIBS += -L$$PWD/lib/ -l<libName senza il prefisso lib>d

unix {
    QMAKE_RPATHLINKDIR += $$PWD/lib
}
```

</details>

### <a name="qmake-include">  </a>

### <a name="cmake-include"> Cmake include - Procedura </a>

Per includere i pacchetti compilati e gestiti attraverso conan in un progetto gestito con il cmake, prego fare riferimento alla guida [Cmake - Conan](../cmake/cmake_conan.md).

---

[GO ---> BACK](README.md)
