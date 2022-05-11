# QT6 porting IPC libs

The  `XMLPATTERN` library present in the  QT5, now is absorbed to the `XML` library, so from the `Ipc.pro` file it are remote from the QT variable
Consequently, these include:

```c++
#include <QXmlDefaultHandler>
```

Not exist anymore, and to include these classes 

```C++
#include <QXmlSimpleReader>
#include <QXmlInputSource>
```

Have to add in the `ipc.pro` file:

```qmake
QT          += core5compat
```

But request the use of Qchart, unfortunatly under GPLV3 licence

For that reason we move to use other standard qt:

1. Because `QXmlDefaultHandler` is now invalid, the `IpcXmlHandler` now simply descend from `QObject`
2. `QXmlInputSource` was responsible to import the XML as class, now we move to `QXmlStreamReader`, it parse **well-formatted** XML and permit to generate a DOM tree
   *Note*: `ValidatorXML` was also remove from QT6, so now we have to found new validator.
   As describe from https://doc.qt.io/qt-6/xml-changes-qt6.html change the `parse()` and include the `fatalError()` function, called through the exeption path, is directly call inside `parse()`.
   For that reason the `FatalError()` is now removed 

# Branch change

After the initial work, comes out there was a precedent work to porting the XML part of the library to the DOM paradigm.

So now we move to this branch and start the porting work.

## QAMQP Library Porting to Qt6

At the beginning comes out the problem are in the `qamqp` library because ii use an old method obsolete in Qt6.

To make the porting correctly, and also use the new features of Qt6, first we need to study what the library want do.

---

### QDate align to standard signature

After the first try compilation in Qt6, the first error found is in the changing of QDate method with new and more standard signature, to correct this error simply search the new equivalent in the on-line manual.

---

### QHash & QMultiHash

#### Start breifing

`QHash` is the Qt class that provide an implementation for hash dictionary, this data type have excellent performance to search information using one search key. (Asymptotically cost for most operation is O(1)). Off course this result are payed with memory occupation.

`Qhash` and his son `QMultiHash` is present in Qt API since Qt4.

Originally QHash support multi key saves, but in fact this Beauvoir aren't desirable for that class, for these reason the QMultiHash class was created.
The data structure is off course the same, so, using the QHash interface method is possible found multiple same keys, but in the mind of the Qt Developers, the only Right way to insert multiple key is passing from `QMultiHash` method.

#### `QAMQP` Situation

All the study do before, comes out for an error compile problem:
the method **`unite`** was called to merge 2 dictionary in to one.

The original implementation (4.8 version) do the following Beauvoir:

> Using this method, the caller dictionary import inside his database the other Dictionary, and if there was multiple instance of the same key the resulting Dictionary will contain 2 same key pared with their original values. 

But how we said before, starting from the Qt5 version, qt developer decide to improve the QMultiHash class.

At this point the possible solution are 2:

1. Change the type of the data-structure into the `QMultiHash`
2. Make more strict the possibility to insert multiple key in the dictionary.

##### `unite` problem solution

 Analyzing the `QAMQP` code, is easy to see that this functionality is use only to permit at the user of the library to add *custom properties*, more of that, in the ELT_IPC library this features is never use, so, ti solve the issue i think the most appropriate solution is the **2**.

For that reason now is a **MUST** for a generic user of this library use a  different key for different information (that sound reasonably).
At the code level, to solve this problem simply we have to change `unite --> insert` method in `void QAmqpClientPrivate::startOk()`.

The `insert` Beauvoir is quite different, instead add multiple key on the list *(this now **(Qt6)** is possible only with `QMultiHash` class through `unite` )*, it replace the key with the new value present in the importing dictionary if already present.
In the case of the imported dictionary have multiple same key saved inside  is unpredictable what will be save (depend from the ordering inside the dictionary in the merge moment), but this should not be our case...

---

### QIOdevice include problem

Inside `qamqpframe.cpp` is used  `QIOdevice` interface method to comunicate with the file.
In the changing from Qt5 to Qt6, the QIOdevice class was enhanced, and some include, precedented did inside other library now are moved.

For that reason to solve this issue only need to add  `#include <QIODevice>` inside `qamqpframe_p.h`

---

## Example issue

During the compilation, there was a problem for the `endl` method, now moved inside a `Qt namespace`, to solve this problem simply change  `endl --> Qt::endl`

# Running Testing sources

After the building success is time for the test.

### QDomDocument Parsing problem

Unfortunatly the Qt5 version has less runtime error than the Qt6porting at this stage, the first failure point is in the QDomDocument Parsing and for that reason now, i start the study of the QdomDocument class in Qt6, an the difference from Qt5.

Fist problem is inside the `document.setContent(&file_)` in `bool eltipc::IpcXmlHandler::parse()`

For some reason in Qt5 version file is recognize end parsed well, but  in Qt6 parsing reach some error and return false at the function, here the snapped:

```c++
bool eltipc::IpcXmlHandler::parse()
{
    bool result = true;
    QDomDocument document;
// In Qt5 success, Qt6 fail
    if (!document.setContent(&file_))
    {
        QString fatalMessage = Q_FUNC_INFO;
        fatalMessage.append("Failed to load the file for reading.");
        fatalMessage.append(file_.fileName());
        result = false;
        qFatal(&fatalMessage.toStdString()[0]);  // fatal or warning?
    }

    // Getting root element
    QDomElement root = document.firstChildElement();
    readConfigurationName(root);
    readKeepaliveConfiguration(root);
    readMessages(root);

    return result;
}
```

After the studies of the problem, comes out that: before in validation function a temporal QFile variable was create from the same resource, in Qt5 this create a bridge to the file with the SO equal for all QFile accessing the file throgh resource module. In Qt6 this error are solve, but now in constructor and destroier need to be open and than close.

After the application of this solution the work restart.

### connect of the signal QSslSocket::error(QAbstractSocket::SocketError)

After the use of the test to understand if the module run, now we start to use the Example to test specific function.
Fist error comes out is: 

```
No such signal QSslSocket::error(QAbstractSocket::SocketError) in ../qamqp/src/qamqpclient.cpp:65`
```

Is a run-time problem and is caused by the change of the error signal signature:

```
        Qt5                          Qt6
QSslSocket::error(...) --> QSslSocket::errorOccurred(...)
```

This change was execute because error function was already overloaded to get information and that semantic could cause mistake.

# Deprecate function update

With the goal to prevent future build fail adapting the deprecate function with the news.

### Qvariant::type() --> New QMetaType system

Now all the information to encapsulate the type of the variable own in QVariant class is conteined inside QMetaType class.

More over, now the enum code is change (user type for example is enanced to 65536, and before was 1024), for all that reason, che porting in the new version could cause some problems.

First change is the `Qvariant::type()` in 2 new different command:

1. typeName() : Return the string printed in the Qt5 version 
2. typeId() : Return the enum code of the type stored for the compare
3. metaType() : Return the QMetaType of the stored value conteined in the QVariant variable.

This change start from the Qt5 -> Qt6 transition and could cause some problem...

Now to test the type the correct form is:

```c++
// Qt5 Style
QVERIFY(arguments.at(1).type() == QVariant::String);
// Qt6 Style (Note the new String --> QString)
QVERIFY(arguments.at(1).typeId() == QMetaType::Type::QString);
```

This change in Qt6 is probably to divide and simplify the aspect to test, because early all the differentiation was doing inside the API and not ever the test was correctly

# Security issue update

During the compile time, the compiler see:

```c++
qFatal(fatalMessage.toStdString().c_str());
```

As ` warning: format string is not a string literal (potentially insecure)`, this is because, qFatal() like printf(), accept the sintax : `printf(<FormatString>, <arguments>)`.
If the message is passed directly, is possible (very hard in reality) manipulate the as the arguments is see from the function.
To eliminate this security risk simply modify the message as:

```c++
qFatal("%s", fatalMessage.toStdString().c_str());
```

---

By Alfystar (Emanuele Alfano) 29/11/2021

[`GO --> BACK`](README.md)
