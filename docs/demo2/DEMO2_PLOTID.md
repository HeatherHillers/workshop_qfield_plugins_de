# Expanding the Plugin Component to Use the Selected Plot

This part is almost pythonically simple.

## Sending the plot id to the component.

In demo2_selection.qml, our pointHandler sent the plot id to the plugin component by passing it to the setPlotId function.

- we got the plot id value using the QgsFeature invokable attribute function
- pluginLoader's item property holds the root element of the plugin component, in this case d2_plugin_component.pluginFrame
- setPlotId is a function that has been added to pluginFrame.

```qml

if (shouldHandle){
    if (it.hasNext()){
        const feature = it.next()
          console.log(feature.id)
          it.close()
          pluginLoader.active = true
          // pass the plot id to the plugin component
          pluginLoader.item.setPlotId(feature.attribute("plot_id"))
          return true
    }
}
```

## Handling plot id in demo2_plugin_component

the function setPlotId simply sets the text in the message box. Ba Da Bing.

```qml
  Rectantgle{
    id: pluginFrame
    function setPlotId(plotId) {
        messageBox.text = "Plot loaded: " + plotId
        console.log("component loaded with plot ID:", plotId)
    }
    Rectangle{
        id: messageBoxFrame
        Text{
            id: messageBox
        }
    }
  }
```
## 📚 **[Closing the Plugin](DEMO2_CLOSE.md)**
## 📚 **[<< Demo2 Map Click Logic](DEMO2_MAP_CLICK.md)**