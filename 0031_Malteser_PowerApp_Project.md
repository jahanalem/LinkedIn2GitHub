# 🚀 Neues Projekt: Malteser Tandem App (WIP)

Ich freue mich, mein aktuelles Projekt vorzustellen. Es ist eine ehrenamtliche Arbeit für den **[Malteser Hilfsdienst](https://www.malteser.de/standorte/frankfurt.html)**.


## 🛠️ Die Technologie
Ich entwickle eine Anwendung mit **Microsoft Power Apps** und **SharePoint**.
Das ist besonders spannend für mich, da es meine **erste praktische Erfahrung** mit der Power Platform und SharePoint als Datenbank ist. Ich lerne dabei sehr viel über Low-Code-Development.

## 🎯 Was macht das Projekt?
Der Integrationsdienst der Malteser bringt Menschen zusammen. Meine App hilft der Projektleitung dabei, diese Verbindungen digital zu verwalten.

Die Hauptfunktionen sind:
* **Verwaltung von Ehrenamtlichen:** Wer bietet Hilfe an? (Mit Status, Fähigkeiten, Kontaktdaten).
* **Verwaltung von Unterstützungssuchenden:** Wer braucht Hilfe? (Mit detailliertem Stammdatenblatt).
* **Tandem-Management:** Die wichtigste Funktion. Die App verknüpft einen Ehrenamtlichen mit einem Suchenden zu einem "Tandem" (Patenschaft).

## 🚧 Status des Projekts
Ich arbeite zurzeit noch aktiv an diesem Projekt. Die Entwicklung ist noch nicht abgeschlossen.
Auch dieses Dokument ist noch **in Arbeit (Work in Progress)**. Ich werde in Zukunft mehr Details, Screenshots und technische Einblicke hinzufügen, sobald das Projekt weiter fortgeschritten ist.

Bleibt dran!

---

# Dokumentation: Power Automate Flow – Teil 1: Konzept und Start

## Einführung: Die Brücke zwischen App und Excel

In diesem Projekt stehen wir vor einer technischen Herausforderung: Power Apps kann zwar Excel-Dateien hochladen, aber es kann den **Inhalt** der Excel-Datei (die einzelnen Zeilen und Spalten) nicht direkt lesen und in die SharePoint-Liste `Suchende` übertragen.

Um dieses Problem zu lösen, verwenden wir **Microsoft Power Automate**.
Der Flow fungiert hier als "Backend-Prozessor". Er arbeitet im Hintergrund, nimmt die Datei entgegen, zerlegt sie in ihre Bestandteile und speichert die Daten korrekt ab.

## Der Workflow-Überblick

Unser Prozess basiert auf einem Hilfs-Mechanismus, den wir "TempImport" nennen.
Anstatt zu versuchen, die Datei direkt zu verarbeiten, nutzt die App eine **Zwischenstation**:

1. **Der Upload:** Die Power App lädt die Excel-Datei in eine separate SharePoint-Liste namens `TempImport` hoch.
2. **Das Signal:** Das Hochladen in diese Liste ist das Startsignal für unseren Flow.
3. **Die Verarbeitung:** Der Flow holt sich die Datei, liest die Daten und schreibt sie in die eigentliche Datenbank (`Suchende`).
4. **Der Abschluss:** Der Flow löscht die Datei aus `TempImport`, um der App zu signalisieren, dass er fertig ist.

---

## Detaillierte Beschreibung der Flow-Schritte

### 1. Der Trigger (Der Auslöser)

Jeder Flow beginnt mit einem Trigger. Unser Trigger reagiert auf Veränderungen in der SharePoint-Liste.

* **Name im Flow:** `Trigger: File Uploaded`
* **Technischer Name:** *When an item is created* (SharePoint Connector)
* **Funktionsweise:**
Dieser Trigger überwacht permanent die Liste `TempImport`. Sobald die Power App hier einen neuen Eintrag erstellt (was passiert, wenn der Nutzer auf "Import Excel" klickt), wacht der Flow auf und startet den Prozess.
* **Wichtige Konfiguration:**
* **Site Address:** Hier ist die URL unserer `Tandem Verwaltung` hinterlegt.
* **List Name:** `TempImport`. Es ist entscheidend, dass hier *nicht* die Hauptliste gewählt wird, da wir sonst eine Endlosschleife riskieren könnten.



### 2. Informationen zum Anhang abrufen

Der Trigger liefert uns zwar die Information, *dass* ein Eintrag erstellt wurde (z.B. "Eintrag Nr. 55"), aber er gibt uns noch keinen direkten Zugriff auf die angehängte Datei selbst.

* **Name im Flow:** `Action: Get Attachment Info`
* **Technischer Name:** *Get attachments*
* **Funktionsweise:**
Wir fragen SharePoint: "Gib mir bitte alle Infos zu den Dateianhängen, die zu dem Eintrag gehören, der gerade erstellt wurde."
* **Die Verknüpfung (ID):**
Damit der Flow weiß, welchen Eintrag er prüfen soll, verwenden wir die **ID** aus dem Trigger-Schritt. Das ist die Verbindung zwischen Schritt 1 und Schritt 2.
* *Input:* `ID` (vom Trigger `When an item is created`)



---

# Dokumentation: Power Automate Flow – Teil 2: Die Hauptschleife & Dateivorbereitung

Nachdem der Flow durch den Trigger gestartet wurde und weiß, welche Anhänge vorhanden sind, beginnt nun die eigentliche Arbeit.

### 3. Die Hauptschleife (Main Loop)

Obwohl wir in unserer App normalerweise nur *eine* einzige Excel-Datei hochladen, ist SharePoint technisch darauf vorbereitet, dass ein Listeneintrag *mehrere* Anhänge haben könnte.
Deshalb gibt uns der vorherige Schritt ("Get Attachment Info") immer eine **Liste** von Anhängen zurück (auch wenn es eine Liste mit nur einem Eintrag ist).

* **Name im Flow:** `Main Loop: Process Each File`
* **Technischer Name:** *Apply to each*
* **Funktionsweise:**
Dieser Container-Baustein sagt: "Führe alle Schritte, die sich *in mir* befinden, für **jede** gefundene Datei aus."
* **Eingabe:**
Wir füttern diese Schleife mit dem `Body` aus dem vorherigen Schritt. Das ist die Liste der gefundenen Anhänge.

---

### 4. Den Datei-Inhalt herunterladen

In der Hauptschleife kümmern wir uns nun um die aktuelle Datei. Bisher kennen wir nur den *Namen* der Datei (Metadaten). Um sie zu verarbeiten, brauchen wir aber den *Inhalt* (die eigentlichen Daten/Bytes).

* **Name im Flow:** `Action: Download File Bytes`
* **Technischer Name:** *Get attachment content*
* **Warum ist das nötig?**
Der Flow kann nicht mit einer Datei arbeiten, die er nicht "in der Hand hält". Mit diesem Schritt lädt der Flow die Excel-Datei digital herunter und hält sie im Arbeitsspeicher fest.
* **Konfiguration:**
* **Id:** Die ID des Listeneintrags (aus dem Trigger).
* **File Identifier:** Die spezifische Kennung des Anhangs (aus dem aktuellen Element der Schleife).

---

### 5. Die temporäre Excel-Datei erstellen (Der technische Trick)

Hier kommen wir zu einem entscheidenden technischen Detail. Der Standard-Connector von Microsoft, um Excel-Tabellen zu lesen (*Excel Online Business*), hat eine Einschränkung: **Er kann keine Dateien direkt aus einem SharePoint-Listen-Anhang lesen.** Er kann nur Dateien lesen, die in einer Dokumentenbibliothek (z.B. OneDrive oder SharePoint Dokumente) liegen.

Deshalb müssen wir die heruntergeladene Datei kurzzeitig an einem Ort speichern, auf den der Excel-Leser zugreifen kann.

* **Name im Flow:** `Action: Create Temp Excel`
* **Technischer Name:** *Create file* (OneDrive for Business oder SharePoint Documents)
* **Funktionsweise:**
Wir nehmen den Inhalt (die Bytes), den wir gerade heruntergeladen haben, und erstellen damit eine physikalische Datei im Stammsystem (z.B. im Root-Folder von OneDrive oder in einem speziellen Ordner).
* **Der Dateiname:**
Wir verwenden oft die **ID** im Dateinamen (z.B. `temp_import_[ID].xlsx`), um sicherzustellen, dass der Name einzigartig ist und wir nicht versehentlich eine andere Datei überschreiben.
* **Das Ergebnis:**
Jetzt liegt eine "echte" Excel-Datei auf dem Laufwerk, die bereit ist, gelesen zu werden.

---

# Dokumentation: Power Automate Flow – Teil 3: Daten lesen und speichern

Jetzt, da wir eine temporäre Excel-Datei haben, können wir sie öffnen und die Daten Zeile für Zeile in unser System übertragen.

### 6. Excel-Zeilen auslesen

Der Flow greift nun auf die temporäre Datei zu, die wir im vorherigen Schritt erstellt haben.

* **Name im Flow:** `Action: Read Excel Rows`
* **Technischer Name:** *List rows present in a table* (Excel Online Business)
* **Funktionsweise:**
Diese Aktion öffnet die Excel-Datei und schaut in die Tabelle (meistens "Tabelle1" oder "Table1"). Sie holt alle darin enthaltenen Informationen heraus und gibt sie als eine Liste von Datenzeilen an den Flow zurück.
* **Wichtige Einstellung:**
Wir müssen dem Flow sagen, *welche* Datei er nehmen soll. Hier verwenden wir die **ID** der gerade erstellten temporären Datei (aus Schritt `Create Temp Excel`).

---

### 7. Die innere Schleife: Daten übertragen

Jetzt haben wir die Daten im Speicher des Flows (z. B. eine Liste mit 50 Personen). Um diese in SharePoint zu speichern, müssen wir jeden Datensatz einzeln anfassen.

* **Name im Flow:** `Inner Loop: Add Rows to List`
* **Technischer Name:** *Apply to each* (Verschachtelt in der Hauptschleife)
* **Funktionsweise:**
Diese Schleife läuft so oft durch, wie es Zeilen in der Excel-Datei gibt. Wenn die Excel-Datei 10 Zeilen hat, läuft diese Schleife 10-mal.

#### Die Aktion in der Schleife: Element erstellen

Innerhalb dieser Schleife passiert das eigentliche "Mapping" (die Zuordnung).

* **Aktion:** *Create item* (SharePoint)
* **Ziel:** Liste `Suchende`
* **Das Mapping (Die Zuordnung):**
Hier verbinden wir die Spalten aus Excel mit den Spalten in SharePoint.
* `Excel: Vorname`  ➡️ `SharePoint: Vorname`
* `Excel: Nachname` ➡️ `SharePoint: Nachname`
* `Excel: Email`    ➡️ `SharePoint: Email`


* **Besonderheit:** Da Excel manchmal Daten anders formatiert als SharePoint, nutzen wir hier oft kleine Formeln (Expressions), um sicherzustellen, dass z. B. ein leeres Feld nicht zum Absturz führt.

---

# Dokumentation: Power Automate Flow – Teil 4: Abschluss und Aufräumen

Nachdem die innere Schleife fertig ist, sind alle Daten sicher in der Liste `Suchende` gespeichert. Aber der Prozess ist noch nicht ganz zu Ende. Wir müssen noch "aufräumen" und der App Bescheid geben, dass wir fertig sind.

### 8. Den Trigger-Eintrag löschen (Das Fertig-Signal)

Dies ist einer der wichtigsten Schritte für die Benutzerfreundlichkeit (User Experience) unserer App.

* **Name im Flow:** `Delete item`
* **Technischer Name:** *Delete item* (SharePoint)
* **Ziel-Liste:** `TempImport` (Der "Briefkasten")
* **Funktionsweise:**
Wir löschen den Eintrag in der Hilfsliste `TempImport`, der den ganzen Prozess ausgelöst hat.
* **Warum tun wir das?**
1. **Speicherplatz & Sauberkeit:** Wir brauchen die temporäre Upload-Datei nicht mehr.
2. **Kommunikation mit der App:** Unsere Power App hat einen Timer, der ständig prüft: *"Ist die Datei in TempImport noch da?"*
* Solange die Datei da ist ➡️ Zeige Lade-Spinner ("Bitte warten...").
* Sobald die Datei weg ist ➡️ Verstecke Spinner, aktualisiere die Liste und zeige "Erfolg!" an.
Durch das Löschen geben wir also das Signal: **"Import erfolgreich abgeschlossen!"**

---

### Zusammenfassung des Flows

Wir haben einen vollautomatischen Prozess geschaffen, der:

1. Bemerkt, wenn eine Datei hochgeladen wird.
2. Die Datei technisch lesbar macht (Temp-File).
3. Alle Daten extrahiert und sauber in die Hauptdatenbank überträgt.
4. Sich selbst bereinigt und der App den Erfolg meldet.

Dies ermöglicht es den Mitarbeitern der Malteser, hunderte von Datensätzen in wenigen Sekunden zu importieren, ohne manuelle Arbeit.

---




`#PowerApps` `#SharePoint` `#Ehrenamt` `#Malteser` `#LearningByDoing` `#LowCode`

