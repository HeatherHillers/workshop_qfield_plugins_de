# Get Ready To Go

1. Attain some inner peace and hold on to it.
2. Choose a development directory, which will be refered to as ${ROOT}
2. clone or download this repository http://github.com/HeatherHillers/workshop_qfield_plugins_de under ${ROOT}
3. clone or download the qfield vegetation monitoring plugin repository from http://github.com/HeatherHillers/qfield_vegetation_monitoring under ${ROOT}
4. create the project directory: ${ROOT}/qfield_project_demo_1 
5. Open up QGIS and QField and log in to your account at https://qfield.cloud


# Setup and sync the vegetation project to QField
1. Copy the project files and project data to ${ROOT}/qfield_project_demo_1
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qgs
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qgs.png
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello_attachments.zip
      - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/vegetation.gpkg
2. Open the project in QGIS.
      - you should see just a single layer (plots) with a few points in it.  This is your selection layer.
      - Each of these points has a plot_id.  This is the field queried by the plugin.
3. Sync the project to your QFieldCloud account using the QField Sync Plugin.
4. Load the project in the Windows QField
      - this project is formatted for an iPad screen, so you are better off using your desktop QField to show the project. Unless you have an iPad.

# Lets look the Plugin Structure
1. Open ${ROOT}\qfield_vegetation_monitoring in Visual Studio Code or your prefered IDE.

## Critical Points
    - main qml must have same name as project and is saved in the project directory with the project file
    - all other qml is in the components subdirectory
    - this subdirectory will be added as an attachment directory in the project

2. Copy the plugin files to ${ROOT}/qfield_project_demo_1
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/demo1_hello.qml
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components
   - ${ROOT}/qfield_vegetation_monitoring/demo1_hello/components/d1_plugin_component.qml


3. Reload the Plugin in QGIS
4. Resync the Plugin in QField Sync
5. Synchronise your Plugin in QField

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


