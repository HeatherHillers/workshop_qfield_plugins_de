````markdown
# Einrichtung der Feature-Auswahl vom Map-Canvas

Um die Feature-Auswahl vom Map-Canvas einzurichten, müssen wir einen QField pointHandler registrieren, ähnlich wie ein QgsMapTool in pyQGIS. Dies geschieht in der Hauptcode-Datei demo2_selection.qml.  

Dabei lernen wir auch:

- wie man einen Layer aus einem Projekt erhält
- wie man ein Feature mit einem Ausdruck erhält
- wie man ein Attribut aus einem Feature erhält

## Was machen wir?

- einen Point-Handler abrufen
- die geklickten Kartenkoordinaten vom Point-Handler abrufen
- ein 20m-Begrenzungsrechteck um die Kartenkoordinaten zeichnen
- eine räumliche Abfrage verwenden, um Features auf dem Plot-Layer zu finden, die sich innerhalb des Begrenzungsrechtecks befinden
- wenn ein Feature gefunden wird, das Plugin aktivieren und die Plot-ID des Features an die Plugin-Component übergeben

## 1. Den Point-Handler von der Schnittstelle abrufen

Abrufen und speichern Sie einen Verweis auf den pointHandler der QField-Schnittstelle als Klassen-Property.

```qml
Item{
    id: plugin
    property var pointHandler: iface.findItemByObjectName("pointHandler")

}
```

- iface ist Ihre QField-Schnittstelleninstanz, wie qgis.utils.iface in pyqgis. Sie wird aus org.qfield importiert.
- Wir haben bereits die Verwendung von iface.logMessage und iface.addItemToPluginsToolbar gesehen
- iface.findItemByObjectName ist eine QField-Schnittstellenfunktion, die jedes Objekt aus der QField-Instanz abruft, sofern es eine objectName-Property hat. (Hier beginnen Sie, durch den Quellcode zu graben.)
- pointHandler ist ein QField-Schnittstellenobjekt, das die Koordinaten einer Karteninteraktion und die Art der Interaktion (Klick, Doppelklick, Halten und Drücken) ausgibt

## 2. Wenn das Plugin lädt (wenn Ihr Projekt öffnet), registrieren Sie Ihre benutzerdefinierte Callback-Funktion für den QField Point-Handler

Die onCompleted-Funktion des Root-Items wird ausgelöst, wenn das Plugin lädt.

```qml
Item{
    id: plugin
    property var pointHandler: iface.findItemByObjectName("pointHandler")
    Component.onCompleted{
        pointHandler.registerHandler("demo2_selection", my_callback);
    }
}
```

## 3. Wenn das Plugin entladen wird (wenn Ihr Projekt schließt), deregistrieren Sie Ihre Callback-Funktion!

Wenn Sie das nicht tun, wird Ihr Point-Handler Ihre anderen Projekte kontaminieren. 

Verwenden Sie Component.onDestruction, um Verhalten beim Schließen zu definieren.

```qml
Item{
    id: plugin
    <...>
    Component.onDestruction{
        pointHandler.deregisterHandler("demo2_selection");
    }
}
```

## 4. Definieren Sie Ihren Callback mit einer Arrow-Funktion

Unsere Callback-Funktion ist in JavaScript geschrieben. Anstatt eine Funktionsreferenz wie unten zu verwenden, wie es ein Python-Programmierer tun würde:

```qml
Item{
    id: plugin
    <...>
    Component.onCompleted{
        pointHandler.registerHandler("demo2_selection", my_callback);
    }
}
```

ist es üblicher, eine JavaScript-Arrow-Funktions-Syntax zu sehen, wie wir sie in demo2_selection sehen werden

```qml
Item{

    Component.onCompleted{
        pointHandler.registerHandler("demo2_selection", (point, type, interactionType)=>{
            iface.logMessage("Interaction Type: " + interactionType)
            return true
        });
    }
}

```
### Vergessen Sie nicht den booleschen Rückgabewert Ihres Callbacks
Der boolesche Rückgabewert vom pointHandler-Callback teilt QField mit, ob Ihr Handler das Event verbraucht hat oder nicht:

return true - Event verbraucht

Ihr Handler hat den Klick verarbeitet
- QField sollte aufhören, das Event an andere Handler weiterzugeben
- Verhindert QFields Standardverhalten für diesen Klick

return false oder kein return - Event nicht verbraucht

- Ihr Handler hat den Klick nicht verarbeitet (oder möchte ihn durchgeben)
- QField gibt weiter an andere registrierte Handler
- QFields Standardverhalten kann immer noch ausgeführt werden

## 5. Definieren Sie den Callback, um Kartenkoordinaten von der Benutzerinteraktion zu erhalten

### Entscheiden Sie, welche Interaktion Sie wollen

- "clicked" :   
  Das Auslösen bei Einzelklick kann Konflikte verursachen, da die Feature-Drawer bei einem Einzelklick öffnet.  
  - In QField für Windows wird der Einzelklick die Feature-Drawer blockieren und durch unser Plugin ersetzen.   
  - In QField für iOS wird, wenn Sie einen Einzelklick verwenden, die Attributtabelle über dem Plugin gerendert.

- "doubleClicked" :  
  Das Auslösen bei Doppelklick verhindert Konflikte. Wir können die Feature-Drawer bei einem Einzelklick öffnen lassen und mit einem Doppelklick in das Plugin eintreten.  
  - In QField für Windows (im Moment) registriert die Doppelklick-Interaktion nicht. 
  - In iOS funktioniert das großartig.

- "pressAndHold":  
   Dies wäre theoretisch auch ein schöner Modus zum Öffnen unseres Plugins, aber in der Praxis ist es eine schlechte Wahl
  - In iOS blockiert das Kontextmenü, das sich beim Drücken öffnet, das Öffnen unseres Plugins bei dieser Interaktion.
  - In Windows registriert die Press-and-Hold-Interaktion nicht.

Bedeutet das, dass der boolesche Rückgabewert nicht ganz wie angekündigt funktioniert? Vielleicht. Es gibt auch eine Priorität, die auf pointHandler-Callbacks gesetzt wird, die die Verarbeitung des booleschen Rückgabewerts aus dem Callback beeinflussen kann.

Um Konflikte mit QField zu vermeiden und sicherzustellen, dass unser Plugin in beiden Umgebungen funktioniert, verwenden Sie clicked, wenn wir in Windows sind, und doubleClicked, wenn wir in iOS sind.

### Vergessen Sie Ihr Return nicht

- return true nach der Behandlung Ihres Punktes, um andere Handler wie die QField-Attributtabelle am Auslösen zu hindern.
- return false, wenn Sie die Interaktion ignorieren, damit der nächste Handler sie aufnehmen kann.

```qml

Item{
    Component.onCompleted{
        pointHandler.registerHandler("demo2_selection", (point, type, interactionType) => {
            var shouldHandle = (Qt.platform.os === "windows" && interactionType === "clicked") ||
                               (Qt.platform.os !== "windows" && interactionType === "doubleClicked")
            if (shouldHandle) {
                iface.logMessage("Platform " + Qt.platform.os)
                iface.logMessage("Interaction " + interactionType)      
                return true // weitere Verarbeitung des Klicks blockieren
            }
            return false // die Interaktion von QField aufnehmen lassen
        });
    }
}

```
### Die Pixelkoordinaten der Bildschirminteraktion abrufen

```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      iface.logMessage(point.x)
      iface.logMessage(point.y)
    }
});
```

### In Kartenkoordinaten umwandeln
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

## 6. Definieren Sie Ihren Callback, um eine räumliche Abfrage auf dem Plot-Layer mit Ihren Benutzerkoordinaten auszuführen

Wir werden eine räumliche Abfrage auf dem Plots-Layer mit einem Begrenzungsrechteck von 20m um unsere geklickten Koordinaten ausführen. Wenn wir ein Feature in diesem Rechteck finden, werden wir das Plugin aktivieren und die Plot-ID des Features an unsere Plugin-Component übergeben.

### Den Plots-Layer aus dem Projekt abrufen

Sie können eine Teilmenge der QGIS-API-Funktionen von QField und JavaScript aufrufen. Sie können herausfinden, welche Funktionen von QField aufgerufen werden können, indem Sie zur QGIS C++ Class Reference für die Klasse gehen, an der Sie interessiert sind. Funktionen, die Sie aufrufen können, sind mit Q_INVOKABLE gekennzeichnet.

![alt text](img/demo2_reference.png)

Wir holen den Map-Layer aus der QgsProject-Instanz, die in QField als qgisProject verfügbar ist, das mit org.qgis importiert wurde.

```qml

pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {
    var shouldHandle = ... 
    if (shouldHandle){
      let layer = qgisProject.mapLayersByName("plots")[0]
      iface.logMessage("Got plots layer")
    }
});
```

### Ein Feature aus dem Plots-Layer mit einem LayerUtils Feature-Iterator mit einem einfachen Ausdruck abrufen

getFeatures ist noch keine aufrufbare QGIS-Funktion. Stattdessen können wir einen Ausdruck übergeben, um einen Feature-Iterator aus QFields LayerUtils-Klasse zu erhalten, die aus org.qfield importiert wird.  

Wir werden unseren Plots-Layer nach dem Feature mit plot_id = 'b.1' abfragen.

Wir werden dieses Feature abrufen und seine Plot-ID ausgeben. (Noch nicht unsere Koordinaten verwenden.)

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
      it.close(); // NIEMALS vergessen, Ihren Iterator zu schließen
      return true
    }
    return false

});
```
#### WICHTIG: Vergessen Sie niemals, einen Feature-Iterator zu schließen. Wenn Sie Ihren Feature-Iterator nicht schließen, führt dies zu einem vollständigen Herunterfahren von QField, nach etwa dem 4. Mal, dass Sie den Feature-Iterator aufrufen.  


### Tauschen Sie nun den einfachen Ausdruck mit einer räumlichen Abfrage aus

Wir werden ein Begrenzungsrechteck von 20m um unsere Interaktionskoordinaten erstellen.  
Wir werden die Begrenzungsrechteck-Koordinaten verwenden, um eine Schnittmengen-Abfrage an LayerUtils zu übergeben, anstelle unserer einfachen Abfrage.

```qml

      if (shouldHandle) {
        // Erstelle ein Paar von Punkten, die einen Pufferbereich darstellen, in dem Features gesucht werden sollen. 
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

### Schließlich, wenn Sie das Feature finden, schalten Sie das Plugin ein und übergeben Sie die Plot-ID

Die Loader-Klasse hat die Property "item", die das Item enthält, das von seiner Source-Component geladen wurde.

In unserer Plugin-Component haben wir eine Funktion setPlotId definiert, die eine Plot-ID erhält und sie verwendet, um die Suchleiste und Textbox des Plugins zu füllen.

**Denken Sie daran, den Iterator zu schließen, unabhängig davon, ob das Feature gefunden wird oder nicht!**

**Denken Sie daran, den booleschen Wert für den pointHandler zurückzugeben. **

```qml
    pointHandler.registerHandler("demo2_selection", (point, type, interactionType) => {
      // ...
      if (shouldHandle) {
        // ...
        if (it.hasNext()) {
          // Holen Sie Ihr Feature
          const feature = it.next()

          // Schließen Sie Ihren Iterator
          it.close()  
          
          // Schalten Sie das Plugin ein, genau wie die Kamera-Schaltfläche
          pluginLoader.active = true
          
          // Übergeben Sie die Plot-ID an die Plugin-Component
          pluginLoader.item.plotId = feature.attribute("plot_id")
          
          // Blockieren Sie das Interaktionssignal
          return true
        }
        // Schließen Sie den Iterator, wenn Sie kein Feature finden
        it.close();
      }
      // Geben Sie die Interaktion weiter
      return false
    });
```

## Die gesamte demo2_selection.qml

```qml
 // imports

Item {
  id: plugin
  parent: iface.mapCanvas() 
  anchors.fill: parent 
  
  // Map Selection: 1. Halten Sie einen Verweis auf das Map-Canvas
  property var mapCanvas: iface.mapCanvas() 
  
  // Map Selection: 2. Fügen Sie den pointHandler zum Plugin hinzu
  property var pointHandler: iface.findItemByObjectName("pointHandler")

  Loader {
    id: pluginLoader
    // ...
  }  


  Component.onCompleted: {
 
    // Map Selection: 3. Registrieren Sie den Point-Handler und definieren Sie seinen Callback
    pointHandler.registerHandler("demo2_searchbar", (point, type, interactionType) => {

      // Entscheiden Sie über die Interaktion  
      var shouldHandle = (Qt.platform.os === "windows" && interactionType === "clicked") ||
                         (Qt.platform.os !== "windows" && interactionType === "doubleClicked")
      if (shouldHandle) {

        // Erstellen Sie ein Paar von Punkten, die einen 20m-Pufferbereich darstellen, in dem Features gesucht werden sollen. 
        let tl = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x - 20, point.y - 20))
        let br = mapCanvas.mapSettings.screenToCoordinate(Qt.point(point.x + 20, point.y + 20))

        // Führen Sie eine räumliche Abfrage aus
        let expression = "intersects(geom_from_wkt('POLYGON(("+tl.x+" "+tl.y+", "+br.x+" "+tl.y+", "+br.x+" "+br.y+", "+tl.x+" "+br.y+", "+tl.x+" "+tl.y+"))'), $geometry)"
        let it = LayerUtils.createFeatureIteratorFromExpression(qgisProject.mapLayersByName("plots")[0], expression)
        if (it.hasNext()) {
          // Sie haben ein Feature, spielen Sie damit! :)
          const feature = it.next()
          console.log(feature.id)
          it.close()
          pluginLoader.active = true
          // Übergeben Sie die Plot-ID an die Plugin-Component
          pluginLoader.item.plotId = feature.attribute("plot_id")
          return true
        }
        it.close();
      }
      return false
    });
  }

  // Map Selection: 4. Deregistrieren Sie den Point-Handler bei Destruktion (sollte beim Projektschluss sein)
  Component.onDestruction: {
    pointHandler.deregisterHandler("demo2_searchbar");
  }
} 

```

### Randnotiz: console.log

In Demo 1 habe ich iface.logMessage verwendet, um Debug-Anweisungen für den Benutzer auszugeben.
In Demo 2 habe ich sie durch console.log-Anweisungen ausgetauscht. Diese werden auf das Terminal ausgegeben. Sie können sie sehen, wenn Sie QField von einer DOS- oder Bash-Shell starten, aber sie erscheinen nicht in den Benutzerprotokollen. Dies ist im Allgemeinen schöner für den Entwickler.

Das war viel. Schauen wir uns an, wie diese Plot-ID zur MessageBox gelangt. Dieser Teil ist ziemlich einfach.

## 📚 **[Handhabung der Plot-ID in der Plugin-Component](DEMO2_PLOTID.md)**
## 📚 **[<< Demo2 Einführung](DEMO2_INTRO.md)**

````