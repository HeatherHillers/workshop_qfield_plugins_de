# Setting Up Your Development Environment

This guide will help you set up your development environment for QField plugin development.

## Required Software

### 1. QGIS

Install the latest Long Term Release (LTR) version of QGIS:

- **Windows/Mac**: Download from [qgis.org](https://qgis.org/download/)
- **Linux**: Use your distribution's package manager or official QGIS repositories

```bash
# Ubuntu/Debian example
sudo apt install qgis qgis-plugin-grass
```

### 2. Python

QField plugins are written in Python. Ensure you have Python 3.x installed:

```bash
python3 --version
```

QGIS comes with its own Python interpreter, but having Python installed separately is useful for development.

### 3. Code Editor or IDE

Choose a code editor that supports Python development:

- **Visual Studio Code** (recommended): Lightweight with excellent Python support
- **PyCharm**: Full-featured Python IDE
- **Sublime Text**: Fast and customizable
- **Atom**: Open-source and extensible

### 4. QField App

Install QField on your mobile device:

- **Android**: Download from [Google Play Store](https://play.google.com/store/apps/details?id=ch.opengis.qfield)
- **iOS**: Download from [App Store](https://apps.apple.com/app/qfield-for-qgis/id1531726814)

For testing without a mobile device, you can use the QFieldCloud web interface or an Android emulator.

## Setting Up Your Development Workspace

### 1. Create a Project Directory

```bash
mkdir qfield-plugin-workshop
cd qfield-plugin-workshop
```

### 2. Clone the Example Repository

Clone the vegetation monitoring example repository:

```bash
git clone https://github.com/HeatherHillers/qfield_vegetation_monitoring.git
```

### 3. Set Up Python Virtual Environment (Optional but Recommended)

```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### 4. Install Required Python Packages

```bash
pip install qgis-plugin-tools
pip install pyqt5
```

## Configuring QGIS for Plugin Development

### 1. Enable Developer Tools

In QGIS:
1. Go to **Settings** → **Options**
2. Navigate to **System** tab
3. Check "Show debugging/development tools"

### 2. Set Up Plugin Path

QGIS loads plugins from specific directories:

- **Linux**: `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/`
- **Windows**: `C:\Users\<username>\AppData\Roaming\QGIS\QGIS3\profiles\default\python\plugins\`
- **Mac**: `~/Library/Application Support/QGIS/QGIS3/profiles/default/python/plugins/`

### 3. Enable Plugin Reloader

Install the "Plugin Reloader" plugin from the QGIS Plugin Manager to help with development:

1. Open **Plugins** → **Manage and Install Plugins**
2. Search for "Plugin Reloader"
3. Install and enable it

## Setting Up QFieldSync Plugin

QFieldSync is essential for preparing QGIS projects for QField:

1. Open **Plugins** → **Manage and Install Plugins**
2. Search for "QFieldSync"
3. Install and enable it

## Testing Your Setup

1. Open QGIS
2. Create a new project
3. Add a vector layer
4. Use QFieldSync to export the project for QField
5. Transfer the exported project to your mobile device (or test locally)

## Development Tools and Extensions

### Recommended VS Code Extensions

If using VS Code, install these extensions:

- Python
- Pylance
- Python Indent
- PyQt5 Tools
- GitLens

### QGIS Plugin Development Resources

- [PyQGIS Developer Cookbook](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/)
- [QGIS Python API Documentation](https://qgis.org/pyqgis/latest/)
- [QField Documentation](https://docs.qfield.org/)

## Troubleshooting

### QGIS Python Console Issues

If the Python console doesn't work:
1. Check QGIS installation
2. Verify Python path in QGIS settings
3. Reinstall QGIS if necessary

### Plugin Not Loading

If your plugin doesn't appear:
1. Check plugin directory path
2. Verify metadata.txt is correct
3. Check QGIS log for errors (View → Panels → Log Messages)

## Next Steps

Now that your environment is set up, continue to [Plugin Structure and Architecture](03-plugin-structure.md) to understand how QField plugins are organized.
