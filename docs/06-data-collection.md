# Data Collection and Validation

Robust data collection and validation are essential for ensuring high-quality field data. This guide covers best practices and implementation techniques.

## Data Collection Workflow

A typical QField data collection workflow:

1. **Prepare project in QGIS**: Configure layers, forms, and validation
2. **Export to QField**: Use QFieldSync to package the project
3. **Collect data in the field**: Use QField on mobile device
4. **Sync data back**: Transfer collected data to QGIS
5. **Validate and process**: Check data quality and process results

## Validation Strategies

### Client-Side Validation

Validate data directly in QField before submission:

```python
def setup_client_validation(layer):
    """Set up validation rules that run in QField."""
    
    # Required fields
    required_fields = ['species', 'date', 'location']
    for field_name in required_fields:
        field_idx = layer.fields().indexOf(field_name)
        if field_idx >= 0:
            layer.setFieldConstraint(
                field_idx,
                QgsFieldConstraints.ConstraintNotNull,
                QgsFieldConstraints.ConstraintStrengthHard
            )
    
    # Range validation
    add_range_constraint(layer, 'temperature', -50, 50, '°C')
    add_range_constraint(layer, 'humidity', 0, 100, '%')
    
    # Format validation
    add_expression_constraint(
        layer,
        'email',
        'regexp_match("email", \'^[^@]+@[^@]+\\.[^@]+$\')',
        'Invalid email format'
    )
```

### Server-Side Validation

Additional validation after data sync:

```python
def validate_synced_data(layer):
    """Validate data after syncing from QField."""
    validation_errors = []
    
    for feature in layer.getFeatures():
        # Check for spatial validity
        if not feature.geometry().isGeosValid():
            validation_errors.append({
                'feature_id': feature.id(),
                'error': 'Invalid geometry'
            })
        
        # Check business rules
        if feature['species'] == 'oak' and feature['height_m'] > 40:
            validation_errors.append({
                'feature_id': feature.id(),
                'error': 'Oak height exceeds typical maximum'
            })
    
    return validation_errors
```

## Common Validation Patterns

### 1. Required Fields

```python
def add_required_constraint(layer, field_name):
    """Make a field mandatory."""
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setFieldConstraint(
            field_idx,
            QgsFieldConstraints.ConstraintNotNull,
            QgsFieldConstraints.ConstraintStrengthHard
        )
```

### 2. Numeric Ranges

```python
def add_range_constraint(layer, field_name, min_val, max_val, unit=''):
    """Validate numeric field is within range."""
    expression = f'"{field_name}" >= {min_val} AND "{field_name}" <= {max_val}'
    message = f'{field_name} must be between {min_val} and {max_val}{unit}'
    
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setConstraintExpression(field_idx, expression, message)
```

### 3. Text Length

```python
def add_length_constraint(layer, field_name, min_len=0, max_len=255):
    """Validate text field length."""
    expression = f'length("{field_name}") >= {min_len} AND length("{field_name}") <= {max_len}'
    message = f'{field_name} must be between {min_len} and {max_len} characters'
    
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setConstraintExpression(field_idx, expression, message)
```

### 4. Date Validation

```python
def add_date_range_constraint(layer, field_name, min_date=None, max_date='now()'):
    """Validate date is within acceptable range."""
    if min_date:
        expression = f'"{field_name}" >= \'{min_date}\' AND "{field_name}" <= {max_date}'
    else:
        expression = f'"{field_name}" <= {max_date}'
    
    message = 'Date is out of acceptable range'
    
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setConstraintExpression(field_idx, expression, message)

# Example: Don't allow future dates
add_date_range_constraint(layer, 'survey_date')
```

### 5. Cross-Field Validation

```python
def add_cross_field_constraint(layer, field1, field2, operator='<'):
    """Validate relationship between two fields."""
    expression = f'"{field1}" {operator} "{field2}"'
    message = f'{field1} must be {operator} {field2}'
    
    field_idx = layer.fields().indexOf(field1)
    if field_idx >= 0:
        layer.setConstraintExpression(field_idx, expression, message)

# Example: start_date must be before end_date
add_cross_field_constraint(layer, 'start_date', 'end_date', '<')
```

### 6. Conditional Requirements

```python
def add_conditional_constraint(layer, field_name, condition, message):
    """Field required only when condition is met."""
    expression = f'{condition} OR "{field_name}" IS NOT NULL'
    
    field_idx = layer.fields().indexOf(field_name)
    if field_idx >= 0:
        layer.setConstraintExpression(field_idx, expression, message)

# Example: comments required when status is 'rejected'
add_conditional_constraint(
    layer,
    'comments',
    '"status" != \'rejected\'',
    'Comments required when status is rejected'
)
```

## Data Quality Checks

### Geometry Validation

```python
def validate_geometry_quality(layer):
    """Check geometry quality of all features."""
    issues = []
    
    for feature in layer.getFeatures():
        geom = feature.geometry()
        
        # Check if geometry is valid
        if not geom.isGeosValid():
            issues.append({
                'id': feature.id(),
                'type': 'invalid_geometry',
                'message': geom.lastError()
            })
        
        # Check for duplicate vertices
        if geom.type() == QgsWkbTypes.LineGeometry:
            if has_duplicate_vertices(geom):
                issues.append({
                    'id': feature.id(),
                    'type': 'duplicate_vertices',
                    'message': 'Geometry contains duplicate vertices'
                })
    
    return issues

def has_duplicate_vertices(geometry):
    """Check if geometry has duplicate consecutive vertices."""
    vertices = geometry.vertices()
    prev_vertex = None
    
    for vertex in vertices:
        if prev_vertex and vertex == prev_vertex:
            return True
        prev_vertex = vertex
    
    return False
```

### Attribute Completeness

```python
def check_data_completeness(layer, required_fields):
    """Check if required fields are filled."""
    incomplete_features = []
    
    for feature in layer.getFeatures():
        missing_fields = []
        
        for field_name in required_fields:
            value = feature[field_name]
            if value is None or value == '':
                missing_fields.append(field_name)
        
        if missing_fields:
            incomplete_features.append({
                'id': feature.id(),
                'missing': missing_fields
            })
    
    return incomplete_features
```

### Spatial Validation

```python
def validate_spatial_constraints(layer, boundary_layer=None):
    """Validate features are within expected spatial bounds."""
    errors = []
    
    for feature in layer.getFeatures():
        geom = feature.geometry()
        
        # Check if within boundary
        if boundary_layer:
            within_boundary = False
            for boundary_feat in boundary_layer.getFeatures():
                if boundary_feat.geometry().contains(geom):
                    within_boundary = True
                    break
            
            if not within_boundary:
                errors.append({
                    'id': feature.id(),
                    'error': 'Feature outside project boundary'
                })
    
    return errors
```

## Implementing a Validation Plugin

Here's a complete example of a validation plugin:

```python
# validation_plugin.py

from qgis.core import QgsProject, QgsMessageLog, Qgis
from qgis.PyQt.QtWidgets import QMessageBox

class ValidationPlugin:
    """Plugin to validate QField data."""
    
    def __init__(self, iface):
        self.iface = iface
        
    def initGui(self):
        """Initialize plugin GUI."""
        # Add menu action
        self.action = QAction("Validate QField Data", self.iface.mainWindow())
        self.action.triggered.connect(self.run_validation)
        self.iface.addPluginToMenu("QField Tools", self.action)
    
    def unload(self):
        """Remove plugin."""
        self.iface.removePluginMenu("QField Tools", self.action)
    
    def run_validation(self):
        """Run validation on all layers."""
        project = QgsProject.instance()
        validation_results = []
        
        for layer in project.mapLayers().values():
            if hasattr(layer, 'getFeatures'):
                results = self.validate_layer(layer)
                if results:
                    validation_results.append({
                        'layer': layer.name(),
                        'errors': results
                    })
        
        # Display results
        self.show_validation_results(validation_results)
    
    def validate_layer(self, layer):
        """Validate a single layer."""
        errors = []
        
        # Check geometry validity
        for feature in layer.getFeatures():
            if feature.hasGeometry():
                if not feature.geometry().isGeosValid():
                    errors.append({
                        'feature_id': feature.id(),
                        'error': 'Invalid geometry'
                    })
        
        # Check attribute constraints
        for feature in layer.getFeatures():
            for idx, field in enumerate(layer.fields()):
                # Check NOT NULL constraints
                if layer.fieldConstraints(idx) & QgsFieldConstraints.ConstraintNotNull:
                    if feature[field.name()] is None:
                        errors.append({
                            'feature_id': feature.id(),
                            'error': f'{field.name()} is required but empty'
                        })
        
        return errors
    
    def show_validation_results(self, results):
        """Display validation results to user."""
        if not results:
            QMessageBox.information(
                self.iface.mainWindow(),
                "Validation Complete",
                "No validation errors found!"
            )
        else:
            error_msg = "Validation Errors Found:\n\n"
            for result in results:
                error_msg += f"Layer: {result['layer']}\n"
                for error in result['errors'][:5]:  # Show first 5 errors
                    error_msg += f"  - Feature {error['feature_id']}: {error['error']}\n"
                if len(result['errors']) > 5:
                    error_msg += f"  ... and {len(result['errors']) - 5} more\n"
                error_msg += "\n"
            
            QMessageBox.warning(
                self.iface.mainWindow(),
                "Validation Errors",
                error_msg
            )
```

## Best Practices

1. **Validate Early**: Catch errors in QField before data is saved
2. **Clear Messages**: Provide clear, actionable error messages
3. **Multiple Layers**: Validate across related layers
4. **Performance**: Keep validation fast, especially for large datasets
5. **Offline Support**: Ensure validation works without network
6. **User Feedback**: Provide immediate feedback to field users
7. **Logging**: Log validation results for later review
8. **Flexibility**: Allow some rules to be warnings vs. hard errors

## Testing Validation Rules

Always test validation rules thoroughly:

```python
def test_validation():
    """Test validation rules."""
    # Create test layer
    layer = create_test_layer()
    
    # Add test features
    valid_feature = create_valid_feature()
    invalid_feature = create_invalid_feature()
    
    # Test validation
    assert validate_feature(valid_feature) == True
    assert validate_feature(invalid_feature) == False
```

## Next Steps

Continue to [Testing and Debugging](07-testing.md) to learn how to test and debug your QField plugins.
