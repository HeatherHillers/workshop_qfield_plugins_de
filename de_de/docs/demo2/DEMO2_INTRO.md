````markdown
# Demonstration 2: Feature-Auswahl 

Diese Demonstration baut auf Demo 1 auf, indem sie die Fähigkeit hinzufügt, ein Feature aus der Karte auszuwählen und Informationen aus diesem Feature abzurufen, um sie im Plugin zu verwenden.

## Was wir lernen werden

- Wie man über die QField-Schnittstelle auf einen Projekt-Layer zugreift
- Wie man Features aus einem Layer über die QField-Schnittstelle abfragt
- Wie man Objekte vom Map-Canvas mit einem Point-Handler auswählt
- Wie man ein Signal zum Schließen des Plugins sendet
- Wie man Items mit Layout-Containern ausrichtet

## Was macht es?
 
- Es öffnet sich, wenn der Benutzer doppelklickt auf ein Plotfeld auf dem Map-Canvas.
- Es zeigt die Plot-ID des ausgewählten Features im Textrahmen an.
- Ein einzelner Klick auf den Punkt führt zum üblichen Attributtabellen-Verhalten. (Auf iOS. Dies ist in der Windows-ausführbaren Datei nicht möglich.)

![plugin screen](img/demo2_screen.png)

- Es gibt auch Nachrichten an die DOS-Konsole anstelle des Benutzer-Nachrichtenprotokolls aus, wenn ein Karten-Feature ausgewählt wird. 

![alt text](img/demo2_log.png)

# Einrichtung

1. Erstellen Sie ein neues Projektverzeichnis: ${ROOT}/qfield_project_demo_2 
2. Kopieren Sie das gesamte Demo-Verzeichnis \${ROOT}/qfield_vegetation_plugin/demo2_selection nach ${ROOT}/qfield_project_demo_2
3. Öffnen Sie das Projekt in QGIS, wenn Sie möchten, aber Sie müssen nicht. Das Projekt hat sich seit Demo 1 nicht geändert.
4. Führen Sie QField von der Befehlszeile aus, um das Projekt direkt als lokales Projekt zu öffnen.
```dos
"C:\Program Files\QField\usr\bin\qfield.exe C:\temp\qfield\demo2_selection\demo2_selection.qgs
```


## 📚 **[Einrichtung der Auswahl von der Karte](DEMO2_MAP_CLICK.md)**
## 📚 **[<< Demonstration 1](../demo1/DEMO1_INTRO.md)**



````