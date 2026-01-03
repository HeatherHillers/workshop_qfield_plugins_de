````markdown
# Demonstration 4: Vom Map-Klick zur Header-Seite gelangen

Lassen Sie uns unseren Platz schnell finden.

## 1. Die demo4_header_form.qml ist dieselbe wie in Demo2. Wenn der Benutzer auf die Karte klickt, setzt das Plugin die plotId-Property der Plugin-Component, die in components/d4_plugin_component.qml definiert ist

```qml
// wenn Sie auf ein Plotfeld klicken
// (um Zeile 76)
pluginLoader.active = true
pluginLoader.item.plotId = feature.attribute("plot_id")
```

## 2. d4_plugin_component.qml hat sich geändert. Es hat jetzt 2 eigene Loader.  

- Ein Loader lädt die d4_titlebar.qml, die wir ignorieren werden. Diese enthält den Titeltext und die Schließen-Schaltfläche. Es ist im Grunde unsere Component in Demo2, aber verkleinert. Wir werden das ignorieren.

- Der zweite Loader ist der, der uns interessiert. Er lädt das d4_tabwidget. Wie bei der Titlebar wird beim Laden des Widgets die plotId an das Tabwidget weitergegeben.

```qml
    // TabWidget zur Anzeige der Suchergebnisse
    Loader {
        id: tabWidgetLoader
        // Definieren Sie die plotId-Property hier, um zu den Loader-Kindern zu propagieren
        property string plotId: pluginFrame.plotId
        width: pluginFrame.width  // Verwenden Sie die Breite des Parents (Column)
        height: pluginFrame.height - titleBarLoader.height - 20 // Füllen Sie den verbleibenden Column-Raum
        source: "d4_tabwidget.qml"
    }
```
### Wichtiger Exkurs: Propagierung von plotId mit dynamischen Bindungen

Innerhalb des d4_tabwidget ist die plotId dynamisch an die ID des Parents gebunden. Sie kann aber nur auf ihren Parent verweisen.

```qml
property string plotId: parent.plotId
```

Sie kann nicht den pluginFrame nach Namen referenzieren.

```qml
// nein. funktioniert nicht.
property string plotId: pluginFrame.plotId
```

Sie hat nur einen Verweis auf ihren Parent, der tabWidgetLoader ist. Also müssen Sie plotId binden, indem Sie es im Loader definieren.

```qml
    // TabWidget zur Anzeige der Suchergebnisse
    Loader {
        id: tabWidgetLoader
        // Definieren Sie die plotId-Property hier, um zu den Loader-Kindern zu propagieren
        property string plotId: pluginFrame.plotId
        <...>
        source: "d4_tabwidget.qml"
    }
```

Alle Properties, die Sie übergeben möchten, sollten auf diese Weise verknüpft werden.

## 3. d4_tabwidget.qml lädt die Header-Seite

Das Tab-Widget definiert eine TabBar und eine SwipeView, die miteinander synchronisiert sind. Die SwipeView wischt zwischen 6 Seiten. Die letzten 5 Seiten sind Strata-Seiten. Hier werden die Arten inventarisiert. Die 1. Seite der SwipeView ist die, die uns interessiert. Es ist der headerPageLoader, der in eine ScrollView eingebettet ist. Der headerPageLoader lädt d4_headerpage.qml. Beachten Sie die Propagierung von plotId vom tabWidget durch zum d4_headerpage über dynamische Bindung.

```qml
    TabBar {
        <...> 
    } // tabBar
    SwipeView {
        id: swipeView
        <...>    
        // Header Tab Content (erstes Item)
        ScrollView {
            id: headerScrollView
            <...>
            
            Loader {
                id: headerPageLoader
                property string plotId: tabWidget.plotId
                source: "d4_headerpage.qml"
                width: headerScrollView.availableWidth
            }
        }

        // Strata Tab Content (verbleibende Items.)
        Repeater {
            id: strataRepeater
            <...>
        }       
    }
```

## 4. Und das bringt uns zur d4_headerpage, die das Formular in der ersten Seite des tabWidget lädt

## 📚 **[Die Header-Seite](DEMO4_HEADERPAGE.md)**
## 📚 **[<< Demonstration 4 Einführung](DEMO4_INTRO.md)**
````