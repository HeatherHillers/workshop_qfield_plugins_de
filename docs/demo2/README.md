# Demo2 Search Bar

This demonstration builds on Demo 1 by adding a search bar that allows a user to select a feature from a layer.  Since we spent all that time on QML structure in demo 1, this demo and the following demos should go faster and be more understandable.  

# Note: combine demo6 and demo2 into one demo that shows both possibilities for feature selection.

## What we will learn

- How to align widgets using the Column and RowLayout Items.
- How to access a project layer through the qfield interface
- How to query features from a layer through the qfield interface
- How to make a searchable Menu

# Get Ready To Go

1. Keep a hold on your inner peace.  It gets easier from here.
2. Create a new project directory: ${ROOT}/qfield_project_demo_2 
3. Copy the whole demo directory ${ROOT}/qfield_vegetation_plugin/demo2_searchbar ${ROOT}/qfield_project_demo_2
4. Open the project in QGIS.  The project has not changed from demo 1.
5. Upload the project to QFieldCloud using the QField Sync Plugin.
6. Load the project in the Windows QField

## What's it do?
## Note: Make slides for this part
- That camera icon is still there.  Click it.  It has the same open and close functionality as before
- Now you will see a drop down menu.  The items in the menu are the feature ids of the plot points.
- You can select manually or you can text search for a plot.
- If you select a valid plot, the plot id will be displayed in the frame.
- If you type an invalid plot id, an error message will be displayed in the frame.

# Now let's take a closer look at the code and explain the new features.

## [QML for demo 2](qml_demo2.md)

