````markdown
# Demo 2 Searchbar 

# demo2_searchbar.qml

Dieser Hauptcode ist derselbe wie demo1_hello.qml. Wir können ihn überspringen.

# d2_plugin_component.qml

Die Plugin-Component hat einen Loader für die Suchleiste und ein ColumnLayout hinzugefügt, um es schön zu machen.

# QML: Ausrichten von Items mit Layouts

Qt hat sich entschieden, 2 verschiedene Klassen für QML-Layouts zu implementieren. Das ist nicht besonders hilfreich.

## Basis-Positioner-Klassen

- Row
- Column
- Grid
- Flow (tatsächlich nützlich für Textumbruch)

Aber natürlich, sobald Sie versuchen, etwas etwas Ausgeklügeltes mit Ihrer Größenbestimmung zu tun, möchten Sie die fortgeschritteneren verwenden:

## Layout-Klassen

- RowLayout
- ColumnLayout
- Grid Layout

Also ist es am besten, einfach mit den Layout-Klassen zu beginnen.

### Ausrichtungs-Properties für Items in Layout-Klassen

Wenn Sie eine Layout-Klasse verwenden, können Sie Ausrichtungs-Properties auf ihre Kinder anwenden. Diese Properties werden aus QtQuick importiert

- **Qt.AlignLeft** (Standard für Items)
- **Qt.AlignHCenter** (horizontal zentriert)
- **Qt.AlignRight**
- **Qt.AlignTop**
- **Qt.AlignVCenter** (vertikal zentriert)
- **Qt.AlignBottom**

Kann kombiniert werden: Qt.AlignHCenter | Qt.AlignVCenter

## Verwendung von Layout-Klassen im Plugin

- d2_plugin_component.qml verwendet ein ColumnLayout, um die Suchleiste und die Nachrichtenleiste vertikal auszurichten.
   - Die Items werden horizontal in der Spalte mit **Layout.alignment: QAlignHCenter** zentriert
   - Die Items erhalten einen 10%-Rand links und rechts mit **width: pluginFrame * 0.8**

- d2_searchbar.qml verwendet ein RowLayout, um die Label- und ComboBox-Items horizontal auszurichten. Wir werden uns das gleich ansehen.

# QML: Ausrichten von Items mit Anchors

Mit dem ColumnLayout und den Layout-Klassen-Properties haben wir die 2 Items in einer Spalte ausgerichtet und horizontal zentriert.
Allerdings würden wir mit nur diesen Änderungen die Items von oben nach unten ausrichten. Wir haben sie noch nicht vertikal zentriert.

**Layout.alignment: QAlignHCenter | QAlignVCenter** würde nur dazu führen, dass die 2 Items übereinander gerendert werden.

Wir können das Problem lösen, indem wir das ColumnLayout so definieren, dass es in seinem Parent mit anchors.centerIn zentriert wird.  
Also wird die Spaltenhöhe auf die kombinierte Höhe ihrer Kinder dimensioniert, zentriert in Bezug auf ihren Parent.

Ein anderer Ort, an dem wir anchors.centerIn: parent verwenden, ist zum Zentrieren von Text in unserer messageBox in Bezug auf ihr Text-Rectangle.  

![anchors reference](image.png)

## Nur um Sie auf Trab zu halten

Wenn Sie anchors.fill verwenden, berechnet QML die Breite Ihres Items entsprechend der Breite des Parent-Elements.
Wenn Sie anchors.centerIn verwenden, berechnet QML die Breite Ihres Items entsprechend der Breite seiner Kinder.
Die Breite der Kinder wird hier auf die Breite des Parents gesetzt.  
Entweder muss das ColumnLayout seine Breite explizit entsprechend einem bestimmten Objekt definieren, oder die Kind-Items müssen es tun.  
Die einfachste Lösung ist, die Breite der Spalte explizit entsprechend ihrem Parent zu setzen.

Hinweis: Wenn wir anchors.centerIn anstelle von anchors.fill verwenden, wird die Größe der Spalte 

# Verbinden von Signalen 

Änderungen an der Suchleiste lösen Änderungen am Text in der MessageBox aus. Entweder wird die ID des ausgewählten Elements angezeigt oder eine Fehlermeldung, wenn eine ungültige Plot-ID eingegeben wird.

Der Code hier ist in Standard-JavaScript geschrieben. Die Verbindungen sehen bekannter aus wie PyQt-Verbindungen. item bezieht sich auf das Root-Item, das von der Loader-Component geladen wurde. Es ist ein magisches Schlüsselwort. Diese Signale: plotNotFound und plotLoaded, sind benutzerdefinierte Signale, die im Root-Item von d2_searchbar definiert sind. Wir werden uns das später ansehen.

Sie werden feststellen, dass wir hier nicht Component.onCompleted verwenden, sondern Loader.onLoaded. Das ist unnötig verwirrend. Warum? Component.onCompleted wird ausgelöst, nachdem der Loader konstruiert wurde, aber bevor seine Component geladen wurde. item ist noch null. Das onLoaded-Signal des Loaders wird erst ausgelöst, nachdem die Source-Component geladen wurde, also garantiert es, dass item existiert, und wir können sicher auf seine Signale zugreifen. 

# d2_searchbar.qml
```qml
<...imports ...>
Rectangle {
  id: searchBar
  property var plotsLayer: null  // Als null initialisieren, in Component.onCompleted setzen
  
  // Plot-Liste
  ListModel {
    id: plot_model
  }

  // Signale zur Kommunikation mit Parent-Component
  signal plotNotFound(string plotId)
  signal plotLoaded(string plotId)
  signal layerLoadError(string message)
  
  // Stil
  width: parent.width
  height: 100
  color: PluginTheme.green


  RowLayout {
    anchors.centerIn: parent
    width: parent.width * 0.8
    spacing: 10

    Label {
      id: title
      horizontalAlignment: Text.AlignHCenter
      verticalAlignment: Text.AlignBottom
      text: "Plot: "
      color: PluginTheme.white
      font.pixelSize: 24
      font.bold: true
    }
    ComboBox {
      id: plotInput
      font.pixelSize: 24
      model: plot_model
      textRole: "label"
      valueRole: "label"
      editable: true
      Layout.preferredWidth: 200
      
      onActivated: {
        search_plot()
      }
      onAccepted: {
        search_plot()
      }
      function search_plot(){
        var plot_id = this.currentText;
        if (!plot_id || plot_id.trim() === "") {
          plotNotFound("Empty plot ID");
          return;
        }
        
        if (!plotsLayer) {
          plotNotFound("Layer not available");
          return;
        }
        
        try {
          var expression = "plot_id = '" + plot_id + "'";
          var feature_iterator = LayerUtils.createFeatureIteratorFromExpression(plotsLayer, expression);

          if (!feature_iterator.hasNext()){
            // Fehler - Signal an Parent ausgeben
            plotNotFound(plot_id);
            feature_iterator.close();
            return;
          } else{
            // Erfolg - Signal an Parent ausgeben
            plotLoaded(plot_id);
          }
          feature_iterator.close();
        } catch (error) {
          console.log("Error searching for plot:", error);
          plotNotFound(plot_id + " (Error: " + error + ")");
        }
      }
    } 
  }

  Component.onCompleted: {
    // Layer initialisieren und Modell füllen, nachdem Component vollständig geladen ist
    plotsLayer = qgisProject.mapLayersByName("plots")[0]
    populatePlotModel()
  }

  function populatePlotModel() {
    if (!plotsLayer) {
      console.log("Warning: plots layer not found")
      layerLoadError("Plots layer not found in project")
      return
    }
    
    try {
      plot_model.clear()
      
      // Alle Features aus dem Plots-Layer holen: es gibt keine Funktion, um alle Features zu erhalten, also verwenden wir einen Ausdruck
      // also müssen Sie einen Catch-All-Ausdruck verwenden
      var feature_iterator = LayerUtils.createFeatureIteratorFromExpression(plotsLayer, "plot_id IS NOT NULL");
      var count = 0
      
      while (feature_iterator.hasNext()) {
        var feature = feature_iterator.next()
        var plot_id = feature.attribute("plot_id")
        if (plot_id) {
          plot_model.append({"label": plot_id})
          count++
        }
      }
      feature_iterator.close()
      
      if (count === 0) {
        layerLoadError("No plots found with plot_id attribute")
      }
    } catch (error) {
      console.log("Error populating plot model:", error)
      layerLoadError("Error loading plot data: " + error)
    }
  }
} // /searchBar Rectangle

```
## Lassen Sie uns schneller werden

An diesem Punkt wird die UI etwas komplizierter, und ich werde einige Dinge überspringen, die Sie später nachschlagen können.

- Wie man über die QField-Schnittstelle auf einen Projekt-Layer zugreift
- Wie man Features aus einem Layer über die QField-Schnittstelle abfragt
- Wie man ein durchsuchbares Menü erstellt
````