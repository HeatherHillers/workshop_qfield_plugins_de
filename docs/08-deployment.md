# Deployment and Distribution

This guide covers packaging, distributing, and maintaining your QField plugin.

## Packaging Your Plugin

### Prepare for Distribution

1. **Clean up code**: Remove debug statements and test code
2. **Update metadata**: Ensure all information is accurate
3. **Add documentation**: Include README and user guide
4. **Test thoroughly**: Final testing on clean installation

### Required Files

Ensure your plugin includes:

```
my_qfield_plugin/
├── __init__.py              # Required
├── metadata.txt             # Required
├── plugin_main.py           # Required
├── README.md                # Recommended
├── LICENSE                  # Recommended
├── icon.png                 # Recommended
└── requirements.txt         # If needed
```

### Metadata.txt Checklist

Verify your `metadata.txt` includes:

```ini
[general]
name=My QField Plugin
qgisMinimumVersion=3.0
qgisMaximumVersion=3.99
description=Detailed description of your plugin
version=1.0.0
author=Your Name
email=your.email@example.com

about=Extended description with usage information

homepage=https://github.com/username/plugin
repository=https://github.com/username/plugin
tracker=https://github.com/username/plugin/issues

icon=icon.png
category=Plugins
tags=qfield,mobile,data collection,field work

experimental=False
deprecated=False

# Optional changelog
changelog=1.0.0
    - Initial release
    - Basic form configuration
    - Validation rules
```

## Creating a Plugin Archive

### Manual Packaging

Create a ZIP file of your plugin:

```bash
# Navigate to parent directory
cd /path/to/plugins/

# Create archive (exclude unwanted files)
zip -r my_qfield_plugin.zip my_qfield_plugin \
    -x "*.pyc" \
    -x "*__pycache__*" \
    -x "*.git*" \
    -x "*tests/*" \
    -x "*.spec"
```

### Automated Packaging Script

Create `build.sh`:

```bash
#!/bin/bash

PLUGIN_NAME="my_qfield_plugin"
VERSION=$(grep "version=" metadata.txt | cut -d'=' -f2)
OUTPUT="${PLUGIN_NAME}-${VERSION}.zip"

# Clean previous build
rm -f "${OUTPUT}"

# Create archive
zip -r "${OUTPUT}" ${PLUGIN_NAME} \
    -x "*.pyc" \
    -x "*__pycache__*" \
    -x "*.git*" \
    -x "*tests/*" \
    -x "*.DS_Store" \
    -x "*build.sh"

echo "Created ${OUTPUT}"
```

Make it executable:

```bash
chmod +x build.sh
./build.sh
```

### Python Setup Script

Create `setup.py` for Python packaging:

```python
from setuptools import setup, find_packages

setup(
    name='my-qfield-plugin',
    version='1.0.0',
    packages=find_packages(),
    include_package_data=True,
    install_requires=[
        # List dependencies
    ],
    author='Your Name',
    author_email='your.email@example.com',
    description='QField plugin for field data collection',
    url='https://github.com/username/my_qfield_plugin',
    classifiers=[
        'Development Status :: 4 - Beta',
        'Intended Audience :: End Users/Desktop',
        'Topic :: Scientific/Engineering :: GIS',
        'License :: OSI Approved :: GNU General Public License v3 (GPLv3)',
        'Programming Language :: Python :: 3',
    ],
)
```

## Distribution Channels

### 1. QGIS Plugin Repository

The official way to distribute QGIS plugins:

**Steps:**
1. Create account at https://plugins.qgis.org/
2. Upload your plugin ZIP file
3. Fill in plugin details
4. Submit for approval

**Repository Structure:**
```
plugins.qgis.org/
└── username/
    └── my_qfield_plugin/
        ├── versions/
        │   ├── 1.0.0.zip
        │   └── 1.1.0.zip
        └── metadata.ini
```

### 2. GitHub Releases

Distribute via GitHub:

**Steps:**
1. Tag your release
2. Create release on GitHub
3. Upload plugin ZIP as release asset

```bash
# Tag release
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# Create release on GitHub web interface
# Upload my_qfield_plugin-1.0.0.zip
```

### 3. Custom Plugin Repository

Host your own plugin repository:

**Create `plugins.xml`:**
```xml
<?xml version="1.0"?>
<plugins>
  <pyqgis_plugin name="My QField Plugin" version="1.0.0">
    <description>QField plugin for field data collection</description>
    <about>Extended description</about>
    <version>1.0.0</version>
    <qgis_minimum_version>3.0</qgis_minimum_version>
    <homepage>https://github.com/username/plugin</homepage>
    <file_name>my_qfield_plugin-1.0.0.zip</file_name>
    <icon>https://example.com/icon.png</icon>
    <author_name>Your Name</author_name>
    <download_url>https://example.com/plugins/my_qfield_plugin-1.0.0.zip</download_url>
    <uploaded_by>username</uploaded_by>
    <create_date>2024-01-01</create_date>
    <update_date>2024-01-01</update_date>
    <experimental>False</experimental>
    <deprecated>False</deprecated>
    <tracker>https://github.com/username/plugin/issues</tracker>
    <repository>https://github.com/username/plugin</repository>
    <tags>qfield,mobile,field work</tags>
  </pyqgis_plugin>
</plugins>
```

**Add to QGIS:**
1. Settings → Options → Plugins
2. Add: https://example.com/plugins/plugins.xml

## Version Management

### Semantic Versioning

Use semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR**: Incompatible API changes
- **MINOR**: New functionality (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

Examples:
- 1.0.0: Initial release
- 1.1.0: Add new features
- 1.1.1: Fix bugs
- 2.0.0: Breaking changes

### Version Update Checklist

When releasing a new version:

- [ ] Update version in `metadata.txt`
- [ ] Update changelog in `metadata.txt`
- [ ] Update version in `README.md`
- [ ] Update any version constants in code
- [ ] Tag release in git
- [ ] Update documentation
- [ ] Test installation from archive

## Installation Methods

### Manual Installation

Users can install manually:

1. Download ZIP file
2. Extract to QGIS plugins directory
3. Restart QGIS
4. Enable in Plugin Manager

### Via Plugin Manager

If in official repository:

1. Open QGIS
2. Plugins → Manage and Install Plugins
3. Search for plugin name
4. Click Install

### Command Line Installation

```bash
# Download plugin
wget https://example.com/my_qfield_plugin-1.0.0.zip

# Extract to plugins directory
unzip my_qfield_plugin-1.0.0.zip -d ~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/
```

## Documentation

### README.md Structure

```markdown
# My QField Plugin

Brief description of what your plugin does.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

Instructions for installation

## Usage

Step-by-step usage guide

## Requirements

- QGIS 3.0 or higher
- QField 2.0 or higher

## Configuration

How to configure the plugin

## Troubleshooting

Common issues and solutions

## Contributing

How others can contribute

## License

License information

## Credits

Acknowledgments
```

### User Guide

Create a comprehensive user guide:

```
docs/
├── installation.md
├── quick-start.md
├── configuration.md
├── features.md
└── troubleshooting.md
```

### API Documentation

Document your code for developers:

```python
def configure_layer(layer, options):
    """Configure a layer for QField use.
    
    Args:
        layer (QgsVectorLayer): The layer to configure
        options (dict): Configuration options with keys:
            - form_layout (str): 'generated' or 'tab'
            - validation (bool): Enable validation rules
            - offline (bool): Configure for offline use
    
    Returns:
        bool: True if successful, False otherwise
    
    Raises:
        ValueError: If layer is not a valid vector layer
        
    Example:
        >>> layer = iface.activeLayer()
        >>> options = {'form_layout': 'tab', 'validation': True}
        >>> configure_layer(layer, options)
        True
    """
    pass
```

## Maintenance

### Bug Tracking

Set up issue tracking:

1. Use GitHub Issues
2. Provide issue template
3. Label issues (bug, enhancement, question)
4. Respond promptly

**Issue Template (.github/ISSUE_TEMPLATE/bug_report.md):**
```markdown
## Bug Description
A clear description of the bug

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
What should happen

## Actual Behavior
What actually happens

## Environment
- QGIS Version:
- QField Version:
- Plugin Version:
- Operating System:

## Additional Context
Any other relevant information
```

### Update Process

Regular maintenance schedule:

1. **Monthly**: Check for issues and questions
2. **Quarterly**: Update dependencies
3. **Annually**: Major version updates

### Deprecation Policy

When deprecating features:

1. Mark as deprecated in code
2. Add deprecation warning
3. Update documentation
4. Provide migration guide
5. Remove in next major version

```python
import warnings

def old_function():
    """Deprecated function.
    
    .. deprecated:: 1.5.0
        Use :func:`new_function` instead.
    """
    warnings.warn(
        "old_function is deprecated, use new_function instead",
        DeprecationWarning,
        stacklevel=2
    )
    return new_function()
```

## Continuous Integration

### GitHub Actions for Releases

`.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Get version
      id: version
      run: echo "::set-output name=version::${GITHUB_REF#refs/tags/v}"
    
    - name: Create plugin archive
      run: |
        ./build.sh
    
    - name: Create Release
      uses: actions/create-release@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ github.ref }}
        release_name: Release ${{ steps.version.outputs.version }}
        draft: false
        prerelease: false
    
    - name: Upload Release Asset
      uses: actions/upload-release-asset@v1
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        upload_url: ${{ steps.create_release.outputs.upload_url }}
        asset_path: ./my_qfield_plugin-${{ steps.version.outputs.version }}.zip
        asset_name: my_qfield_plugin-${{ steps.version.outputs.version }}.zip
        asset_content_type: application/zip
```

## Best Practices

1. **Version Control**: Use git for all development
2. **Semantic Versioning**: Follow semver principles
3. **Testing**: Test before each release
4. **Documentation**: Keep docs up to date
5. **Changelog**: Maintain detailed changelog
6. **Compatibility**: Test with multiple QGIS/QField versions
7. **Security**: Regular security audits
8. **Backup**: Keep backups of all releases
9. **Communication**: Announce updates to users
10. **Feedback**: Actively seek and respond to user feedback

## Next Steps

Continue to [Real-World Example: Vegetation Monitoring](09-vegetation-example.md) to see a complete, practical implementation.
