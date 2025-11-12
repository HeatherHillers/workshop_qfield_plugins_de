# Real-World Example: Vegetation Monitoring

This guide presents a complete real-world example based on the [qfield_vegetation_monitoring](https://github.com/HeatherHillers/qfield_vegetation_monitoring) repository.

## Project Overview

The vegetation monitoring example demonstrates a practical QField plugin for collecting botanical field data. It showcases:

- Custom form widgets for species identification
- Validation rules for measurements
- Photo documentation
- GPS accuracy tracking
- Offline data collection
- Data quality assurance

## Project Structure

```
qfield_vegetation_monitoring/
├── plugin/
│   ├── __init__.py
│   ├── metadata.txt
│   ├── vegetation_plugin.py
│   └── validation/
│       ├── __init__.py
│       └── validators.py
├── projects/
│   ├── vegetation_survey.qgs
│   └── data/
│       ├── survey_points.gpkg
│       └── species_list.csv
├── forms/
│   └── vegetation_form.ui
└── docs/
    └── user_guide.md
```

## Use Case: Forest Survey

### Scenario

Field teams need to survey forest plots to:
- Identify and count tree species
- Measure tree dimensions
- Assess health conditions
- Document location with photos
- Ensure data quality in the field

### Requirements

- Work completely offline
- Easy to use on mobile devices
- Minimize data entry errors
- Support photo documentation
- Validate measurements in real-time

## Implementation

### 1. Layer Configuration

Create a point layer for vegetation observations:

```python
from qgis.core import (
    QgsVectorLayer,
    QgsField,
    QgsProject
)
from PyQt5.QtCore import QVariant

def create_vegetation_layer():
    """Create vegetation survey layer."""
    # Create layer
    layer = QgsVectorLayer(
        "Point?crs=EPSG:4326&field=id:integer&field=species:string(50)&"
        "field=common_name:string(100)&field=height_m:double&"
        "field=dbh_cm:double&field=health:integer&field=notes:string(500)&"
        "field=photo:string(255)&field=surveyor:string(50)&"
        "field=survey_date:date&field=gps_accuracy:double",
        "Vegetation Survey",
        "memory"
    )
    
    if not layer.isValid():
        raise Exception("Failed to create layer")
    
    # Add to project
    QgsProject.instance().addMapLayer(layer)
    
    return layer
```

### 2. Form Configuration

Configure the data entry form:

```python
from qgis.core import (
    QgsEditFormConfig,
    QgsEditorWidgetSetup,
    QgsDefaultValue,
    QgsAttributeEditorContainer,
    QgsAttributeEditorField
)

def configure_vegetation_form(layer):
    """Configure form for vegetation survey."""
    form_config = layer.editFormConfig()
    
    # Species dropdown from CSV
    species_list = load_species_list()
    species_map = {
        f"{sp['common_name']} ({sp['scientific_name']})": sp['code']
        for sp in species_list
    }
    
    widget_setup = QgsEditorWidgetSetup(
        'ValueMap',
        {'map': species_map}
    )
    form_config.setWidgetConfig('species', widget_setup.config())
    
    # Height measurement with range slider
    height_setup = QgsEditorWidgetSetup(
        'Range',
        {
            'Min': 0,
            'Max': 50,
            'Step': 0.5,
            'Style': 'Slider',
            'Suffix': ' m'
        }
    )
    form_config.setWidgetConfig('height_m', height_setup.config())
    
    # DBH (Diameter at Breast Height) measurement
    dbh_setup = QgsEditorWidgetSetup(
        'Range',
        {
            'Min': 0,
            'Max': 200,
            'Step': 1,
            'Style': 'Slider',
            'Suffix': ' cm'
        }
    )
    form_config.setWidgetConfig('dbh_cm', dbh_setup.config())
    
    # Health condition dropdown
    health_map = {
        'Excellent - No visible issues': 5,
        'Good - Minor issues': 4,
        'Fair - Moderate issues': 3,
        'Poor - Significant issues': 2,
        'Critical - Dying or dead': 1
    }
    health_setup = QgsEditorWidgetSetup(
        'ValueMap',
        {'map': health_map}
    )
    form_config.setWidgetConfig('health', health_setup.config())
    
    # Notes field - multiline text
    notes_setup = QgsEditorWidgetSetup(
        'TextEdit',
        {
            'IsMultiline': True,
            'UseHtml': False
        }
    )
    form_config.setWidgetConfig('notes', notes_setup.config())
    
    # Photo attachment
    photo_setup = QgsEditorWidgetSetup(
        'ExternalResource',
        {
            'DocumentViewer': 1,
            'RelativeStorage': 1,
            'FileWidget': True,
            'UseLink': False,
            'FullUrl': False
        }
    )
    form_config.setWidgetConfig('photo', photo_setup.config())
    
    # Survey date with default value
    date_setup = QgsEditorWidgetSetup(
        'DateTime',
        {
            'display_format': 'yyyy-MM-dd',
            'field_format': 'yyyy-MM-dd',
            'calendar_popup': True
        }
    )
    form_config.setWidgetConfig('survey_date', date_setup.config())
    
    # Set default values
    layer.setDefaultValueDefinition(
        layer.fields().indexOf('survey_date'),
        QgsDefaultValue('now()')
    )
    layer.setDefaultValueDefinition(
        layer.fields().indexOf('surveyor'),
        QgsDefaultValue('@user_account_name')
    )
    layer.setDefaultValueDefinition(
        layer.fields().indexOf('gps_accuracy'),
        QgsDefaultValue('@gps_accuracy')
    )
    
    # Create tabbed form layout
    form_config.clearTabs()
    form_config.setLayout(QgsEditFormConfig.TabLayout)
    
    root = form_config.invisibleRootContainer()
    root.clear()
    
    # Tab 1: Identification
    tab_id = QgsAttributeEditorContainer('Identification', root)
    for field_name in ['species', 'common_name']:
        idx = layer.fields().indexOf(field_name)
        if idx >= 0:
            tab_id.addChildElement(QgsAttributeEditorField(field_name, idx, tab_id))
    root.addChildElement(tab_id)
    
    # Tab 2: Measurements
    tab_measure = QgsAttributeEditorContainer('Measurements', root)
    for field_name in ['height_m', 'dbh_cm', 'health']:
        idx = layer.fields().indexOf(field_name)
        if idx >= 0:
            tab_measure.addChildElement(QgsAttributeEditorField(field_name, idx, tab_measure))
    root.addChildElement(tab_measure)
    
    # Tab 3: Documentation
    tab_doc = QgsAttributeEditorContainer('Documentation', root)
    for field_name in ['notes', 'photo', 'survey_date', 'surveyor', 'gps_accuracy']:
        idx = layer.fields().indexOf(field_name)
        if idx >= 0:
            tab_doc.addChildElement(QgsAttributeEditorField(field_name, idx, tab_doc))
    root.addChildElement(tab_doc)
    
    layer.setEditFormConfig(form_config)

def load_species_list():
    """Load species list from CSV."""
    import csv
    
    species = []
    with open('data/species_list.csv', 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            species.append(row)
    
    return species
```

### 3. Validation Rules

Implement comprehensive validation:

```python
from qgis.core import QgsFieldConstraints

def add_vegetation_validation(layer):
    """Add validation rules for vegetation data."""
    
    # Required fields
    required_fields = ['species', 'survey_date', 'surveyor']
    for field_name in required_fields:
        idx = layer.fields().indexOf(field_name)
        if idx >= 0:
            layer.setFieldConstraint(
                idx,
                QgsFieldConstraints.ConstraintNotNull,
                QgsFieldConstraints.ConstraintStrengthHard
            )
    
    # Height validation (must be positive, reasonable maximum)
    height_idx = layer.fields().indexOf('height_m')
    if height_idx >= 0:
        layer.setConstraintExpression(
            height_idx,
            '"height_m" > 0 AND "height_m" <= 50',
            'Height must be between 0 and 50 meters'
        )
    
    # DBH validation
    dbh_idx = layer.fields().indexOf('dbh_cm')
    if dbh_idx >= 0:
        layer.setConstraintExpression(
            dbh_idx,
            '"dbh_cm" >= 0 AND "dbh_cm" <= 200',
            'DBH must be between 0 and 200 cm'
        )
    
    # Health rating validation
    health_idx = layer.fields().indexOf('health')
    if health_idx >= 0:
        layer.setConstraintExpression(
            health_idx,
            '"health" >= 1 AND "health" <= 5',
            'Health rating must be between 1 and 5'
        )
    
    # GPS accuracy warning (soft constraint)
    gps_idx = layer.fields().indexOf('gps_accuracy')
    if gps_idx >= 0:
        layer.setConstraintExpression(
            gps_idx,
            '"gps_accuracy" <= 10',
            'Warning: GPS accuracy > 10m (consider retaking measurement)'
        )
        layer.setFieldConstraint(
            gps_idx,
            QgsFieldConstraints.ConstraintExpression,
            QgsFieldConstraints.ConstraintStrengthSoft
        )
    
    # Survey date validation (no future dates)
    date_idx = layer.fields().indexOf('survey_date')
    if date_idx >= 0:
        layer.setConstraintExpression(
            date_idx,
            '"survey_date" <= now()',
            'Survey date cannot be in the future'
        )
```

### 4. Complete Plugin Class

```python
# vegetation_plugin.py

import os
from qgis.PyQt.QtWidgets import QAction, QMessageBox
from qgis.PyQt.QtGui import QIcon
from qgis.core import QgsProject

class VegetationMonitoringPlugin:
    """QField plugin for vegetation monitoring."""
    
    def __init__(self, iface):
        self.iface = iface
        self.plugin_dir = os.path.dirname(__file__)
    
    def initGui(self):
        """Initialize plugin GUI."""
        icon_path = os.path.join(self.plugin_dir, 'icon.png')
        
        # Action to configure vegetation layer
        self.action_configure = QAction(
            QIcon(icon_path),
            'Configure Vegetation Layer',
            self.iface.mainWindow()
        )
        self.action_configure.triggered.connect(self.configure_layer)
        self.iface.addPluginToMenu('Vegetation Monitoring', self.action_configure)
        
        # Action to validate data
        self.action_validate = QAction(
            'Validate Survey Data',
            self.iface.mainWindow()
        )
        self.action_validate.triggered.connect(self.validate_data)
        self.iface.addPluginToMenu('Vegetation Monitoring', self.action_validate)
    
    def unload(self):
        """Remove plugin."""
        self.iface.removePluginMenu('Vegetation Monitoring', self.action_configure)
        self.iface.removePluginMenu('Vegetation Monitoring', self.action_validate)
    
    def configure_layer(self):
        """Configure active layer for vegetation survey."""
        layer = self.iface.activeLayer()
        
        if not layer:
            QMessageBox.warning(
                self.iface.mainWindow(),
                'No Layer',
                'Please select a layer first.'
            )
            return
        
        try:
            configure_vegetation_form(layer)
            add_vegetation_validation(layer)
            
            QMessageBox.information(
                self.iface.mainWindow(),
                'Success',
                f'Layer "{layer.name()}" configured for vegetation survey.'
            )
        except Exception as e:
            QMessageBox.critical(
                self.iface.mainWindow(),
                'Error',
                f'Failed to configure layer: {str(e)}'
            )
    
    def validate_data(self):
        """Validate collected data."""
        layer = self.iface.activeLayer()
        
        if not layer:
            QMessageBox.warning(
                self.iface.mainWindow(),
                'No Layer',
                'Please select a layer first.'
            )
            return
        
        errors = []
        for feature in layer.getFeatures():
            # Check geometry
            if not feature.hasGeometry() or not feature.geometry().isGeosValid():
                errors.append(f"Feature {feature.id()}: Invalid geometry")
            
            # Check required fields
            if not feature['species']:
                errors.append(f"Feature {feature.id()}: Missing species")
            
            # Check GPS accuracy
            if feature['gps_accuracy'] and feature['gps_accuracy'] > 10:
                errors.append(f"Feature {feature.id()}: Poor GPS accuracy ({feature['gps_accuracy']}m)")
        
        if errors:
            msg = "Validation Issues:\n\n" + "\n".join(errors[:10])
            if len(errors) > 10:
                msg += f"\n\n... and {len(errors) - 10} more issues"
            QMessageBox.warning(
                self.iface.mainWindow(),
                'Validation Results',
                msg
            )
        else:
            QMessageBox.information(
                self.iface.mainWindow(),
                'Validation Results',
                'All data passed validation!'
            )
```

## Field Workflow

### 1. Preparation (In Office)

1. Create QGIS project with vegetation layer
2. Run plugin to configure form and validation
3. Add base layers (aerial imagery, boundaries)
4. Use QFieldSync to export project
5. Transfer to mobile device

### 2. Data Collection (In Field)

1. Open project in QField
2. Navigate to survey location
3. Add new point feature
4. Fill in form:
   - Select species from dropdown
   - Measure and record height
   - Measure and record DBH
   - Assess health condition
   - Take photo
   - Add notes
5. Save feature
6. Validation runs automatically
7. Repeat for all survey points

### 3. Data Processing (Back in Office)

1. Transfer data back to QGIS
2. Run validation plugin
3. Fix any issues
4. Generate reports
5. Export to desired formats

## Key Features Demonstrated

### 1. Species Database

Pre-populated dropdown prevents typos and ensures consistency:

```csv
# species_list.csv
code,scientific_name,common_name
QUER_ROBU,Quercus robur,English Oak
PINU_SYLV,Pinus sylvestris,Scots Pine
BETU_PEND,Betula pendula,Silver Birch
```

### 2. Measurement Tools

Range sliders make measurements quick and reduce errors:
- Visual feedback
- Predefined ranges
- Step increments
- Units displayed

### 3. Photo Documentation

Linked photos provide visual evidence:
- Automatic naming
- Relative paths for portability
- Thumbnail preview
- Full-resolution storage

### 4. Quality Assurance

Multiple levels of validation:
- Required fields (hard constraint)
- Measurement ranges (hard constraint)
- GPS accuracy (soft warning)
- Date validation (hard constraint)

### 5. Default Values

Auto-populated fields reduce data entry:
- Current date/time
- Username
- GPS accuracy
- Device ID

## Best Practices Demonstrated

1. **User-Friendly Forms**: Organized into logical tabs
2. **Data Validation**: Multiple layers of validation
3. **Offline Capability**: Works completely offline
4. **Photo Documentation**: Integrated photo capture
5. **GPS Awareness**: Tracks and validates GPS accuracy
6. **Consistent Data**: Controlled vocabularies via dropdowns
7. **Audit Trail**: Auto-capture of surveyor and date
8. **Error Prevention**: Validation rules prevent common mistakes

## Extending the Example

You can extend this example by:

1. **Adding More Measurements**: Crown width, canopy condition, etc.
2. **Multiple Photos**: Support multiple photos per tree
3. **Site Information**: Add related table for plot-level data
4. **Species Identification Aid**: Images or descriptions in dropdown
5. **Offline Basemaps**: Include offline map tiles
6. **Data Export**: Custom export formats for analysis
7. **Reports**: Generate field summary reports
8. **Integration**: Connect to external databases

## Learning Resources

To learn more and see the complete implementation:

1. Visit the [qfield_vegetation_monitoring repository](https://github.com/HeatherHillers/qfield_vegetation_monitoring)
2. Review the complete source code
3. Try the example project
4. Adapt for your own use case

## Conclusion

This vegetation monitoring example demonstrates how QField plugins can:
- Streamline field data collection
- Ensure data quality
- Work efficiently offline
- Provide user-friendly interfaces
- Support complex workflows

Use this as a template for your own field data collection projects!
