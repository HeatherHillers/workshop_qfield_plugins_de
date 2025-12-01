# Get Ready To Go

1. Attain some inner peace and hold on to it.
2. Choose a development directory, which will be refered to as ${ROOT}
2. clone or download this repository http://github.com/HeatherHillers/workshop_qfield_plugins_de under ${ROOT}
3. clone or download the qfield vegetation monitoring plugin repository from http://github.com/HeatherHillers/qfield_vegetation_monitoring under ${ROOT}
4. create the project directory: ${ROOT}/qfield_project_demo_1 
5. Open up QGIS and QField and log in to your account at https://qfield.cloud


# Setup and sync the vegetation project to QField
1. Copy the project files and project data to ${ROOT}/qfield_project_demo_1
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qgs
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qgs.png
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello_attachments.zip
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/vegetation.gpkg
2. Open the project in QGIS.
      - you should see just a single layer (plots) with a few points in it.  This is your selection layer.
      - Each of these points has a plot_id.  This is the field queried by the plugin.
3. Special Note: Make sure the Editing Mode Configuration wont cause problems:
    - Open the Project Properties
    - Open the QField Tab.
    - In the QFieldCloud Packaging tab, on the top right there is a config icon.  
    - Click on it and select "prefer Offline layers".
    - Select Offline editing as the Packaging Action for plots and any other layers.
    - If you dont do this, you may get a Network error when you sync changes to the project in QField.  If you get such an error, selecting prefer Offline Layers and resyncing should solve the problem.
4. Upload the project to QFieldCloud using the QField Sync Plugin.
5. Load the project in the Windows QField
      - this project is formatted for an iPad screen, so you are better off using your desktop QField to show the project. Unless you have an iPad.

# Lets look the Plugin Structure
1. Open ${ROOT}\qfield_vegetation_monitoring in Visual Studio Code or your prefered IDE.

## Critical Points
    - main qml must have same name as project and is saved in the project directory with the project file
    - all other qml is in the components subdirectory
    - this subdirectory will be added as an attachment directory in the project

# Load the Plugin in QField.
Lets see what it does before we look at the code.

2. Copy the plugin files to ${ROOT}/qfield_project_demo_1
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qml
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components/d1_plugin_component.qml
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components/PluginTheme.qml
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components/qmldir


3. Reload the Plugin in QGIS
4. Resync the Plugin in QField Sync
5. Synchronise your Plugin in QField
   - you will probably get a network error the first time.
   - if so, click on synchronise again.
   - then return to the Project List and reopen the project.
   - You should now see a dialog about enabling the plugin.  If not, the sync did not work.  If yes, click yes.

## What's it do?
## Note: Make slides for this part
- You should see a camara icon on the right, that wasnt there before.  Click it.
- You will now see that the screen color changes and there is a title text.
- Also, on the right the message icon has turned on.  That isn't an error.  The Plugin is printing some debug messages there for you to read.
- click the camera again to turn the plugin off.  A couple more debug messages print. 
- keep clicking.  It is oddly satisfying and good for your mental health.

# Now let's take a closer look at the code and explain some QML.

## [QML for demo1](qml_demo1.md)

