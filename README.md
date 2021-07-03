# Qt5/Qt6 Internationalization with CMake and Qmake

This project is a tutorial and sample project of a Qt application using 
[internationalization](https://doc.qt.io/qt-5/internationalization.html), 
with translations embedded inside the executable as resources. Both CMake and Qmake build systems are included.

## Loading and installing translations in C++

This is an excerpt from the `main.cpp` file in the project:

    QLocale locale;
    QTranslator qtTranslator;
    qDebug() << "load Qt translator:" << locale.name()
             << qtTranslator.load(locale, "qt", "_", ":/");
    QCoreApplication::installTranslator(&qtTranslator);

    QTranslator appTranslator;
    qDebug() << "load app translator:" << locale.name()
             << appTranslator.load(locale, "i18n", "_", ":/");
    QCoreApplication::installTranslator(&appTranslator);

Two instances of [QTranslator](https://doc.qt.io/qt-5/qtranslator.html) are created. 
The first instance loads the Qt translations. The second one loads the program 
translations. Both instances `load()` the QM files corresponding to the default `QLocale` 
that is created without arguments. You may want to use environment variables on Linux to change the 
default system locale:

    LANGUAGE=ca_ES
    LC_CTYPE=ca_ES.UTF-8

## Using the Qmake build system

The translations are declared in the project.pro file as usual using the variables `TRANSLATIONS` and 
[EXTRA_TRANSLATIONS](https://doc.qt.io/qt-5/qmake-variable-reference.html#extra-translations). 
I'm declaring here four of the languages spoken in my country.

    TRANSLATIONS += \
        i18n_ca.ts \
        i18n_es.ts \
        i18n_eu.ts \
        i18n_gl.ts

The creation and maintenance of the language translations can be done directly from the command line 
shell with the help of the [lupdate utility](https://doc.qt.io/qt-5/linguist-manager.html):

    $ lupdate project.pro

To generate the *.qm binary files and the *.qrc resources, there are already standard configurations.

    CONFIG += lrelease embed_translations
    LRELEASE_DIR=.
    QM_FILES_RESOURCE_PREFIX=/
    
Including in the `CONFIG` variable the configuration [options](https://doc.qt.io/qt-5/qmake-variable-reference.html#config) 
`lrelease` and `embed_translations`, the translations are compiled and the resources generated at build time. 
The undocumented variable `LRELEASE_DIR` determine the output directory of the *.qm files 
(relative to the project's output). and `QM_FILES_RESOURCE_PREFIX` determine the prefix used by 
the compiled resources. You need this prefix when loading the translations in your *.cpp sources.

The only missing spots are the Qt translations, the translations of Widgets and other classes shipped by Qt. 
`lconvert.pri` is the only custom script provided by this recipe.

    LCONVERT_LANGS=es ca gl eu
    include(lconvert.pri)

With these lines, `LCONVERT_LANGS` determines the languages that are processed, 
and `LCONVERT_PATTERNS` the modules that are merged into each translation, 
by default: `qtbase`, `qtmultimedia`, `qtscript` and `qtxmlpatterns`.

## Using the CMake build system

Qt provides already a CMake function to build translations in the 
[LinguistTools module](https://doc.qt.io/qt-5/cmake-command-reference.html#qt5-linguisttools) 
that you only need to request it along with other modules in your `CMakeLists.txt`

    find_package(QT NAMES Qt6 Qt5 REQUIRED)
    find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Core Gui Widgets LinguistTools REQUIRED)

    set(TS_FILES
        i18n_ca.ts
        i18n_es.ts
        i18n_eu.ts
        i18n_gl.ts
    )

    qt5_add_translation(QM_FILES ${TS_FILES})

Please see the documentation of the 
[qt5_add_translation](https://doc.qt.io/qt-5/qtlinguist-cmake-qt5-add-translation.html) 
function for details and available options.

The LinguistTools module provides also a 
[qt5-create-translation](https://doc.qt.io/qt-5/qtlinguist-cmake-qt5-create-translation.html) 
function that is not used in this tutorial.

You may include the variable `${QM_FILES}` directly in the list of sources for your target, 
and install the *.qm files as always. But instead, we are going to embed the translations 
as resources. To do so, this project has a `TranslationUtils.cmake` script that you need to 
include in your project to use the new macros.

    include(TranslationUtils)
    add_app_translations_resource(APP_RES ${QM_FILES})
    add_qt_translations_resource(QT_RES ca es eu gl)

    add_executable(CMakeI18n
        main.cpp
        ${APP_RES}
        ${QT_RES}
    )

The `add_app_translations_resource()` function produces the resource.qrc file from your 
`${QM_FILES}`, and the `add_qt_translations_resource()` produces another resource file for the 
requested languages and the Qt provided translation files.

Finally, there is also a `lupdate` custom target, in case you need to update your sources 
translations from the project sources and using the command line shell:

    $ cmake --build . --target lupdate

Copyright © 2019-2021 Pedro López-Cabanillas.  See the LICENSE file for details.
