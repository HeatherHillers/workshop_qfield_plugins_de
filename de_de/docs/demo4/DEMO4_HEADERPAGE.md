````markdown
# Demonstration 4: Die Header-Seite

Die d4_headerpage.qml ist ziemlich einfach. Sie erstellt das grundlegende Layout und die Struktur des Formulars im Header-Formular, delegiert aber die meiste Arbeit an den formController.

## Struktur und Layout: Repeater lädt Inputs aus Konfiguration

Die headerPage ist ein einfaches Rectangle, das enthält:

 - Titeltext
 - Speichern-Schaltfläche
 - Repeater für Eingaben

 ## FormDataModel hält die Formularkonfiguration

 Das Formular ist in mehrere Abschnitte unterteilt, die jeweils eine Anzahl von Eingabe-Widgets vom Typ Dezimal oder Text enthalten. Diese Abschnitte und Eingaben sind im FormDataModel konfiguriert und werden aus diesem abgerufen, das in d4_data_model.qml definiert ist. Wir werden dies für Ihre Durchsicht nach dem Workshop belassen. 

 ## Warum kann QML sowieso keine For-Schleife machen?

 Der Repeater ist ein For-Schleifen-Konstruktor, der über die groupBoxes-Liste im Datenmodell iteriert und für jede groupBox sein delegate-Objekt erstellt, das eine FormSection ist, die in d4_form_section.qml definiert ist. Diese erhält als Property den Form Controller. Wir werden auch d4_form_section.qml für Ihre spätere Durchsicht belassen. Diese Komponenten werden wiederum über die Eingabe-Widgets iterieren, die in ihrer groupBox in der Konfiguration enthalten sind. Das Wichtige ist, dass sich die Eingabe-Objekte beim Erstellen beim Form Controller registrieren.

 ## Fokus auf den FormController

 Der Form Controller erledigt hier die ganze Arbeit. Wir werden uns ihn als Nächstes genauer ansehen.
 
 - Registrierung von Widgets: Er wird vom Repeater an jedes Eingabe-Widget gesendet, das sich beim Form Controller registriert.  
 - Laden von Daten: Der onPlotChanged-Slot ruft formController.loadPlot mit der ausgewählten plotId auf. Der FormController holt die Daten aus dem plot_header-Layer und füllt die registrierten Repeater-Widgets mit den Daten.
 - Speichern von Daten: Die saveButton ruft die save-Funktion des formControllers onClicked auf. Dies ruft den aktuellen Datenzustand aus den registrierten Widgets ab und speichert ihn im plot_header-Layer
 - hasUnsavedChanges: Wenn Daten in einem der registrierten Widgets eingegeben werden, wird diese State-Property auf true geändert, bis die save-Funktion aufgerufen wird.

```qml
<...>
Rectangle {
    id: headerPage
    property string plotId: parent.plotId
    <...>
    
    // Formularkoordination zwischen Daten und UI
    FormController {
        id: controller
    }
    
    // Verschleierung: Property-Alias, um Controller Kindern zugänglich zu machen
    property var formController: controller
    
    // Plot-Daten laden, wenn sich plotId ändert (über Bindung vom Parent gesetzt)
    onPlotIdChanged: {
        if (plotId ) {
            console.log("HeaderPage: Loading plot", plotId)
            formController.loadPlot(plotId)
        }
    }
    
    // ===================================================================
    // LAYOUT
    // ===================================================================
    
    Column {
        <...>
        // Titel
        Text {
                text: "Vegetation Monitoring - Plot: " + (plotId || "None")
                <...>
            }
            
        // Speichern-Schaltfläche
        Rectangle {
            id: saveButton
            <...>
            color: formController.hasUnsavedChanges ? "#2E7D32" : "#1B5E20"
            <...>
            
            Text {
                text: formController.hasUnsavedChanges ? "⚠ Daten speichern" : "✓ Gespeichert"
                <...>
            }
            
            MouseArea {
                anchors.fill: parent
                onClicked: {
                    formController.save()
                }
            }
        }
        
        // Dynamische Formularabschnitte aus Datenmodell
        Repeater {
            model: FormDataModel.groupBoxes
            
            delegate: FormSection {
                width: parent.width
                sectionData: modelData
                
                // Controller-Referenz für Registry übergeben
                controller: headerPage.formController
            }
        }
    }  // Column
} // Rectangle

```

## 📚 **[Der Form Controller](DEMO4_FORM_CONTROLLER.md)**
## 📚 **[<< Demonstration 4 Plugin Component](DEMO4_PLUGIN_COMPONENT.md)**
````