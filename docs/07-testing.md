# Testing and Debugging

Testing and debugging are crucial for developing reliable QField plugins. This guide covers techniques and best practices.

## Testing Strategy

A comprehensive testing strategy includes:

1. **Unit Tests**: Test individual functions
2. **Integration Tests**: Test component interactions
3. **Field Tests**: Test on actual devices in real conditions
4. **User Testing**: Test with actual users

## Setting Up Testing Environment

### Install Testing Tools

```bash
pip install pytest pytest-qgis pytest-cov
```

### Project Structure for Testing

```
my_qfield_plugin/
├── plugin/
│   ├── __init__.py
│   └── main.py
├── tests/
│   ├── __init__.py
│   ├── test_validation.py
│   └── test_forms.py
└── pytest.ini
```

### Configure pytest

Create `pytest.ini`:

```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```

## Unit Testing

### Basic Test Structure

```python
# tests/test_validation.py

import pytest
from qgis.core import QgsVectorLayer, QgsFeature, QgsField
from PyQt5.QtCore import QVariant

from plugin.validation import validate_numeric_range, add_required_field

class TestValidation:
    """Test validation functions."""
    
    @pytest.fixture
    def test_layer(self):
        """Create a test layer."""
        layer = QgsVectorLayer("Point?crs=epsg:4326", "test", "memory")
        layer.dataProvider().addAttributes([
            QgsField("name", QVariant.String),
            QgsField("age", QVariant.Int),
            QgsField("height", QVariant.Double)
        ])
        layer.updateFields()
        return layer
    
    def test_required_field(self, test_layer):
        """Test required field validation."""
        add_required_field(test_layer, "name")
        
        # Check constraint was added
        field_idx = test_layer.fields().indexOf("name")
        constraints = test_layer.fieldConstraints(field_idx)
        
        assert constraints & QgsFieldConstraints.ConstraintNotNull
    
    def test_numeric_range_valid(self, test_layer):
        """Test valid numeric range."""
        result = validate_numeric_range(25, 0, 100)
        assert result == True
    
    def test_numeric_range_invalid(self, test_layer):
        """Test invalid numeric range."""
        result = validate_numeric_range(150, 0, 100)
        assert result == False
```

### Testing Form Configuration

```python
# tests/test_forms.py

import pytest
from qgis.core import QgsVectorLayer, QgsEditFormConfig

from plugin.forms import configure_dropdown, configure_date_field

class TestForms:
    """Test form configuration functions."""
    
    @pytest.fixture
    def test_layer(self):
        """Create a test layer with fields."""
        layer = QgsVectorLayer("Point?crs=epsg:4326", "test", "memory")
        layer.dataProvider().addAttributes([
            QgsField("species", QVariant.String),
            QgsField("date", QVariant.Date)
        ])
        layer.updateFields()
        return layer
    
    def test_configure_dropdown(self, test_layer):
        """Test dropdown configuration."""
        options = {'Oak': 'oak', 'Pine': 'pine'}
        configure_dropdown(test_layer, 'species', options)
        
        form_config = test_layer.editFormConfig()
        widget_config = form_config.widgetConfig('species')
        
        assert 'map' in widget_config
        assert widget_config['map'] == options
    
    def test_configure_date_field(self, test_layer):
        """Test date field configuration."""
        configure_date_field(test_layer, 'date')
        
        form_config = test_layer.editFormConfig()
        widget_config = form_config.widgetConfig('date')
        
        assert 'display_format' in widget_config
        assert widget_config['display_format'] == 'yyyy-MM-dd'
```

## Integration Testing

Test how components work together:

```python
# tests/test_integration.py

import pytest
from qgis.core import QgsProject, QgsVectorLayer

from plugin.main import MyQFieldPlugin

class TestIntegration:
    """Test plugin integration."""
    
    @pytest.fixture
    def plugin(self, qgis_iface):
        """Create plugin instance."""
        return MyQFieldPlugin(qgis_iface)
    
    @pytest.fixture
    def project_with_layer(self):
        """Create project with test layer."""
        project = QgsProject.instance()
        layer = QgsVectorLayer("Point?crs=epsg:4326", "test", "memory")
        project.addMapLayer(layer)
        yield project, layer
        project.removeMapLayer(layer.id())
    
    def test_plugin_initialization(self, plugin):
        """Test plugin initializes correctly."""
        plugin.initGui()
        assert len(plugin.actions) > 0
    
    def test_configure_layer_workflow(self, plugin, project_with_layer):
        """Test complete layer configuration workflow."""
        project, layer = project_with_layer
        
        # Run plugin
        plugin.configure_layer_for_qfield(layer)
        
        # Verify configuration
        form_config = layer.editFormConfig()
        assert form_config.layout() == QgsEditFormConfig.GeneratedLayout
```

## Debugging Techniques

### 1. QGIS Python Console

Use the Python console in QGIS for interactive debugging:

```python
# In QGIS Python Console
from plugin.main import MyQFieldPlugin
iface = qgis.utils.iface
plugin = MyQFieldPlugin(iface)
plugin.run()
```

### 2. Print Debugging

Add debug prints to your code:

```python
def configure_layer(layer):
    """Configure layer with debug output."""
    print(f"Configuring layer: {layer.name()}")
    print(f"Field count: {layer.fields().count()}")
    
    for field in layer.fields():
        print(f"Field: {field.name()}, Type: {field.typeName()}")
    
    # Your code here
```

### 3. QGIS Message Log

Use QGIS message log for structured logging:

```python
from qgis.core import QgsMessageLog, Qgis

def log_debug(message):
    """Log debug message."""
    QgsMessageLog.logMessage(message, 'QField Plugin', Qgis.Info)

def log_error(message):
    """Log error message."""
    QgsMessageLog.logMessage(message, 'QField Plugin', Qgis.Critical)

# Usage
log_debug(f"Processing layer: {layer.name()}")
log_error(f"Failed to configure field: {field_name}")
```

### 4. Python Debugger (pdb)

Use Python's built-in debugger:

```python
import pdb

def complex_function(data):
    """Function with breakpoint."""
    pdb.set_trace()  # Execution will pause here
    
    result = process_data(data)
    return result
```

### 5. PyCharm Remote Debugging

Set up remote debugging in PyCharm:

```python
# Add to your plugin
import pydevd_pycharm

def run_with_debug(self):
    """Run with remote debugging."""
    pydevd_pycharm.settrace(
        'localhost',
        port=5678,
        stdoutToServer=True,
        stderrToServer=True
    )
    
    # Your code here
```

## Testing on Mobile Devices

### Preparation

1. **Export test project**: Use QFieldSync to export
2. **Transfer to device**: Use USB, cloud storage, or QFieldCloud
3. **Test scenarios**: Document test cases to execute

### Field Testing Checklist

- [ ] Form opens correctly
- [ ] All widgets display properly
- [ ] Touch interaction works
- [ ] Validation triggers appropriately
- [ ] Data saves correctly
- [ ] Works offline
- [ ] Photos/attachments work
- [ ] GPS accuracy acceptable
- [ ] Battery usage reasonable
- [ ] Performance acceptable

### Collecting Device Logs

#### Android

```bash
# Connect device via USB
adb logcat -s QField:* > qfield_log.txt
```

#### iOS

Use Xcode Console to view logs:
1. Connect device to Mac
2. Open Xcode
3. Window → Devices and Simulators
4. Select device
5. View console logs

## Common Issues and Solutions

### Issue: Plugin Not Loading

**Debug Steps:**
1. Check Python console for errors
2. Verify `__init__.py` and `metadata.txt`
3. Check file permissions
4. Review QGIS message log

```python
# Add to __init__.py for debugging
import traceback

def classFactory(iface):
    try:
        from .plugin_main import MyQFieldPlugin
        return MyQFieldPlugin(iface)
    except Exception as e:
        print(f"Error loading plugin: {e}")
        traceback.print_exc()
        raise
```

### Issue: Form Widget Not Appearing

**Debug Steps:**
1. Verify widget type spelling
2. Check widget configuration
3. Test in QGIS first
4. Check QField version compatibility

```python
def debug_widget_config(layer, field_name):
    """Debug widget configuration."""
    form_config = layer.editFormConfig()
    widget_config = form_config.widgetConfig(field_name)
    widget_type = form_config.widgetType(field_name)
    
    print(f"Field: {field_name}")
    print(f"Widget Type: {widget_type}")
    print(f"Widget Config: {widget_config}")
```

### Issue: Validation Not Working

**Debug Steps:**
1. Test expression in QGIS expression builder
2. Check field names and types
3. Verify constraint strength
4. Test with simple expression first

```python
def test_constraint_expression(layer, field_name):
    """Test if constraint expression is valid."""
    field_idx = layer.fields().indexOf(field_name)
    expression = layer.constraintExpression(field_idx)
    
    print(f"Field: {field_name}")
    print(f"Expression: {expression}")
    
    # Test expression
    from qgis.core import QgsExpression
    expr = QgsExpression(expression)
    if expr.hasParserError():
        print(f"Parser Error: {expr.parserErrorString()}")
    else:
        print("Expression is valid")
```

### Issue: Poor Mobile Performance

**Optimization Steps:**
1. Reduce layer complexity
2. Simplify expressions
3. Use appropriate zoom levels
4. Minimize external dependencies

```python
def optimize_for_mobile(layer):
    """Optimize layer for mobile performance."""
    # Simplify rendering
    layer.setSimplifyMethod(QgsVectorSimplifyMethod())
    
    # Set scale-based visibility
    layer.setScaleBasedVisibility(True)
    layer.setMinimumScale(100000)  # Don't show when zoomed out too far
```

## Automated Testing in CI/CD

### GitHub Actions Example

Create `.github/workflows/test.yml`:

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
    
    - name: Install dependencies
      run: |
        pip install pytest pytest-cov
        pip install qgis-testing
    
    - name: Run tests
      run: |
        pytest tests/ --cov=plugin --cov-report=xml
    
    - name: Upload coverage
      uses: codecov/codecov-action@v2
```

## Best Practices

1. **Write Tests First**: Test-driven development helps design better code
2. **Test on Real Devices**: Emulators don't catch all issues
3. **Automate Where Possible**: Use CI/CD for regression testing
4. **Document Test Cases**: Keep a test plan document
5. **Test Edge Cases**: Test with empty data, large datasets, etc.
6. **Test Offline Scenarios**: Ensure plugin works without connectivity
7. **Version Testing**: Test on different QGIS and QField versions
8. **User Acceptance Testing**: Get feedback from actual users

## Next Steps

Continue to [Deployment and Distribution](08-deployment.md) to learn how to package and distribute your plugin.
