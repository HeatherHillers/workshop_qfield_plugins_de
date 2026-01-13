
# Plugin-Struktur
Öffnen Sie ${ROOT}\qfield_vegetation_monitoring in Visual Studio Code oder Ihrer bevorzugten IDE und werfen Sie einen Blick auf das Layout:

![alt text](img/demo1_struct.png)

  - Die Haupt-QML-Datei ist demo1_hello.qml. Sie muss denselben Namen wie das Projekt haben und muss im Projektverzeichnis mit der Projektdatei gespeichert werden.
  - Alle anderen QML-Dateien werden im Unterverzeichnis components gespeichert.
  - Dieses Unterverzeichnis muss als Anhang-Verzeichnis im Projekt hinzugefügt werden.

## Global importierte Klassen werden in qmldir definiert

Die qmldir-Datei ist im Grunde eine globale Import-Direktive.  

d1_plugin_theme.qml enthält ein Stildefinitionsobjekt, das Farben und Schriften enthält, die wir konsistent in unserem Plugin verwenden werden. Es ist wie eine CSS-Datei.  

Die qmldir-Datei definiert d1_plugin_theme.qml als Modul, das global verwendet und automatisch importiert wird. Sie definiert, dass das Root-Element dieser Datei importiert und im Code als PluginTheme-Objekt bezeichnet wird.  

Beachten Sie, dass d1_plugin_theme.qml keinen Verweis auf den Klassennamen PluginTheme hat. Dies wird nur in der qmldir-Datei definiert.

### Definition von PluginTheme in qmldir

```qml
singleton PluginTheme 1.0 d1_plugin_theme.qml
```
### Inhalt der PluginTheme-Datei hat keinen Verweis auf PluginTheme

```qml
pragma Singleton
import QtQuick

QtObject {
    readonly property color vanilla: "#ffecd1"
    readonly property color green: "#6baa75"
    readonly property color white: "#ffffff"
}
```
# 📚 **[>> Schauen wir uns das Hauptmodul an](DEMO1_MAIN.md)**
## 📚 **[<< Plugin-Bereitstellung](DEMO1_DEPLOY.md)**

