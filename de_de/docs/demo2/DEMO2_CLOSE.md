````markdown
# Schließen des Plugins: Senden von Signalen

Anstatt die Schaltfläche in der PluginToolbar zu verwenden, um das Plugin ein- und auszuschalten, verlassen wir uns jetzt auf den Kartenklick, um das Plugin zu öffnen, und ich habe eine Ok-Schaltfläche zur Plugin-Component hinzugefügt, um es zu schließen. Dies erstellt ein einfaches Beispiel zum Erstellen und Verwenden benutzerdefinierter Signale.


## Einrichtung des Signals von der Component in d2_plugin_compontent.qml

Genau wie Sie es in PyQt tun würden, wird unser Signal innerhalb unseres Component-Items definiert

```qml
Rectangle {
  id: pluginFrame
  signal closed()
}
```

Wir fügen dann eine Schaltfläche zum pluginFrame hinzu, die das Signal ausgibt, wenn darauf geklickt wird

```qml
Rectangle {
  id: pluginFrame
  signal closed()
  Button {
    id: closeButton
    onClicked: {
        closed()
    }
  }
}
```

## Verbinden des Signals im Plugin

In demo2_select.qml haben wir ein Connections-Item hinzugefügt, um das Signal von pluginFrame zu verbinden. Dies ist die 3. unterschiedliche Methode zur Handhabung von Signalen und Slots in QML.

```qml

Item {
  id: plugin
  Loader{
    id:pluginLoader
  }
  Connections {
    target: pluginLoader.item
    function onClosed(){
      pluginLoader.active = false
    }
  }
}
```

Dies entspricht dem PyQt

```python
 pluginLoader.item.closed.connect(onClosed)
 def onClosed(){
    pluginLoader.active=false
 }
```

Beachten Sie hier, dass die Verbindung zwischen Slot und Funktion einer obligatorischen Namenskonvention folgt, wobei, wenn der Signalname closed ist, der Slot-Name onClosed sein muss. Hey. Ich mache die Regeln nicht.

Ich könnte Ihnen etwas über Layouts erzählen, aber ich denke, Sie werden sehr gelangweilt sein, wenn ich das tue. Also lassen Sie uns bis zur Demonstration 4 überspringen und ich zeige Ihnen, wie man ein Feature hinzufügt und aktualisiert.

## 📚 **[Demonstration 4: Hinzufügen und Aktualisieren von Features](../demo4/DEMO4_INTRO.md)**
## 📚 **[<< Demo2 Handhabung der Plot-ID](DEMO2_MAP_CLICK.md)**
````