# Demonstration 4: The Header Page

The d4_headerpage.qml is quite simple.  It creates the basic layout and structure of the form in the header form, but delegates most of the work to the formController.

## Structure and Layout: Repeater loads Inputs from Configuration

The headerPage is a vanilla Rectangle containing:

 - title text
 - save button
 - Repeater for inputs

 ## FormDataModel holds the form configuration

 The Form is divided into multiple sections, each containing a number of inputs widgets of decimal or text type.  These sections and inputs are configured in and retrieved from the FormDataModel defined in d4_data_model.qml.  We will leave this for your perusal after the workshop. 

 ## Why can't QML make a for loop, anyway?

 The Repeater is a for loop constructor that iterates over the groupBoxes List in the data model, and for each groupBox creates its delegate object, which is a FormSection, which is defined in d4_form_section.qml.  This recieves as a Property the Form Controller.  We will also leave d4_form_section.qml to your later perusal.  These components will in turn iterate over the input widgets contained in their groupBox in the configuration.  The important bit is that when the input objects are created, they register themselves with the form controller.

 ## Focus on the FormController

 The Form Controller does all the work here.  We will look at it more closely next.
 
 - Registering Widgets: It gets sent by the repeater down to each input widget, which registers itself with the Form Controller.  
 - Loading Data:  the onPlotChanged slot calls formController.loadPlot with the selected plotId.  The FormController will get the data from the plot_header layer and populate the registered Repeater widgets with the data.
 - Saving Data:  the saveButton calls the formController's save function onClicked.  This will retrieve the current data state from the registered widgets and save it to the plot_header layer
 - hasUnsavedChanges: if data is entered in any of the registered widgets, this state property will be changed to true until the save function is called.

```qml
<...>
Rectangle {
    id: headerPage
    property string plotId: parent.plotId
    <...>
    
    // Form coordination between data and UI
    FormController {
        id: controller
    }
    
    // Obfuscation: Property alias to expose controller to children
    property var formController: controller
    
    // Load plot data when plotId changes (set via binding from parent)
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
        // Title
        Text {
                text: "Vegetation Monitoring - Plot: " + (plotId || "None")
                <...>
            }
            
        // Save button
        Rectangle {
            id: saveButton
            <...>
            color: formController.hasUnsavedChanges ? "#2E7D32" : "#1B5E20"
            <...>
            
            Text {
                text: formController.hasUnsavedChanges ? "⚠ Save Data" : "✓ Saved"
                <...>
            }
            
            MouseArea {
                anchors.fill: parent
                onClicked: {
                    formController.save()
                }
            }
        }
        
        // Dynamic form sections from data model
        Repeater {
            model: FormDataModel.groupBoxes
            
            delegate: FormSection {
                width: parent.width
                sectionData: modelData
                
                // Pass controller reference for registry
                controller: headerPage.formController
            }
        }
    }  // Column
} // Rectangle

```

## 📚 **[The Form Controller](DEMO4_FORM_CONTROLLER.md)**
## 📚 **[<< Demonstration 4 Plugin Component](DEMO4_PLUGIN_COMPONENT.md)**