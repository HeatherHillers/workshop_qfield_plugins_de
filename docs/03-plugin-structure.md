# Plugin Structure and Architecture

Understanding the structure of a QField plugin is essential for development. This guide explains the key components and architecture.

## Basic Plugin Structure

A typical QField/QGIS plugin has the following directory structure:

```
my_qfield_plugin/
├── __init__.py              # Plugin initialization
├── metadata.txt             # Plugin metadata
├── plugin_main.py           # Main plugin class
├── resources.qrc            # Qt resources
├── resources.py             # Compiled resources
├── ui/                      # User interface files
│   └── form.ui
├── icons/                   # Plugin icons
│   └── icon.png
├── scripts/                 # Python scripts
│   └── validation.py
└── README.md
```

## Essential Files

### 1. `__init__.py`

This file initializes the plugin and is required for Python to recognize the directory as a package.

```python
def classFactory(iface):
    from .plugin_main import MyQFieldPlugin
    return MyQFieldPlugin(iface)
```

### 2. `metadata.txt`

Contains metadata about your plugin:

```ini
[general]
name=My QField Plugin
description=A plugin for QField data collection
version=1.0
qgisMinimumVersion=3.0
author=Your Name
email=your.email@example.com

# Optional
homepage=https://github.com/username/my_qfield_plugin
repository=https://github.com/username/my_qfield_plugin
tracker=https://github.com/username/my_qfield_plugin/issues
icon=icons/icon.png
category=Plugins
tags=qfield,mobile,data collection
```

### 3. Main Plugin Class

The main plugin class (e.g., `plugin_main.py`) implements the plugin logic:

```python
from qgis.PyQt.QtWidgets import QAction
from qgis.PyQt.QtGui import QIcon

class MyQFieldPlugin:
    def __init__(self, iface):
        self.iface = iface
        self.plugin_dir = os.path.dirname(__file__)
        
    def initGui(self):
        """Create the menu entries and toolbar icons"""
        icon_path = os.path.join(self.plugin_dir, 'icons', 'icon.png')
        self.action = QAction(
            QIcon(icon_path),
            "My QField Plugin",
            self.iface.mainWindow()
        )
        self.action.triggered.connect(self.run)
        self.iface.addToolBarIcon(self.action)
        
    def unload(self):
        """Remove the plugin menu item and icon"""
        self.iface.removeToolBarIcon(self.action)
        
    def run(self):
        """Run method that performs all the plugin functionality"""
        # Your plugin code here
        pass
```

## QField-Specific Considerations

### Form Configuration

For QField plugins that customize forms, you'll often work with:

- **Form configurations**: XML or Python-based form definitions
- **Widget configurations**: Settings for individual form fields
- **Validation rules**: Python expressions or functions to validate input

### Project Configuration

QField plugins can modify QGIS project settings for mobile use:

```python
from qgis.core import QgsProject

def configure_for_qfield(project):
    """Configure project for QField"""
    # Enable offline editing
    project.writeEntry("Digitizing", "DefaultSnapTolerance", "10")
    project.writeEntry("Digitizing", "DefaultSnapToleranceUnit", "0")
    
    # Configure layers for offline use
    for layer in project.mapLayers().values():
        if layer.type() == QgsMapLayer.VectorLayer:
            layer.setEditFormConfig(configure_form(layer))
```

## Plugin Architecture Patterns

### 1. Model-View-Controller (MVC)

Separate your plugin logic into:
- **Model**: Data handling and business logic
- **View**: User interface components
- **Controller**: Coordinates between model and view

### 2. Event-Driven Architecture

QField plugins often respond to events:

```python
from qgis.core import QgsProject

class MyPlugin:
    def initGui(self):
        # Connect to signals
        QgsProject.instance().layersAdded.connect(self.on_layers_added)
        
    def on_layers_added(self, layers):
        """Handle layer addition event"""
        for layer in layers:
            # Configure layer for QField
            self.configure_layer(layer)
```

### 3. Plugin Manager Pattern

For complex plugins, use a manager to coordinate different components:

```python
class PluginManager:
    def __init__(self):
        self.validators = []
        self.form_configs = {}
        
    def register_validator(self, validator):
        self.validators.append(validator)
        
    def apply_validations(self, layer):
        for validator in self.validators:
            validator.apply(layer)
```

## Working with QField APIs

### Layer Configuration

```python
from qgis.core import QgsEditFormConfig, QgsEditorWidgetSetup

def configure_layer_form(layer):
    """Configure form for QField"""
    form_config = layer.editFormConfig()
    
    # Set form layout
    form_config.setLayout(QgsEditFormConfig.TabLayout)
    
    # Configure individual fields
    for field in layer.fields():
        widget_setup = QgsEditorWidgetSetup(
            'TextEdit',
            {'IsMultiline': False}
        )
        form_config.setWidgetConfig(field.name(), widget_setup.config())
        
    layer.setEditFormConfig(form_config)
```

### Data Validation

```python
from qgis.core import QgsExpression

def add_validation_constraint(layer, field_name, expression):
    """Add validation constraint to field"""
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setConstraintExpression(
            field_idx,
            expression,
            "Validation error message"
        )
```

## Best Practices

1. **Keep it Simple**: Mobile devices have limited resources
2. **Offline First**: Assume no network connectivity
3. **Touch-Friendly**: Design for touch interfaces
4. **Minimize Dependencies**: Reduce external library requirements
5. **Error Handling**: Implement robust error handling for field conditions
6. **Testing**: Test thoroughly on actual devices

## Resource Management

### Memory Management

Be mindful of memory usage on mobile devices:

```python
def process_large_dataset(layer):
    """Process features in chunks"""
    chunk_size = 100
    features = layer.getFeatures()
    
    while True:
        chunk = list(itertools.islice(features, chunk_size))
        if not chunk:
            break
        process_chunk(chunk)
```

### Battery Optimization

Minimize battery usage:
- Avoid continuous GPS polling
- Reduce unnecessary processing
- Cache data when possible

## Next Steps

Continue to [Building Your First Plugin](04-first-plugin.md) to create your first QField plugin.
