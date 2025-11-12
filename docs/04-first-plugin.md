# Building Your First Plugin

This guide walks you through creating a simple QField plugin from scratch.

## Step 1: Create Plugin Directory Structure

Create a new directory for your plugin:

```bash
mkdir -p my_first_qfield_plugin/icons
cd my_first_qfield_plugin
```

## Step 2: Create metadata.txt

Create a `metadata.txt` file with plugin information:

```ini
[general]
name=My First QField Plugin
qgisMinimumVersion=3.0
description=A simple plugin for QField data collection
version=1.0.0
author=Your Name
email=your.email@example.com

about=This is a tutorial plugin for learning QField plugin development.
      It demonstrates basic plugin structure and functionality.

tracker=https://github.com/yourusername/my_first_qfield_plugin/issues
repository=https://github.com/yourusername/my_first_qfield_plugin
tags=qfield,tutorial,beginner
category=Plugins
icon=icons/icon.png
experimental=False
deprecated=False
```

## Step 3: Create __init__.py

Create the initialization file that QGIS will load:

```python
# __init__.py

def classFactory(iface):
    """Load MyFirstQFieldPlugin class from file my_first_plugin.py
    
    :param iface: A QGIS interface instance.
    :type iface: QgsInterface
    """
    from .my_first_plugin import MyFirstQFieldPlugin
    return MyFirstQFieldPlugin(iface)
```

## Step 4: Create Main Plugin File

Create `my_first_plugin.py` with the main plugin class:

```python
# my_first_plugin.py

import os
from qgis.PyQt.QtWidgets import QAction, QMessageBox
from qgis.PyQt.QtGui import QIcon
from qgis.core import QgsProject, QgsEditFormConfig

class MyFirstQFieldPlugin:
    """QGIS Plugin Implementation."""

    def __init__(self, iface):
        """Constructor.
        
        :param iface: An interface instance that will be passed to this class
            which provides the hook by which you can manipulate the QGIS
            application at run time.
        :type iface: QgsInterface
        """
        self.iface = iface
        self.plugin_dir = os.path.dirname(__file__)
        
        # Initialize plugin variables
        self.actions = []
        self.menu = 'My First QField Plugin'

    def add_action(
        self,
        icon_path,
        text,
        callback,
        enabled_flag=True,
        add_to_menu=True,
        add_to_toolbar=True,
        status_tip=None,
        whats_this=None,
        parent=None
    ):
        """Add a toolbar icon to the toolbar.
        
        :param icon_path: Path to the icon for this action.
        :type icon_path: str
        
        :param text: Text that should be shown in menu items for this action.
        :type text: str
        
        :param callback: Function to be called when the action is triggered.
        :type callback: function
        
        :param enabled_flag: A flag indicating if the action should be enabled
            by default. Defaults to True.
        :type enabled_flag: bool
        
        :param add_to_menu: Flag indicating whether the action should also
            be added to the menu. Defaults to True.
        :type add_to_menu: bool
        
        :param add_to_toolbar: Flag indicating whether the action should also
            be added to the toolbar. Defaults to True.
        :type add_to_toolbar: bool
        
        :param status_tip: Optional text to show in a popup when mouse pointer
            hovers over the action.
        :type status_tip: str
        
        :param parent: Parent widget for the new action. Defaults None.
        :type parent: QWidget
        
        :param whats_this: Optional text to show in the status bar when the
            mouse pointer hovers over the action.
        :type whats_this: str
        
        :returns: The action that was created.
        :rtype: QAction
        """
        icon = QIcon(icon_path)
        action = QAction(icon, text, parent)
        action.triggered.connect(callback)
        action.setEnabled(enabled_flag)

        if status_tip is not None:
            action.setStatusTip(status_tip)

        if whats_this is not None:
            action.setWhatsThis(whats_this)

        if add_to_toolbar:
            self.iface.addToolBarIcon(action)

        if add_to_menu:
            self.iface.addPluginToMenu(self.menu, action)

        self.actions.append(action)

        return action

    def initGui(self):
        """Create the menu entries and toolbar icons inside the QGIS GUI."""
        icon_path = os.path.join(self.plugin_dir, 'icons', 'icon.png')
        self.add_action(
            icon_path,
            text='Configure for QField',
            callback=self.run,
            parent=self.iface.mainWindow()
        )

    def unload(self):
        """Removes the plugin menu item and icon from QGIS GUI."""
        for action in self.actions:
            self.iface.removePluginMenu(self.menu, action)
            self.iface.removeToolBarIcon(action)

    def run(self):
        """Run method that performs all the real work"""
        # Get the current project
        project = QgsProject.instance()
        
        # Get all vector layers
        layers = [layer for layer in project.mapLayers().values() 
                 if hasattr(layer, 'editFormConfig')]
        
        if not layers:
            QMessageBox.information(
                self.iface.mainWindow(),
                'No Layers',
                'No vector layers found in the project.'
            )
            return
        
        # Configure each layer for QField
        for layer in layers:
            self.configure_layer_for_qfield(layer)
        
        QMessageBox.information(
            self.iface.mainWindow(),
            'Success',
            f'Configured {len(layers)} layer(s) for QField.'
        )

    def configure_layer_for_qfield(self, layer):
        """Configure a layer for optimal use in QField.
        
        :param layer: The layer to configure
        :type layer: QgsVectorLayer
        """
        # Get the edit form configuration
        form_config = layer.editFormConfig()
        
        # Set to use a generated UI (better for mobile)
        form_config.setLayout(QgsEditFormConfig.GeneratedLayout)
        
        # Apply the configuration
        layer.setEditFormConfig(form_config)
        
        # Save to project
        layer.triggerRepaint()
```

## Step 5: Create a Simple Icon

Create a simple icon file at `icons/icon.png`. You can use any 24x24 pixel PNG image. For testing, you can use a simple colored square or download a free icon.

Alternatively, create a placeholder:

```bash
# On Linux/Mac with ImageMagick installed
convert -size 24x24 xc:blue icons/icon.png

# Or just create an empty file for now
touch icons/icon.png
```

## Step 6: Install and Test Your Plugin

### Installation

1. **Copy to QGIS plugins directory:**

```bash
# Linux
cp -r my_first_qfield_plugin ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/

# Windows (PowerShell)
Copy-Item -Path my_first_qfield_plugin -Destination "$env:APPDATA\QGIS\QGIS3\profiles\default\python\plugins\" -Recurse

# Mac
cp -r my_first_qfield_plugin ~/Library/Application\ Support/QGIS/QGIS3/profiles/default/python/plugins/
```

2. **Enable in QGIS:**
   - Open QGIS
   - Go to **Plugins** → **Manage and Install Plugins**
   - Go to **Installed** tab
   - Check the box next to "My First QField Plugin"

### Testing

1. Create a new QGIS project
2. Add a vector layer (e.g., create a simple point layer)
3. Look for the plugin icon in the toolbar
4. Click the icon
5. You should see a success message

## Step 7: Enhance Your Plugin

Now that you have a working plugin, you can enhance it:

### Add Validation Rules

Extend the `configure_layer_for_qfield` method:

```python
def configure_layer_for_qfield(self, layer):
    """Configure a layer for optimal use in QField."""
    form_config = layer.editFormConfig()
    form_config.setLayout(QgsEditFormConfig.GeneratedLayout)
    
    # Add validation to numeric fields
    for idx, field in enumerate(layer.fields()):
        if field.type() in [2, 4, 6]:  # Numeric types
            layer.setConstraintExpression(
                idx,
                f'"{field.name()}" IS NOT NULL',
                f'{field.name()} is required'
            )
    
    layer.setEditFormConfig(form_config)
```

### Add Custom Form Widgets

Configure specific widgets for fields:

```python
from qgis.core import QgsEditorWidgetSetup

def configure_field_widget(layer, field_name, widget_type, config):
    """Configure widget for a specific field."""
    form_config = layer.editFormConfig()
    field_idx = layer.fields().indexOf(field_name)
    
    if field_idx >= 0:
        widget_setup = QgsEditorWidgetSetup(widget_type, config)
        form_config.setWidgetConfig(field_name, widget_setup.config())
        layer.setEditFormConfig(form_config)
```

## Common Issues and Solutions

### Plugin Not Appearing in QGIS

- Check that `__init__.py` and `metadata.txt` are in the plugin directory
- Verify plugin directory name doesn't contain spaces or special characters
- Check QGIS message log for errors (View → Panels → Log Messages)
- Restart QGIS after copying the plugin

### Icon Not Showing

- Verify icon path in the code matches actual file location
- Ensure icon file is a valid PNG
- Check file permissions

### Plugin Crashes QGIS

- Check Python syntax errors
- Review QGIS Python console for error messages
- Use try-except blocks for error handling

## Next Steps

Your first plugin is complete! Continue to [Working with QField Forms](05-qfield-forms.md) to learn more about customizing forms for mobile data collection.
