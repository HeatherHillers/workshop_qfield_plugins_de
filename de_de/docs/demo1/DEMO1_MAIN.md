
# demo1_hello.qml

Dies ist die Haupt-QML-Datei, die den Plugin-Code und seine Toolbar-Schaltflächen lädt und entlädt.


```qml
// <!--Imports...-->

import org.qfield 
import Theme  



Item {  
  id: plugin
  parent: iface.mapCanvas() 
  anchors.fill: parent 
  Loader {
    id: pluginLoader
    active: false
    anchors.fill: parent
    source: Qt.resolvedUrl('./components/d1_plugin_component.qml')
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

## QML ist nicht ganz Qt

QML ist im Grunde eine Transformation der XML-formatierten Qt .ui-Layout-Datei in eine Programmiersprache, mit etwas JavaScript, das hinzugefügt wurde, um zusätzliche Funktionalität zu bieten. Eine QML-App denkt nicht von sich selbst als Programm. Sie denkt von sich selbst als Benutzeroberfläche, die Dinge tut. Wenn Sie also als Programmierer anfangen, der daran gewöhnt ist, QGIS-Plugins zu erstellen, die Programme mit Benutzeroberflächen sind, wird das frustrierend sein.  

Was Sie erwarten können:

- Eine verschachtelte, layout-basierte Code-Struktur. 
- Viel Betonung auf Stil.
- Ein Bedürfnis, mehr als üblich auf Timing zu achten.
- Momente der Frustration.

## Verbindungen: Property-Ausdrücke vs Signale und Slots 

QML verwendet JavaScript, um Signal- und Slot-Code zu implementieren, den Sie normalerweise in einem Python-Slot sehen würden. (z.B. der onClicked-Slot im QfToolButton)

Anstatt jedoch Signale und Slots für alles zu verwenden, wie es ein PyQt-Programmierer tun würde, werden wir von QML ermutigt, unsere Interaktionen direkt in die Properties zu programmieren, so:

```qml
Rectangle {id: foo
      height: 200}
Rectangle {id: bar
      height: foo.height + 100}
```

## Klassen in unserer Demo

![alt text](img/demo1_obj.png)


### Item (plugin):

- Item ist unser Root-Objekt. Alles im Plugin muss innerhalb dieses Objekts definiert werden.  
- Item ist eine Basiselternklasse, die QtWidget entspricht. Alle anderen Widgets erben davon.

#### Properties
- **id**: Dies ist der Objektname ("plugin"). Diese Property ist für alle QML-Objekte verfügbar. Seltsamerweise ist sie jedoch optional und wird oft für Items weggelassen, auf die nicht programmatisch verwiesen wird (wie Labels)
- **parent**: Genau wie in PyQt ist dies ein Verweis auf das Eltern-Widget. Es ist nur notwendig, parent im Root-Item zu definieren. Für alles innerhalb des Items ist parent implizit durch die verschachtelte Struktur definiert. Es ist absolut notwendig, den parent auf dem Root-Item als Map-Canvas zu definieren, denn sonst hätte es keinen Parent und keinen Platz in QFields Benutzeroberfläche und würde nicht gerendert werden. 
- **anchors**: Dies ist ein sehr CSS-ähnlicher Mechanismus, der die Geometriegrenzen des Widgets relativ zu anderen Items definiert. Normalerweise wird die Geometrie in Bezug auf den Parent definiert.  
  - **anchors.fill:** parent sagt dem Widget, dass es sich ausdehnen soll, um das gesamte Eltern-Widget zu bedecken. Also sagt uns der Plugin-Item-Anchor, dass dieses Widget das gesamte Map-Canvas bedecken wird - also den gesamten Bildschirm.
  - **Warnung:** Achten Sie besonders auf Ihre Anchors. Wenn Sie sie falsch konfigurieren, könnte Ihre Komponente verschwinden oder auf unerwartete Weise gerendert werden, und es kann schwer zu debuggen sein.

#### Mitglieder des Plugin-Items:

- **Loader**: Der Loader lädt und entlädt das eigentliche Plugin-Objekt.
- **QfToolButton**: Dies ist die Schaltfläche, die geklickt wird, um das Plugin zu öffnen und zu schließen.

#### Component.onCompleted

Dies ist die Signal-Slot-Konstruktion, die das Hinzufügen des pluginButton zur QField-Toolbar auslöst, wenn das Plugin-Item erstellt wurde.

Wenn man sich Component.onCompleted innerhalb von Item ansieht und auch berücksichtigt, dass der Loader eine Component lädt, könnte man dazu verleitet werden zu denken, dass Component die Elternklasse von Item ist, und dieses Signal geerbt wird.  

Das ist **nicht** der Fall. Component und Item haben nichts miteinander zu tun. 

- Component ist ein "attached property mechanism". Was bedeutet das? Ich denke mir das als eine Klasse, die andere Objekte verpackt und Signale bereitstellt, wenn sie konstruiert und destruiert werden. 

- Component hat 2 Signale:
 - onCompleted (das Objekt wurde fertig geladen)
 - onDestruction (wie close auf einem QtWidget)

- In unserem Item wird die Component verwendet, um eine Signal/Slot-Verbindung zu erstellen, die ausgeführt wird, wenn die Item-Konstruktion abgeschlossen ist. Diese Zeile entspricht grob dem PyQt-Connector:

```python
plugin.onCompleted.connect(addItemstoToolbar) 
```
### Loader (pluginLoader):

Ein Loader ist ein Item, das dynamisch andere Items lädt und entlädt. Loader konstruieren und destruieren ihre Components, wenn ihr "active"-Property-Wert umgeschaltet wird, was Runtime-Trigger für Konstruktion und Destruktion ermöglicht. Dies macht es möglich, das Laden des Plugins mit einer Schaltfläche auszulösen.

#### Warum wir hier einen Loader brauchen:  

Unser Plugin wird Dinge tun wie Layer im Projekt lesen. Wir wollen nicht, dass dies passiert, bis das Projekt geladen ist. Tatsächlich wollen wir, dass unser Plugin-Code wartet, bis der Benutzer danach fragt, indem er auf die Tool-Schaltfläche des Plugins klickt oder später ein Objekt auf der Karte auswählt. 

Wenn wir unseren Plugin-Code direkt in unser Root-Item ohne den Loader legen würden, würden wir versuchen, Layer zu lesen, die noch nicht existieren, und viel Verarbeitung durchführen, die möglicherweise nicht benötigt wird.  


#### Properties
- **id und anchors**: weil es ein Item-Widget ist
- **nicht parent**: weil parent implizit aus der Verschachtelung definiert ist. Das plugin-Item ist sein Parent. Nur das Root-Item muss seinen Parent definieren.
- **active**: die boolesche Property, die bestimmt, ob der Code des Loaders konstruiert ist oder nicht. Der QfToolButton wird den Code laden und entladen, indem er Änderungen am Wert dieser Property auslöst.
- **source**: der Pfad zur QML-Datei, die die zu ladenden Items enthält.  
              Die Source wird als Source-Component bezeichnet, obwohl, wenn Sie in die QML-Datei schauen, Sie Items sehen werden, nicht Components.  
              Das liegt daran, dass der Loader intern ein Component-Objekt erstellt, dessen Aufgabe es ist, die Items in der von source referenzierten QML-Datei zu laden.
        
### QfToolButton (pluginButton)

Das ist einfach. QfToolButton ist ein Tool-Button-Widget, das wir mit dem Loader verbunden haben. Wir könnten den QToolButton verwenden, aber wir verwenden hier den QfToolButton von QField, damit er QFields Design beibehält. Für den Rest des Plugins bevorzuge ich tendenziell die Qt-Objekte.  

Ich kann QfToolButton oder jedes andere QField-Widget verwenden, weil ich sie aus org.qfield importiert habe.

Dieses Item ist nicht im Loader, weil es das Item ist, das den Loader steuert.

#### Properties

- **round**: Eine Property von QfToolButton. Macht die Schaltfläche zu einem Kreis.
- **iconSource**: Eine Property von QfToolButton. Aus purer Faulheit habe ich eines der vordefinierten Symbole im QField Theme-Modul verwendet. Wir werfen diese Schaltfläche sowieso in der nächsten Demo weg.  
- **iconColor und bgcolor**: Ich habe definierte Farben aus dem importierten Theme-Modul übernommen, was eine gute Idee ist, wenn Sie möchten, dass Ihre Farben zu QFields Farben passen. 
- **onClicked** : Ist ein Signal-Handler, der JavaScript enthält. Wenn die Schaltfläche geklickt wird, gibt sie eine Nachricht an das Log aus (das das QField-Log ist, das der Benutzer sieht) und schaltet die boolesche active-Property des pluginLoaders um, was entweder eine Konstruktion oder Destruktion der Plugin-Component (Sammlung von Items in der Component-QML-Datei) auslöst

**Denken Sie daran, es ist nicht Component.onClicked. Alles außer onCompletion und onDestruction sind Mitglieder der Item-Klassen und haben nichts mit Component zu tun.**



## 📚 **[>> Schauen wir uns die Plugin-Component an](DEMO1_COMPONENT.md)**
## 📚 **[<< Plugin-Struktur](DEMO1_STRUCTURE.md)**
