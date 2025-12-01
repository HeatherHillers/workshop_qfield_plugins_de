# Heather explains enough QML to get us through the first demo
## 1. The main qml file

This is the main qml file, which loads and unloads the plugin code and it's toolbar buttons.

### Key Points to understand:

1. How does QML work

QML is basically a transformation of the Qt .ui layout file into a programming language, with some javascript smooshed in to give extra functionality.  A qml app doesn't think of itself as a program.  It thinks of itself as a user interface that does stuff.  So if you start out as a programmer used to building QGIS plugins which are programs with user interfaces, this is going to be frustrating.  But imagine the xml in a ui file, and then imagine that it was transformed into a different syntax, and this qml file structure will start to make more sense.

### QML vs Qt .ui (XML) side-by-side comparison:

<div style="display: flex; gap: 20px;">
<div style="flex: 1;">

**QML Code**

```qml
import QtQuick 
import QtQuick.Controls 
import QtQuick.Layouts  
import QtCore
import org.qfield  
import org.qgis
import Theme  
import "qrc:/qml" as QFieldItems

Item {  
  id: plugin
  parent: iface.mapCanvas() 
  anchors.fill: parent 

  Loader {
    id: pluginLoader
    active: false
    anchors.fill: parent
    source: Qt.resolvedUrl(
      './components/d1_plugin_component.qml'
    )
  }  

  QfToolButton {
    id: pluginButton
    bgcolor: Theme.darkGray
    iconSource: Theme.getThemeVectorIcon(
      'ic_camera_photo_black_24dp'
    )
    iconColor: Theme.mainColor
    round: true

    onClicked: {
      iface.logMessage(
        "Loading d1_plugin_component.qml"
      )
      pluginLoader.active = 
        !(pluginLoader.active)
    }
  }

  Component.onCompleted: {
    iface.addItemToPluginsToolbar(
      pluginButton
    )
  }
} 
```

</div>
<div style="flex: 1;">

**Approximate Qt .ui (XML) equivalent**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <widget class="QWidget" name="plugin">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>600</height>
   </rect>
  </property>
  <layout class="QVBoxLayout">
   <item>
    <widget class="QWidget" name="pluginLoader">
     <property name="visible">
      <bool>false</bool>
     </property>
     <!-- Dynamic loading needs code -->
    </widget>
   </item>
   <item>
    <widget class="QPushButton" name="pluginButton">
     <property name="text">
      <string></string>
     </property>
     <property name="icon">
      <iconset theme="camera">
       <normaloff>
         :/icons/ic_camera_photo_black_24dp
       </normaloff>
      </iconset>
     </property>
     <property name="styleSheet">
      <string>
        background-color: rgb(64, 64, 64); 
        border-radius: 20px;
      </string>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <!-- No Component.onCompleted -->
 <!-- No onClicked logic -->
 <!-- Would need Python/C++ code -->
</ui>
```

</div>
</div>
**Similar differences**

1. **id/name identifier** : the id tags in the qml are your variable names, equivalent to the name tag in the ui file. (objectName in the Qt Designer).
2. **layout/anchor**: the anchor property of the widget defines the geometry of the object, similar to the layout and geometry in the ui file.  anchors are usually defined in reference to the parent widget.  The primary difference here is that anchors work more like css.
3. **property expressions/signals and slots**: In Qt Designer, you have the ability to set up a signal/slot connection between 2 widgets that is then stored in the ui file.  But these are very basic.  QML uses javascript to implement complex functionality you would normally put in your python code. (ex. the onClicked slot in the QfToolButton)

**The QML adds:**

1. **Loaders for dynamic widget construction**: Loaders construct and destruct their components when their "active" property value is toggled, allowing runtime construction and destruction.  They also have the nice effect of allowing us to load a component defined in an external file.
2. **Javascript and access to external modules, like the QGIS API**: Anything you can't do with layout manipulation, you do with Javascript.

## Objects in our demo

![alt text](image.png)


### Item (plugin):

- Item is our root object.  Everything in the plugin must be defined inside this object.  
- Item is a base parent class equivalent to QtWidget.  All other widgets inherit from it.

#### Properties
- **id**: this is the object name ("plugin").  This property is available to all qml objects.  Freakishly though, it is optional.  And it is often left out for Items that aren't referenced programatically (like labels)
- **parent**: just like in pyqt, this is a reference to the parent widget.  It is only necessary to define parent in the root Item.  For everything inside the item, the parent is inherently defined by the nested structure.  It is absolutely necessary to define the parent on the root item as the map canvas, because otherwise it will have no parent and no place in QFields user interface, and will not be rendered. 
- **anchors**: this is a very css like mechanism that defines the geometry of the widget.  Usually the geometry is defined in relation to the parent.  anchors.fill: parent tells the widget to expand to cover the whole parent Widget.  So the Item anchor tells us that this plugin ui will cover the entire map canvas - so the entire screen.

#### Members:
- **Loader**: The Loader loads and unloads the actual plugin object.
- **QfToolButton**: This is the button that is clicked to open and close the plugin.
- **Component**: This is a Signal/Slot connection that triggers when the Plugin has finished loading.


### Loader (pluginLoader):

A Loader is an Item that dynamically loads and unloads other Items.  

Our plugin will do things like read layers in the project.  We dont want this to happen until the project is loaded.  In fact, we want our plugin code to wait to load until the user asks for it by clicking on the plugin's tool button.  Writing this code without a Loader would be like writing all our QGIS plugin code in our Plugin's constructor, making it run right when QGIS starts.  Like fools!

The Loader will construct its code when it's "active" property is set to true, and destruct its code when it's "active" property is set to false. This makes triggering the plugin load with a button possible.

#### Properties
- **id and anchors**: because it is an Item widget
- **not parent**: because parent is implicitly defined from the nesting.  the plugin Item is its parent.
- **active**: the boolean property that determines if the Loader's code is constructed or not.  The tool button will load and unload the code by changing the value of this property.
- **source**: the path to the qml file containing the items to be loaded.  
              The source is refered to as a source Component, even though when you look in the qml file you will see Items, not Components.  
              This is because the loader is internally creating a Component object who's job it is to load the Items in the qml file referenced by source.
        
### QfToolButton (pluginButton)

This is easy.  QfToolButton is an Tool Button widget, which we have connected to the Loader.  We could use the QToolButton, but we use the QfToolButton from QField here, so that it keeps QField's design.  For the rest of the plugin I tend to favor the Qt objects.  

I can use QfToolButton or any other QField Widget because I imported them from org.qfield.

This Item is not in the Loader because it is the Item that controls the Loader.

#### Properties

- **id**:
- **round**: A property of QfToolButton.  Makes the button a circle.
- **iconSource**: A property of QfToolButton.  Out of laziness, I used one of the predefined icons in the QField Theme Module.  When I make this productive, I will have to swap out for my own icon because a camera icon makes no sense.  
- **iconColor and bgcolor**: I have taken defined colors from the imported Theme module, which is a good idea, if you want your colors to match QField's colors. 
- **onClicked** : Is a signal handler containing javascript.  When the button is clicked, it prints a message to the log (which is the QField log that the user sees) and it toggles the pluginLoader's boolean active property which triggers either a construction or destruction of the plugin Component (collection of Items in the component QML file)

### Component.onCompleted

This is the signal slot construction that triggers adding the pluginButton to QField's Toolbar when the Plugin Item has been built.

Looking at the Component.onCompleted Member of Item, and also considering that the Loader loads a Component, you may be tricked into thinking that Component is the parent class of Item, and this signal is inherited.  

It is **Not**.  Component and Item dont have anything to do with each other. 

- Component is an "attached property mechanism".  What does that mean?  I think of it as a utility class, which includes the functionality to add special (but not all) signal/slot mechanisms and also includes the functionality of packaging Items in a way that Loader can use them.

- In our Item, the Component "utility" is used to create a signal slot connection that executes when the Item finishes loading.  this line is roughly equivalent to plugin.onCompleted.connect(addItemstoToolbar) in pyQt.

**Why did I use Component.onCompleted in the Item, but a plain onClicked as in the QfToolButton?**

Was I being lazy and inconsistent?  **No!**
The onClicked in QfToolButton is a regular signal/slot construction.

Component.onCompleted is a "lifecycle hook" which is exactly like a signal/slot construction but Qt decided it needed a completely different implementation.

Conversely, the onClicked signal/slot construction in QfToolButton has nothing to do with the Component class.  

Component.onCompleted is a special signal/slot that is only fired after construction of the Item is complete.  It doesn't belong to Item, but is bubblegummed on by the Component utility.

Yes. It is confusing. And bad design. I think. Maybe someone has a clever design justification for it.

Luckily, Component only attaches 2 signals, so its easy to remember:
 - onCompleted (the object finished loading)
 - onDestruction (like close on a QtWidget)

 Every other signal/slot uses the normal Item construction like the onClicked in QfToolButton.

 Now we feel a bit better about that, I hope.

# We are ready to look at the d1_plugin_component.

That was a lot of theory, I know.  But I think we have now covered enough of the QML structure that the rest will go a lot faster.

Lets look at that plugin code that is loaded by the pluginLoader.  

It is pretty simple.  Just a vanilla screen with a label, that expands to the screen.

```qml
  /*
    This plugin component is just a rectangle.
    It's size is the full size of it's parent widget, which
    is the map canvas.
    The Rectangle contains a Text Component with a title string.
    */
 
import QtQuick 
import QtQuick.Controls 
import QtQuick.Layouts  

import QtCore

import org.qfield  
import org.qgis
import Theme  

import "qrc:/qml" as QFieldItems
Rectangle {
    id: pluginFrame
    anchors.fill: parent
    property color background_color: "#ffecd1"
    property color text_color: "#6baa75"
    color: background_color
    Text {
        text: "Vegetation Monitoring: Plugin Component"
        color: pluginFrame.text_color
        font.pixelSize: 20
        horizontalAlignment: Text.AlignHCenter             
        anchors.centerIn: parent
    }
    Component.onCompleted: {
        //setup menu list models
        iface.logMessage("d1_plugin_component.qml loaded")
    }
}

```
Here you don't have an Item but a Rectangle, which is an Item, just like a QtFrame is a QtWidget.

QObject (C++ base class for all Qt objects)
│
└── QtObject (QML base type)
    │
    ├── Item (base for VISUAL QML types)
    │   │
    │   ├── Rectangle (visual shape with fill/border)
    │   │
    │   ├── Text (displays text)
    │
    └── Component 

This looks a lot more familiar now that we have slogged through the main qml Item.  I'll just point out the main features.

### Custom Properties (are just variables)

We know about properties like id and anchors.  They belong to the Item class.  You can also define your own custom properties.  I have 2 in the Rectangle.  

    - property color background_color: "#ffecd1"
    - property color text_color: "#6baa75"

These are constants I use to consistently apply my colors in the child widgets.  I only use them once here, but in more complicated demos this will be more useful.

### Text vs QfText

Like QfToolButton, I also would have the option to use a QfText here.  But I have decided to use the plain Qt Items beyond the main code file.  Why?

- The QField Items are good if you want your plugin to look like a QField component, like it is an integrated part of QField.
- But I need to make my plugin visually distinct.  So I'm going my own way.
- Also, If I stick with the Qt Items, I only have to learn about the Qt Items.


### Other things to remember
- **anchors**: pay attention to your anchors. Forgetting them or misconfiguring them causes a lot of errors.


## Next Step

Wir sind jetzt bereit für [Demo2: Search Bar](../docs/demo2/README.md)


