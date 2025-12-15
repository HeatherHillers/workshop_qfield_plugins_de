# Demonstration 2: Feature Selection 

This demonstration builds on Demo 1 by adding the ability to select a feature from the map and to retrieve information from that feature to use in the plugin.

## What we will learn

- How to access a project layer through the QField interface
- How to query features from a layer through the QField interface
- How to select objects from the map canvas using a point handler
- How to send a signal to close the plugin
- How to align items using Layout Containers

## What does it do?
 
- It opens when the user doubleclicks on a plot on the map canvas.
- It displays the plot id of the selected feature in the text frame.
- A single click on the point will result in the usual Attribute Table behaviour. (On iOS.  This is not possible in the Windows executable.)

![plugin screen](img/demo2_screen.png)

- It also prints messages to the dos console instead of the user's message log when a map feature is selected. 

![alt text](img/demo2_log.png)

# Set it up

1. Create a new project directory: ${ROOT}/qfield_project_demo_2 
2. Copy the whole demo directory ${ROOT}/qfield_vegetation_plugin/demo2_selection ${ROOT}/qfield_project_demo_2
3. Open the project in QGIS.  The project has not changed from demo 1.
4. Upload the project to QFieldCloud using the QField Sync Plugin.
5. Load the project in the Windows QField

## 📚 **[Setting up Selection from the Map](DEMO2_MAP_CLICK.md)**
## 📚 **[<< Demonstration 1](../demo1/DEMO1_INTRO.md)**


