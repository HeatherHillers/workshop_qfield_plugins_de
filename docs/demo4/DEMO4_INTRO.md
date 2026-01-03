# Demonstration 4: Feature Editing 

This demonstration adds to the tab widget that was introduced in qfield_vegetation_monitoring/demo3_tabwidget.  Every year, the biologists take some basic information about the plot, like the average vegetation height, and record it in the table plot_header.  When we open our tab widget, the header page is the first tab.  This demonstration sets up the header page for the tab widget.  The ui in this demonstration is too complex to cover in this workshop.  We will leave it to you to review later.  We are going to concentrate here on the good bits- the qfield interface for editing features.

## What we will learn

- How to add a row to a table over the QField interface
- How to update a row over the QField interface
- A LOT of UI stuff that we are going to just skim over.

## What does it do?

- When the user clicks on the canvas, the plot id is sent from the plugin component to the tab widget to the tab widget's header page in d5_headerpage.qml
- Note: the form is long.  The form has an invisible scroll area accessible by the scroll wheel on Windows or swipe on ios.
- When it loads, it checks to see if a row already exists in the plot_header table.  

    - If no plot_header row exists, it adds one immediately.  This row has default values for all inputs.
        - 0 for all numerics
        - empty string for text fields
        - a uuid for the primary key
        - the current datestamp
        - the plot id as a foreign key to plot.
    - If a plot_header row already exists, it loads the values to the form
- This form does not have autosave, by user request.
- If a value is updated, the text of the save button will change to indicate unsaved changes.
- When the user clicks the save button, changes to all values are saved and committed.
 
![plugin screen](img/demo4_screen.png)



# Set it up

1. Create a new project directory: ${ROOT}/qfield_project_demo_4 
2. Copy the whole demo directory \${ROOT}/qfield_vegetation_plugin/demo4_header_form ${ROOT}/qfield_project_demo_4
3. Open the project in QGIS if you wish to see the structure of the plot_header table.  This is included in the project as a private (hidden) layer without geometry.
4. Run Qfield from the command line to open the project directly as a local project.
```dos
"C:\Program Files\QField\usr\bin\qfield.exe C:\temp\qfield\demo4_header_form\demo4_header_form.qgs
```


## 📚 **[Getting to the Header Page](DEMO4_PLUGIN_COMPONENT.md)**
## 📚 **[<< Demonstration 1](../demo2/DEMO2_INTRO.md)**


