````markdown
# Demonstration 4 Data Controller

Hier lernen wir alle wichtigen Teile der QField-Schnittstelle:

- Ein Feature abrufen (nochmals)
- Ein Feature-Attribut abrufen (nochmals)
- Ein Feature einfügen
- Ein Feature aktualisieren
- Ein Feature löschen (zusätzlicher Einschub)


## Das Wichtigste:

Versuchen Sie nicht, die QField-API zu umgehen, indem Sie direkt auf die Datenanbieter zugreifen. Denken Sie daran, dass die Synchronisation von Daten auf QGIS Offline Editing beruht. Wenn Sie versuchen, Ihre Daten direkt zu bearbeiten, werden die Änderungen nicht erkannt, wenn Sie das Projekt synchronisieren, und werden überschrieben. Verlassen Sie sich auf die QField-API, um sicherzustellen, dass die Synchronisierung gut funktioniert und alle QField-Signale korrekt verarbeitet werden.

## Die QField-API

FeatureUtils und LayerUtils, zusammen mit iface, sind die wichtigsten Komponenten der QField-API. Diese werden aus org.qfield importiert.

## Ein Feature abrufen

Wir haben dies bereits im Point-Handler behandelt, aber es ist hier enthalten, um eine vollständige Referenz bereitzustellen.

1. Einen QgsFeature-Iterator von LayerUtils mit einem QGIS-Stil-Ausdrucksstring abrufen.
2. Denken Sie daran, Ihren Iterator zu schließen.

```qml
QtObject {
    id: dataController
    
    property var layer: qgisProject.mapLayersByName("plot_header")[0]
    property var currentFeature: null
    
    function loadHeaderForPlot(plotId) {
        <...>
        // Abfrage für vorhandenes Feature
        var expression = "plot_id = '" + plotId + "'"
        var iterator = LayerUtils.createFeatureIteratorFromExpression(layer, expression)
        
        if (iterator.hasNext()) {
            // Feature existiert - laden Sie es
            currentFeature = iterator.next()
            iterator.close()  // KRITISCH: Immer Iteratoren schließen
        } else {
            iterator.close()  // KRITISCH: Immer Iteratoren schließen
            
            // Feature existiert nicht - erstellen Sie es
            currentFeature = createNewFeature(plotId)
        }
        
        featureLoaded(currentFeature)
        return currentFeature
    }
```

## Ein Feature einfügen

Hier wird eine Mischung aus QGIS- und QField-Funktionen verwendet. Wenn es ein QField-Utility gibt, um etwas zu tun, werden wir das gegenüber der direkten Verwendung von QGIS-Funktionen bevorzugen.

1. Signalisieren Sie, wie in QGIS, den Beginn der Bearbeitung.

- layer.startEditing() wird vor allen Einfügungen, Aktualisierungen oder Löschungen aufgerufen.


2. Erstellen Sie ein neues leeres Feature.

- FeatureUtils.createFeature(QgsVectorLayer) erstellt ein leeres QgsFeature mit der entsprechenden Attributstruktur für den Layer. **Dies bearbeitet den Layer nicht. Es erstellt nur ein separates Feature-Objekt, das noch nicht zum Layer hinzugefügt wurde**

3. Ein Feature-Attribut setzen

- Verwenden Sie die QgsFeature.setAttribute-Funktion, um Feature-Werte zu setzen.  

Hier setzen wir die automatischen Werte, die aus dem Formular ausgeblendet sind: eine UID und einen Zeitstempel für das neue Feature sowie die Plot-ID. **Dies bearbeitet den Layer nicht. Nur das Feature.**

4. Ein Feature einfügen

- Verwenden Sie QFields LayerUtils.addFeature, um ein Feature zum Layer hinzuzufügen.

5. Wie in QGIS, committen Sie Ihre Änderungen am Layer.

- layer.commitChanges() wird aufgerufen, um alle Einfügungen, Aktualisierungen oder Löschungen seit startEditing() zu speichern.

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
        
        // Erneut abrufen, um gültige Feature-ID nach Commit zu erhalten
        var iterator = LayerUtils.createFeatureIteratorFromExpression(
            layer, 
            "plot_id = '" + plotId + "'"
        )
        var committedFeature = iterator.next()
        iterator.close()  // KRITISCH: Immer Iteratoren schließen
        
        console.log("Created feature with ID:", committedFeature.id)
        return committedFeature
    }
```

## Ein Feature-Attribut abrufen

Wir haben dies in unserer Diskussion über den Point-Handler behandelt, aber wir werden es hier der Vollständigkeit halber einbeziehen.

Verwenden Sie QgsFeature.attribute(fieldName), um einen Wert abzurufen. Verwenden Sie niemals einen hart kodierten Index-Wert, um einen Wert mit QgsFeature.attribute(index) abzurufen. Die Index-Werte sind nicht konsistent zwischen QGIS-Projekt und QField-Projekt. Heiterkeit wird folgen.

    ```qml
    function getFieldValue(fieldName) {
        <...>
        return currentFeature.attribute(fieldName)
    }
    ```

## Ein vorhandenes Feature mit neuen Attributwerten aktualisieren

Das plot_header-Feature wird abgerufen oder erstellt und committed, wenn der Header-Tab geöffnet wird, also wenn der Benutzer auf Speichern klickt, aktualisiert er immer ein vorhandenes Feature.

1. Holen Sie sich Ihr QgsFeature 
- Unser Header-Feature ist bereits in der currentFeature-Property gespeichert.

2. Signalisieren Sie wie in QGIS die Bearbeitungsaktion

- QgsVectorLayer.startEditing() wird aufgerufen

3. Holen Sie sich die Feature-ID

- Die QgsFeature.id-Property gibt Ihnen die Feature-ID.

4. Holen Sie sich die Feldliste

- Die QgsFeature.fields-Property gibt Ihnen das QgsFields-Objekt.

5. Holen Sie sich den QField-Index des Feldes, das Sie ändern möchten

QgsFields.indexOf(fieldName) gibt Ihnen den echten QField- (nicht QGIS-) Attribut-Feldindex.

6. Ändern Sie den Attributwert direkt im Layer

- QgsVectorLayer.changeAttributeValue(fid, attribute_index, new_value) aktualisiert den Layer für die gegebene Feature-ID und den Attribut-Index.

7. Committen Sie Ihre Änderungen.

- Verwenden Sie QgsVectorLayer.commitChanges(), um die Änderungen am Layer zu speichern


```qml

    function saveFieldValues(fieldValueMap) {
        <...>
        layer.startEditing()
        
        var fid = currentFeature.id
        var fields = currentFeature.fields
        
        // Zeitstempel aktualisieren
        var logDateIndex = fields.indexOf("log_date")
        layer.changeAttributeValue(fid, logDateIndex, new Date().toISOString())
        
        // Alle Feldwerte aus der Map aktualisieren
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

## Ein Feature löschen

In Demonstration 4 löschen wir tatsächlich keine Features. Wie werden plot_headers gelöscht, wenn ein Feature gelöscht wird? Werden sie nicht. Mein praktischer Anwendungsfall ist ein kleines Team von Biologen, und am Ende der Saison werde ich einfach alle Waisen in meiner Datenbank bereinigen.  

In Demonstration 5 können wir Arteneinträge in den Strata-Seiten löschen. Aber ich habe das Gefühl, dass wir jetzt in diesem Workshop möglicherweise an Energie verlieren. Ich weiß, dass ich das beim Schreiben tue. Also anstatt Sie durch die Strata-Seiten-Benutzeroberfläche in Demo5 zu ziehen, nur um Ihnen eine einfache Löschfunktion zu zeigen, lassen Sie uns sie einfach hier anhängen.

1. Signalisieren Sie den Start der Bearbeitung

- QgsVectorLayer.startEditing wird vor der Löschung aufgerufen

2. Holen Sie sich die Feature-ID

- QgsFeature.id gibt Ihnen die Feature-ID. Wir haben sie bereits als Funktionsparameter.

3. Löschen Sie das Feature

- QgsVectorLayer.deleteFeature(id) wird aufgerufen, um das Feature zu löschen.

4. Zurückrollen einer fehlerhaften Bearbeitung

- QgsVectorLayer.rollBack() wird aufgerufen, um Bearbeitungen am Layer abzubrechen.

5. Bearbeitungen speichern

- QgsVectorLayer.commitChanges() wird aufgerufen, um Bearbeitungen am Layer zu speichern.

```qml
// Löschfunktion von demo5_strata_pages\components\d5_strata_data_controller.qml

    /**
     * Einen Arteneintrag nach Feature-ID löschen
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

## 📚 **[Das Ende](..\..\WORKSHOP_END.md)**
## 📚 **[<< Demonstration 4 Form Controller](DEMO4_FORM_CONTROLLER.md)**
````