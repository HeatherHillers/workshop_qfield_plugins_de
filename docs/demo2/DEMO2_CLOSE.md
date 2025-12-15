# Closing the Plugin: Sending signals

Instead of using the button in the PluginToolbar to turn the plugin on and off, we now rely on the map click to open the plugin, and I have added an Ok button to the plugin component to close it.  This creates a simple example of creating and using custom signals.


## Setting up the Signal from the component in d2_plugin_compontent.qml

Just as you would in pyQt, our signal is defined inside our component item

```qml
Rectangle {
  id: pluginFrame
  signal closed()
}
```

We then add a button to the pluginFrame which emits the signal when it is clicked

```qml
Rectangle {
  id: pluginFrame
  signal closed()
  Button {
    id: closeButton
    onClicked: {
        closed()
    }
  }
}
```

## Connecting the signal in the plugin

In demo2_select.qml, we have added a Connections item to connect the signal from pluginFrame.  This is the 3rd distinct method for handling signals and slots in QML.

```qml

Item {
  id: plugin
  Loader{
    id:pluginLoader
  }
  Connections {
    target: pluginLoader.item
    function onClosed(){
      pluginLoader.active = false
    }
  }
}
```

This is equivalent to the pyQt

```python
 pluginLoader.item.closed.connect(onClosed)
 def onClosed(){
    pluginLoader.active=false
 }
```

Note here that the connection between slot and function follows a mandatory naming convention whereby if the signal name is closed, the slot name must be onClosed.  Hey. I don't make the rules.

I could tell you about Layouts, but I think you will be very bored if I do.  So let's skip to the next demo and I'll show you how to add and update a feature.

## 📚 **[Demonstration 3: Adding and Updating a Feature, and a cool Tab Widget.](../demo3/DEMO3_INTRO.md)**
## 📚 **[<< Demo2 Handling the Plot id](DEMO2_MAP_CLICK.md)**