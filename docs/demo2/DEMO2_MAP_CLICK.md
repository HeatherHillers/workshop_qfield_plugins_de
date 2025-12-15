# Setting up Feature selection from the map canvas

To set up feature selection from the map canvas, we have to register a QField pointHandler, much like a QgsMapTool in pyQGIS.  This is done in the main code file demo2_selection.qml.  

In doing this bit we are also going to learn:

- how to grab a layer from a project
- how to grab a feature with an expression
- how to grab an attribute from a feature

## What are we doing?

- get a point handler
- retrieve the clicked map coordinates from the point handler
- draw a 20m bounding box around the map coordinates.
- use a spatial query to find features on the plot layer that are inside the bounding box.
- if a feature is found, activate the plugin and pass the feature's plot id to the plugin component.

## 1. Retrieve the point handler from the interface

Retrieve and store a reference the the QField Interface's pointHandler as a class property.

```qml
Item{
    id: plugin
    property var pointHandler: iface.findItemByObjectName("pointHandler")

}
```

- iface is your QField interface instance, like qgis.utils.iface in pyqgis.  It is imported from org.qfield.
- we have already seen the use of iface.logMessage and iface.addItemToPluginsToolbar
- iface.findItemByObjectName is a QField interface function that retrieves any object from the QField instance, provided it has an objectName property.  (This is where you start digging through the source code.)
- pointHandler is a QField interface object that emits the coordinates of a map interaction and what kind of an interaction it was (click, double click, hold and press)

## 2. When the plugin loads, (when your project opens) register your custom callback function for the QField point handler

The root Item is your plugin.  It's Component.onCompleted function fires when the plugin loads.

```qml
Item{
    id: plugin
    property var pointHandler: iface.findItemByObjectName("pointHandler")
    Component.onCompleted{
        pointHandler.registerHandler("demo2_searchbar", my_callback);
    }
}
```

## 3. When the plugin unloads (when your project closes), deregister your callback function!

If you don't, your plugin is going to contaminate your other projects. 

Use Component.onDestruction to define behavior on close.

```qml
Item{
    id: plugin
    <...>
    Component.onDestruction{
        pointHandler.deregisterHandler("demo2_searchbar");
    }
}
```

## 4. Define your callback, using an arrow function

Our callback function is written in Javascript.  Instead of using a function reference like below, as a Python programmer would:

```qml
Item{
    id: plugin
    <...>
    Component.onCompleted{
        pointHandler.registerHandler("demo2_searchbar", my_callback);
    }
}
```

it is more common to see a Javascript arrow function syntax, as we will see in demo2_selection

```qml
Item{

    Component.onCompleted{
        pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType)=>{
            iface.logMessage("Interaction Type: " + interactionType)
            return true
        });
    }
}

```
### Do not forget the boolean return on your callback
The boolean return value from the pointHandler callback tells QField whether your handler consumed the event or not:

return true - Event consumed

Your handler processed the click
- QField should stop propagating the event to other handlers
- Prevents default QField behavior for this click

return false or no return - Event not consumed

- Your handler didn't process the click (or wants to pass it through)
- QField continues propagating to other registered handlers
- Default QField behavior can still execute

## 5. Define the callback to get map coordinates from the user interaction

### Decide which interaction you want

- "clicked" :   
  Triggering on single click can cause conflict, because the Feature Drawer opens on a single click.  
  - In QField for Windows, single click will block the Feature Drawer and replace it with our plugin.   
  - In QField for iOS, if you use a single click, the Attribute Table will be rendered on top of the plugin.

- "doubleClicked" :  
  Triggering on double click will prevent conflict.  We can allow the Feature Drawer to open on a single click, and enter the plugin with a double click.  
  - In QField for Windows (at the moment) the double click interaction doesn't register. 
  - In iOS, this works great.
- "pressAndHold":  
   This would theoretically be a nice mode for opening our plugin as well, but in practice it is a bad choice
  - In iOS, the context menu that opens on press blocks our plugin from opening on this interaction.
  - In Windows, the press and hold interaction doesn't register.

Does this mean that the boolean return isn't quite working as advertised?  Maybe.  There is also a priority set on point handler callbacks, which may influence the processing of the boolean return from the callback.

To avoid conflicts with QField and ensure our plugin works in both environments, use clicked if we are in Windows, and doubleClicked if we are in iOS.

### Dont forget your return

- return true after handling your point, in order to block other handlers like the QField Attribute Table from firing.
- return false if you are ignoring the interaction, so that the next handler can pick it up.

```qml

Item{
    Component.onCompleted{
        pointHandler.registerHandler("demo2_selection", (point, type, interactionType) => {
            var shouldHandle = (Qt.platform.os === "windows" && interactionType === "clicked") ||
                               (Qt.platform.os !== "windows" && interactionType === "doubleClicked")
            if (shouldHandle) {
                iface.logMessage("Platform " + Qt.platform.os)
                iface.logMessage("Interaction " + interactionType)      
                return true // block further processing of the click
            }
            return false // let the interaction be picked up by QField
        });
    }
}

```
### Get the pixel coordinates of the screen interaction

```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      iface.logMessage(point.x)
      iface.logMessage(point.y)
    }
});
```

### Convert them into map coordinates
```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      let coords = iface.mapCanvas().mapSettings.screenToCoordinate(Qt.point(point.x, point.y))
      iface.logMessage(coords.x)
      iface.logMessage(coords.y)
    }
});
```

## 6. Define your callback to run a spatial query on the plot layer using your user coordinates.

We are going to run a spatial query on the plots layer using a bounding box of 20m around our clicked coordinates.  If we find a feature in that box, we are going to activate the plugin and pass the plot id of the feature to our plugin component.

### Retrieve the plots layer from the project

You can call a subset of the QGIS API functions from QField and Javascript.  You can find out which functions can be called from QField by going to the QGIS C++ Class Reference for the class you are interested in.  Functions you can call are tagged with Q_INVOKABLE.

![alt text](img/demo2_reference.png)

We pull the map player from the QgsProject instance, which is available in QField as qgisProject, which was imported with org.qgis.

```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      let layer = qgisProject.mapLayersByName("plots")[0]
      iface.logMessage("Got plots layer")
    }
});
```

### Retrieve a feature from the plots layer with a LayerUtils feature iterator using a simple expression

getFeatures is not yet an invokable qgis function.  Instead we can pass an expression to get a feature iterator from QField's LayerUtils class, which is imported from org.qfield.  

We will query our plots layer for the feature with plot_id = 'b.1'.

We will retrieve this feature and print its plot id.  (Not using our coordinates yet.)

```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      var layer = qgisProject.mapLayersByName("plots")[0]
      var expression = "plot_id = 'b.1'";

      let it = LayerUtils.createFeatureIteratorFromExpression(layer, expression)
      
      if (it.hasNext()){
        feature = it.next()
        plot_id = feature.attribute("plot_id")
        iface.logMessage("found the feature of the plots layer: " + plot_id)
      }
      it.close(); // NEVER forget to close your iterator
      return true
    }
    return false

});
```
#### IMPORTANT:  Do not ever ever forget to close a feature iterator.  If you do not close your feature iterator, it will lead to a complete shut down of QField, after about the 4th time you call the feature iterator.  


### Now swap out the simple expression with a spatial query

We will create a bounding box of 20m around our interaction coordinates.  
We will use the bounding box coordinates to pass an intersection query to LayerUtils instead of our simple query.

```qml

      if (shouldHandle) {
        // create a pair of point that'll represent a buffer area within which features are to be searched. 
        let tl = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x - 20, point.y - 20))
        let br = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x + 20, point.y + 20))

        let expression = "intersects(geom_from_wkt('POLYGON(("+tl.x+" "+tl.y+", "+br.x+" "+tl.y+", "+br.x+" "+br.y+", "+tl.x+" "+br.y+", "+tl.x+" "+tl.y+"))'), $geometry)"
        let it = LayerUtils.createFeatureIteratorFromExpression(qgisProject.mapLayersByName("plots")[0], expression)
        if (it.hasNext()) {
          const feature = it.next()
          console.log(feature.id + " " + feature.attribute("plot_id"))
        }
        it.close();
      }
      return false
    
```

### Finally, if you find the feature, turn on the plugin and pass the plot id.

The Loader class has the property "item", which contains the Item loaded by its source Component.

In our plugin component, we have defined a function setPlotId, which will receive a plot id and use it to populate the plugin's search bar and text box.

**Remember to close the iterator whether or not the feature is found!**

**Remember to return the boolean for the pointHandler. **

```qml
    pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
      // ...
      if (shouldHandle) {
        // ...
        if (it.hasNext()) {
          // get your feature
          const feature = it.next()

          // close your iterator
          it.close()  
          
          // turn on the plugin, just as the camera button would
          pluginLoader.active = true
          
          // pass the plot id to the plugin component
          pluginLoader.item.setPlotId(feature.attribute("plot_id"))
          
          // block the interaction signal
          return true
        }
        // close the iterator if you dont find a feature
        it.close();
      }
      // pass on the interaction
      return false
    });
```

## The whole demo2_searchbar.qml

```qml
 // imports

Item {
  id: plugin
  parent: iface.mapCanvas() 
  anchors.fill: parent 
  
  // Map Selection: 1. Hold a reference to the map canvas
  property var mapCanvas: iface.mapCanvas() 
  
  // Map Selection: 2. add the pointHandler to the plugin
  property var pointHandler: iface.findItemByObjectName("pointHandler")

  Loader {
    id: pluginLoader
    // ...
  }  


  Component.onCompleted: {
 
    // Map Selection: 3. register the point handler and define its callback
    pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {

      // decide on the interaction  
      var shouldHandle = (Qt.platform.os === "windows" && interactionType === "clicked") ||
                         (Qt.platform.os !== "windows" && interactionType === "doubleClicked")
      if (shouldHandle) {

        // create a pair of points that will represent a 20m buffer area within which features are to be searched. 
        let tl = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x - 20, point.y - 20))
        let br = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x + 20, point.y + 20))

        // run a spatial query
        let expression = "intersects(geom_from_wkt('POLYGON(("+tl.x+" "+tl.y+", "+br.x+" "+tl.y+", "+br.x+" "+br.y+", "+tl.x+" "+br.y+", "+tl.x+" "+tl.y+"))'), $geometry)"
        let it = LayerUtils.createFeatureIteratorFromExpression(qgisProject.mapLayersByName("plots")[0], expression)
        if (it.hasNext()) {
          // you've got a feature, play with it! :)
          const feature = it.next()
          console.log(feature.id)
          it.close()
          pluginLoader.active = true
          // pass the plot id to the plugin component
          pluginLoader.item.setPlotId(feature.attribute("plot_id"))
          return true
        }
        it.close();
      }
      return false
    });
  }

  // Map Selection: 4. Deregister the point handler on destruction (should be on project close)
  Component.onDestruction: {
    pointHandler.deregisterHandler("demo2_searchbar");
  }
} 

```

### Side note: console.log

In demo 1 I used iface.logMessage to print debug statements for the user to see.
In demo 2 I have swapped them out with console.log statements.  These print to the terminal.  You can see them if you start qfield from a dos or bash shell, but they wont show up in the user logs.  This is generally nicer for the developer.

That was a lot.  Lets take a look at how that plot id gets to the messageBox. This part is pretty easy.

## 📚 **[Handling the plot id in the Plugin Component](DEMO2_PLOTID.md)**
## 📚 **[<< Demo2 Introduction](DEMO2_INTRO.md)**
