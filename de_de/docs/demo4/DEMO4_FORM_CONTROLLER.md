````markdown
# Demonstration 4: Form Controller

Der Form Controller erledigt die ganze Arbeit der Verwaltung des Formulars, aber wir werden uns hier auf die 2 Funktionen zum Speichern und Laden des Plots konzentrieren, weil diese uns zur Schnittstelle mit QField führen.

## dataController behandelt Interaktionen mit dem Projekt-Layer

Die Header-Seite schreibt ihre Formulardaten in den Nicht-Geo-privat-Layer plot_header im QGIS-Projekt. d4_data_controller wird als DataController importiert und der dataController-Property zugewiesen. Dies ist die Klasse, die wir als Nächstes untersuchen werden, um schließlich die Mechanismen zum Einfügen, Aktualisieren und Löschen von Features zu enthüllen.

## Die widgets-Property enthält Verweise auf die UI-Elemente

Alle Widgets im Header-Formular verwenden die Registrierungsfunktionen im Controller, um sich zum widgets-Array-Property hinzuzufügen, wo sie von den Speicher- und Ladefunktionen aufgerufen werden können. Dies geschieht, wenn ein Widget von der Header-Seite erstellt wird.

## Load und Save

- Die Load-Funktion ruft die plot_header-Zeile vom Data Controller ab und sendet sie an die Funktionen, die das Formular füllen.

- Die Save-Funktion ruft die Widget-Werte als Dictionary ab und übergibt sie an den Data Controller zum Speichern im plot_header-Layer.

Ba da Bing.

```qml
<...>

// Form Controller: Brücke zwischen Datenlayer und UI-Widgets
// Handhabt Widget-Wert-Synchronisation ohne QField-API-Details
QtObject {
    id: formController
    
    // Datenlayer - QField/QGIS API-Interaktionen
    property var dataController: DataController {
        id: dataControllerInstance
    }
    
    <...>
    
    // Widget-Registry - bildet Feld-IDs auf Widget-Accessors ab
    property var widgets: ({})
    
    <...>

    function save() {
        <...>  

        var fieldValues = collectFormValues()
        var success = dataController.saveFieldValues(fieldValues)

        <...>
    }
    
    /**
     * Feature für einen Plot laden und Formular füllen
     */
    function loadPlot(plotId) {
        <...>
        
        var feature = dataController.loadHeaderForPlot(plotId)
        if (feature) {
            populateFormFromFeature(feature)
        }
    }
    
    // Alle Formularwerte sammeln und eine Feld-Wert-Map zurückgeben
    function collectFormValues() {...}
    
    function populateFormFromFeature(feature) {...}

    function registerWidget(fieldId, widgetAccessor) { ... }
    
    function allWidgetsRegistered() {...}
    
    function getWidgetValue(fieldId) {...}
    
    function setWidgetValue(fieldId, value) {...}
    
    function clearAllWidgets() {...}
    
    function markChanged() {...}
}



```

## 📚 **[Der Data Controller und die QField-Schnittstelle](DEMO4_DATA_CONTROLLER.md)**
## 📚 **[<< Demonstration 4 Header-Seite](DEMO4_HEADER_PAGE.md)**
````