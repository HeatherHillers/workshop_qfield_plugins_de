# Get Ready To Go

- Display: docs/demo1/README.md

# Seup and sync the project
- Stage Dir: ${PROOT}/qfield_project_demo_1
     - contains project only
- Remember: Remove existing copies of workshop project from qfield.cloud 
- Display: Open QGIS with project
     - Show Attribute Table on plots, point out the plot_id field
     - Show Project Properties
- Demonstrate: Upload the project to qfield.cloud
- Demonstrate: Loading it into QField, using QField.exe
- Display: Project in QField.exe


# Lets look at the Plugin
- Display: VS Code with qfield_vegetation_monitoring
     - open browser to demo_1_hello/demo1_hello.qml

## Plugin Structure
- Narrative: 
     1. It is extremely important to get the structure right, or the plugin will not sync and load properly, and it wont necessarily tell you why.
     2. Your plugin starting point is a qml with the same name as the project and the project directory. (demo1_hello.qml)
     3. You can and should break up your plugin into multiple files.  All of them except demo1_hello.qml have to be in a subdirectory (components).
     4. The subdirectory must be added as an attachment directory to the project.
- Display:
    - In QGIS, show the project properties:QField, that components is an attachment directory.
- Narrative:
     5. It does not matter what you name your qml files in the components directory.
- Demonstrate:
    - Resync the Plugin and show the working Plugin in QField






## Table of Contents

1. [Introduction to QField Plugins](01-introduction.md)
2. [Setting Up Your Development Environment](02-setup.md)
3. [Plugin Structure and Architecture](03-plugin-structure.md)
4. [Building Your First Plugin](04-first-plugin.md)
5. [Working with QField Forms](05-qfield-forms.md)
6. [Data Collection and Validation](06-data-collection.md)
7. [Testing and Debugging](07-testing.md)
8. [Deployment and Distribution](08-deployment.md)
9. [Real-World Example: Vegetation Monitoring](09-vegetation-example.md)


## Prerequisites

- Basic knowledge of QGIS
- Familiarity with Python programming
- Understanding of GIS concepts
- QField app installed on a mobile device (optional but recommended)

## Getting Started

Begin with the [Introduction to QField Plugins](01-introduction.md) to understand the basics, then proceed through the guides in order.

## Additional Resources

- [QField Official Documentation](https://docs.qfield.org/)
- [QGIS Plugin Development](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/)
- [Vegetation Monitoring Example Repository](https://github.com/HeatherHillers/qfield_vegetation_monitoring)

## Workshop Materials

The workshop is based on the demonstrations available in the [qfield_vegetation_monitoring](https://github.com/HeatherHillers/qfield_vegetation_monitoring) repository. Please refer to that repository for working examples and code samples.

## Documentation



The documentation covers everything you need to know to develop QField plugins:

1. **[Introduction to QField Plugins](docs/01-introduction.md)** - Learn the basics of QField plugin development
2. **[Setting Up Your Development Environment](docs/02-setup.md)** - Get your tools and environment ready
3. **[Plugin Structure and Architecture](docs/03-plugin-structure.md)** - Understand how plugins are organized
4. **[Building Your First Plugin](docs/04-first-plugin.md)** - Create your first working plugin
5. **[Working with QField Forms](docs/05-qfield-forms.md)** - Design effective mobile data collection forms
6. **[Data Collection and Validation](docs/06-data-collection.md)** - Implement robust data validation
7. **[Testing and Debugging](docs/07-testing.md)** - Test and debug your plugins effectively
8. **[Deployment and Distribution](docs/08-deployment.md)** - Package and distribute your plugins
9. **[Real-World Example: Vegetation Monitoring](docs/09-vegetation-example.md)** - Complete practical implementation

## Quick Start

1. Clone this repository
2. Follow the [setup guide](docs/02-setup.md) to configure your environment
3. Work through the [first plugin tutorial](docs/04-first-plugin.md)
4. Study the [vegetation monitoring example](docs/09-vegetation-example.md)
5. Explore the [qfield_vegetation_monitoring repository](https://github.com/HeatherHillers/qfield_vegetation_monitoring) for working code


## Prerequisites

- Basic knowledge of QGIS
- Understanding of GIS concepts
- QField app installed on a mobile device (optional but recommended)

## Resources

- [QField Official Documentation](https://docs.qfield.org/)
- [QGIS Plugin Development](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/)
- [Example Repository: qfield_vegetation_monitoring](https://github.com/HeatherHillers/qfield_vegetation_monitoring)


