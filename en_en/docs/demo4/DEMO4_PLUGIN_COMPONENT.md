# Demonstration 4: Getting from the Map Click to the Header Page.

Lets find our place really quick.

## 1. The demo4_header_form.qml is the same as in demo2.  When the user clicks on the map, the plugin sets the plotId property of the plugin component defined in components/d4_plugin_component.qml

```qml
// if you click on a plot
// (around line 76)
pluginLoader.active = true
pluginLoader.item.plotId = feature.attribute("plot_id")
```

## 2. d4_plugin_component.qml has changed.  It now has 2 of it's own loaders.  

- One Loader loads the d4_titlebar.qml, which we will ignore. This holds the title text and the close button.  It is basically our component in demo2, but shrunk down.  We will ignore this.

- The second loader is the one that we care about.  It loads the d4_tabwidget. As with the titlebar, when the widget is loaded, it passes the plotId down to the tabwidget.

```qml
    // TabWidget to display search results
    Loader {
        id: tabWidgetLoader
        // define the plotId property here to propagate to the Loader children
        property string plotId: pluginFrame.plotId
        width: pluginFrame.width  // Use parent (Column) width
        height: pluginFrame.height - titleBarLoader.height - 20 // Fill remaining Column space
        source: "d4_tabwidget.qml"
    }
```
### Important tangent: Propagating plotId with dynamic bindings

Inside of the d4_tabwidget, the plotId is dynamically bound to the parent's id.  But it can only reference its parent.

```qml
property string plotId: parent.plotId
```

It cannot reference the pluginFrame by name.

```qml
// no.  won't work.
property string plotId: pluginFrame.plotId
```

It only has a reference to it's parent, which is the tabWidgetLoader.  So, you have to bind plotId by defining it in the Loader.

```qml
    // TabWidget to display search results
    Loader {
        id: tabWidgetLoader
        // define the plotId property here to propagate to the Loader children
        property string plotId: pluginFrame.plotId
        <...>
        source: "d4_tabwidget.qml"
    }
```

Any properties you want to pass should be linked in this manner.

## 3. d4_tabwidget.qml loads the header page.

The tab widget defines a TabBar and a SwipeView that are synchronised with each other.  The SwipeView swipes between 6 pages.  The last 5 pages are strata pages.  Here is where the species will be inventoried.  The 1st page of the swipe view is the one that we care about.  it is the headerPageLoader, which is wrapped in a ScrollView.  The headerPageLoader loads d4_headerpage.qml.  Notice the propagation of plotId from the tabWidget through to the d4_headerpage via dynamic binding.

```qml
    TabBar {
        <...> 
    } // tabBar
    SwipeView {
        id: swipeView
        <...>    
        // Header Tab Content (first item)
        ScrollView {
            id: headerScrollView
            <...>
            
            Loader {
                id: headerPageLoader
                property string plotId: tabWidget.plotId
                source: "d4_headerpage.qml"
                width: headerScrollView.availableWidth
            }
        }

        // Strata Tab Content (remaining items.)
        Repeater {
            id: strataRepeater
            <...>
        }       
    }
```

## 4. And that brings us to the d4_headerpage, which loads the form in the first page of the tabWidget.

## 📚 **[The Header Page](DEMO4_HEADERPAGE.md)**
## 📚 **[<< Demonstration 4 Introduction](DEMO4_INTRO.md)**