````markdown
# Erweiterung der Plugin-Component zur Verwendung des ausgewählten Plots

Dieser Teil ist fast pythonisch einfach.

## Senden der Plot-ID an die Component

In demo2_selection.qml hat unser pointHandler die Plot-ID an die Plugin-Component gesendet, indem er sie an die setPlotId-Funktion übergeben hat.

- Wir haben den Plot-ID-Wert mit der QgsFeature invokable attribute-Funktion erhalten
- Die item-Property des pluginLoaders enthält das Root-Element der Plugin-Component, in diesem Fall d2_plugin_component.pluginFrame
- setPlotId ist eine Funktion, die zu pluginFrame hinzugefügt wurde.

```qml

if (shouldHandle){
    <...>
    if (it.hasNext()){
        const feature = it.next()
        console.log(feature.id)
        it.close()
        pluginLoader.active = true
        // Übergeben Sie die Plot-ID an die Plugin-Component
        pluginLoader.item.plotId = feature.attribute("plot_id")
        return true
    }
}
```

## Handhabung der Plot-ID in demo2_plugin_component

Die Funktion setPlotId setzt einfach den Text in der MessageBox. Ba Da Bing.

```qml
  Rectantgle{
    id: pluginFrame

    // Plot ID Property - von Parent gesetzt, propagiert über Bindungen
    property string plotId: ""

    Rectangle{
        id: messageBoxFrame
        Text{
            id: messageBox

            text: "Plot loaded: " + plotId
        }
    }
  }
```
## 📚 **[Schließen des Plugins](DEMO2_CLOSE.md)**
## 📚 **[<< Demo2 Map-Klick-Logik](DEMO2_MAP_CLICK.md)**
````