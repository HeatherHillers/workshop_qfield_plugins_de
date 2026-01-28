# Demonstration 4 Data Controller

Here we will learn all the important bits about the qfield interface:

- getting a feature (again)
- getting a feature attribute (again)
- inserting a feature
- updating a feature
- delete a feature (extra add in)


## The most important thing:

Do not try to bypass the QField API by directly accessing the data providers.  Remember that the synchronisation of data relies on QGIS Offline Editing.  If you try to edit your data directly, the changes will not be recognised when you synchronise the project and will be overwritten.  Rely on the QField API to assure that synchronising works well and any QField signals are correctly processed.

## The QField API

FeatureUtils and LayerUtils, together with iface, are the most importants components of the QField API.  These are imported from org.qfield.

## Get a feature

We have already covered this in the point handler, but it is included here to provide a complete reference.

1. retrieve a QgsFeature iterator from LayerUtils using a QGIS-style expression string.
2. remember to close your iterator.

```qml
QtObject {
    id: dataController
    
    property var layer: qgisProject.mapLayersByName("plot_header")[0]
    property var currentFeature: null
    
    function loadHeaderForPlot(plotId) {
        <...>
        // Query for existing feature
        var expression = "plot_id = '" + plotId + "'"
        var iterator = LayerUtils.createFeatureIteratorFromExpression(layer, expression)
        
        if (iterator.hasNext()) {
            // Feature exists - load it
            currentFeature = iterator.next()
            iterator.close()  // CRITICAL: Always close iterators
        } else {
            iterator.close()  // CRITICAL: Always close iterators
            
            // Feature doesn't exist - create it
            currentFeature = createNewFeature(plotId)
        }
        
        featureLoaded(currentFeature)
        return currentFeature
    }
```

## Insert a Feature

Here a mix of QGIS and QField Functions are used.  If there is a QField Utility for doing a thing, we will prefer that over directly using QGIS functions.

1. As in QGIS, Signal the start of editing.

- layer.startEditing() is called before any inserts, updates, or deletes.


2. Create a new empty feature.

- FeatureUtils.createFeature(QgsVectorLayer) creates an empty QgsFeature with the appropriate attribute structure for the layer.  **This does not edit the layer. It only creates a separate feature object that is not yet added to the layer**

3. Set a feature attribute

- Use the QgsFeature.setAttribute function to set feature values.  

Here we set the automatic values hidden from the form: a uid and timestamp for the new feature, as well as the plot id.  **This does not edit the layer. Only the feature.**

4. Insert a feature

- Use QField's LayerUtils.addFeature to add a feature to the layer.

5. As in QGIS, commit your changes to the layer.

- layer.commitChanges() is called to save any inserts, updates or deletes since startEditing() was called.

```qml
    function createNewFeature(plotId) {
        console.log("Creating new feature for plot:", plotId)
        
        layer.startEditing()
        
        var newFeature = FeatureUtils.createFeature(layer)
        newFeature.setAttribute("f_uid", StringUtils.createUuid().replace(/[\{\}]/g, ""))
        newFeature.setAttribute("plot_id", plotId)
        newFeature.setAttribute("log_date", new Date().toISOString())
        
        LayerUtils.addFeature(layer, newFeature)
        layer.commitChanges()
        
        // Re-fetch to get valid feature ID after commit
        var iterator = LayerUtils.createFeatureIteratorFromExpression(
            layer, 
            "plot_id = '" + plotId + "'"
        )
        var committedFeature = iterator.next()
        iterator.close()  // CRITICAL: Always close iterators
        
        console.log("Created feature with ID:", committedFeature.id)
        return committedFeature
    }
```

## Get a feature attribute

We covered this in our discussion of the point handler, but we will include it here for the sake of completion.

Use QgsFeature.attribute(fieldName) to retrieve a value.  Never use a hard coded index value to retrieve a value using QgsFeature.attribute(index).  The index values are not consistent between QGIS project and the QField project.  Hilarity will ensue.

    ```qml
    function getFieldValue(fieldName) {
        <...>
        return currentFeature.attribute(fieldName)
    }
    ```

## Update an existing feature with new attribute values

The plot_header feature is retrieved or created and commited when the header tab is opened, so when the user hits save, they are always updating an existing feature.

1. get your QgsFeature 
- Our header feature is already saved to the currentFeature property.

2. As in QGIS, signal the editing action

- QgsVectorLayer.startEditing() is called

3. Get the Feature id

- the QgsFeature.id property will give you the feature id.

4. Get the fields list

- the QgsFeature.fields property will give you the QgsFields object.

5. Get the QField index of the field you want to change

QgsFields.indexOf(fieldName) will give you the real QField (not QGIS) attribute field index.

6. Change the attribute value directly in the layer

- QgsVectorLayer.changeAttributeValue(fid, attribute_index, new_value) will update the layer for the given feature id and attribute index.

7. Commit your changes.

- use QgsVectorLayer.commitChanges() to save the changes to the layer


```qml

    function saveFieldValues(fieldValueMap) {
        <...>
        layer.startEditing()
        
        var fid = currentFeature.id
        var fields = currentFeature.fields
        
        // Update timestamp
        var logDateIndex = fields.indexOf("log_date")
        layer.changeAttributeValue(fid, logDateIndex, new Date().toISOString())
        
        // Update all field values from the map
        for (var fieldName in fieldValueMap) {
            var value = fieldValueMap[fieldName]
            var fieldIndex = fields.indexOf(fieldName)
            
            if (fieldIndex >= 0) {
                layer.changeAttributeValue(fid, fieldIndex, value)
                console.log("Saved field:", fieldName, "=", value)
            } else {
                console.log("Warning: Field not found:", fieldName)
            }
        }
        
        layer.commitChanges()
        featureSaved()
        
        console.log("Successfully saved feature")
        return true
    }
``` 

## Delete a feature

In Demonstration 4, we don't actually delete features.  How do plot_headers get deleted if a feature gets deleted? They don't.  My practical use case is one small team of biologists, and at the end of the season I'll just clean up any orphans in my database.  

In Demonstration 5, we can delete species entries in the strata pages.  But I have a feeling that by now in this workshop we might be running out of time and steam.  I know I am in writing it.  So instead of dragging you through the strata page user interface in demo5 just to show you a simple delete function, lets just tack it on here.

1. Signal start Editing

- QgsVectorLayer.startEditing is called before the deletion

2. Get the Feature id

- QgsFeature.id will give you the feature id.  We have it already as a function parameter.

3. delete the feature

- QgsVectorLayer.deleteFeature(id) is called to delete the feature.

4. rolling back a bad edit

- QgsVectorLayer.rollBack() is called to cancel edits to the layer.

5. save edits

- QgsVectorLayer.commitChanges() is called to save edits to the layer.

```qml
// delete Function von demo5_strata_pages\components\d5_strata_data_controller.qml

    /**
     * Delete a species entry by feature ID
     */
    function deleteEntry(featureId) {
        if (!speciesLayer) {
            error("Species layer not available")
            return false
        }
        
        speciesLayer.startEditing()
        
        var success = speciesLayer.deleteFeature(featureId)
        if (!success) {
            error("Failed to delete feature")
            speciesLayer.rollBack()
            return false
        }
        
        var commitSuccess = speciesLayer.commitChanges()
        if (!commitSuccess) {
            error("Failed to commit deletion")
            speciesLayer.rollBack()
            return false
        }
        
        console.log("Deleted entry with ID:", featureId)
        entryDeleted(featureId)
        return true
    }
    ```

## 📚 **[The End](..\..\WORKSHOP_END.md)**
## 📚 **[<< Demonstration 4 Form Controller](DEMO4_FORM_CONTROLLER.md)**