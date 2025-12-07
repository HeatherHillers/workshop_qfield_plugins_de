# d1_plugin_component.qml

That was a lot of theory, I know.  But I think we have now covered enough of the QML structure that the rest will go a lot faster.  

Lets look at that plugin code that is loaded by the pluginLoader.  

It is pretty simple.  Just a vanilla screen with a label, that expands to the screen.

```qml
< ... Imports ...>

Rectangle {
    id: pluginFrame
    anchors.fill: parent
    color: PluginTheme.vanilla
    Text {
        text: "Demo 1 Vegetation Monitoring: Plugin Component"
        color: PluginTheme.green
        font.pixelSize: 20
        horizontalAlignment: Text.AlignHCenter             
        anchors.centerIn: parent
    }
    Component.onCompleted: {
        iface.logMessage("d1_plugin_component.qml loaded")
    }
}

```
Here you don't have an Item but a Rectangle, which is an Item, just like a QtFrame is a QtWidget.  

![alt text](img/d1_comp_obj.png)  


## A QML Text Box

Our vanilla screen with a label is just a text box.  There is no Text Box in QML.   We need 2 Objects: Rectangle and Text.  

### Text vs QfText

Like QfToolButton, I also would have the option to use a QfText here.  But I have decided to use the plain Qt Items beyond the main code file.  Why?

- The QField Items are good if you want your plugin to look like a QField component, like it is an integrated part of QField.
- But I need to make my plugin visually distinct.  So I'm going my own way.
- Also, If I stick with the Qt Items, I only have to learn about the Qt Items.

## Using PluginTheme

We have discussed in [Plugin Structure](DEMO1_STRUCTURE.md) the definition of PluginTheme as a configuration object that holds our color palette.  In d1_plugin_component, you will see that the color of the Rectangle is set to PluginTheme.vanilla, and the color of the Text is set to PluginTheme.green.  Easy Peasy.


## 📚 **[>> Demo2: Feature Selection!](../demo2/DEMO2_INTRO.md)**
## 📚 **[<< Main Module](DEMO1_MAIN.md)**

