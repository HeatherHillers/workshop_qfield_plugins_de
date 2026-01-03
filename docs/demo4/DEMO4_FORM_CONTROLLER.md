# Demonstration 4: Form Controller

The Form Controller does all the work of managing the form, but we are going to concentrate here on the 2 functions for saving the plot and loading the plot, because these lead us to the interface with QField.

## dataController handles interactions with the project layer

The header page is writing its form data to the non-geo private layer plot_header in the qgis project.  d4_data_controller is imported as DataController and assigned to the dataController property.  This is the class we will explore next to finally reveal the mechanisms for inserting, updating and deleting features.

## the widgets property holds references to the ui elements

All widgets in the header form use the register functions in the controller to add themselves to the widgets array property, where they can be accessed by the save and load functions.  This happens when a widget is created by the header page.

## Load and Save

- the load function retrieves the plot_header row from the data controller and sends it to the functions that populate the form.

- the save function retrieves the widget values as a dictionary and passes it to the data controller to save to the plot_header layer.

Ba da Bing.

```qml
<...>

// Form Controller: Bridges between data layer and UI widgets
// Handles widget value synchronization without QField API details
QtObject {
    id: formController
    
    // Data layer - QField/QGIS API interactions
    property var dataController: DataController {
        id: dataControllerInstance
    }
    
    <...>
    
    // Widget registry - maps field IDs to widget accessors
    property var widgets: ({})
    
    <...>

    function save() {
        <...>  

        var fieldValues = collectFormValues()
        var success = dataController.saveFieldValues(fieldValues)

        <...>
    }
    
    /**
     * Load feature for a plot and populate form
     */
    function loadPlot(plotId) {
        <...>
        
        var feature = dataController.loadHeaderForPlot(plotId)
        if (feature) {
            populateFormFromFeature(feature)
        }
    }
    
    // Collect all form values and return a field-value map
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

## 📚 **[The Data Controller and the QField Interface](DEMO4_DATA_CONTROLLER.md)**
## 📚 **[<< Demonstration 4 Header Page](DEMO4_HEADER_PAGE.md)**