# Keyword-Driven Testing – ein verständlicher Bericht aus dem imbus-Webinar

*Ein Überblick für alle, die – so wie ich – aus der Entwicklung kommen und mit dem Thema Testen bisher wenig zu tun hatten.*

## Worum ging es?

Heute habe ich an einem Webinar von **imbus** teilgenommen. Die beiden Referenten, Rolf Glunz und Christian Kupfer, haben ein Thema vorgestellt, das in der Welt der Softwarequalität sehr wichtig ist: **Keyword-Driven Testing** (kurz: KDT). Die Leitfrage des Webinars war:

> Ist Keyword-Driven Testing der „Königsweg" zum wirtschaftlichen Testen?

„Königsweg" bedeutet so viel wie der beste, eleganteste Weg zum Ziel. Am Ende gab es darauf eine klare, aber ehrliche Antwort – dazu später mehr.

Ich selbst komme aus der Programmierung und habe noch nie als Tester gearbeitet. Viele Begriffe waren für mich neu. Deshalb erkläre ich in diesem Bericht alles Schritt für Schritt und vergleiche es immer wieder mit Dingen, die man aus dem Programmieren kennt.

## Das Grundproblem: Warum braucht man so etwas überhaupt?

Bevor man eine Lösung versteht, sollte man das Problem kennen. Klassische **Testautomatisierung** (also Tests, die der Computer automatisch ausführt, statt dass ein Mensch klickt) hat in der Praxis oft mehrere Schwächen:

- Man braucht **Programmierkenntnisse**. Tests werden zum Beispiel in Python geschrieben, und das kann nicht jeder im Team.
- Die Tests sind aufwendig in der **Wartung** (also im Pflegen und Anpassen), besonders in großen Teams.
- Wenn man in so ein Test-Skript hineinschaut, ist oft **nicht klar, was der Test eigentlich prüfen soll**.
- Manuelle und automatisierte Tests werden meist **getrennt verwaltet**, in unterschiedlichen Werkzeugen. Dadurch geht viel mögliche Zusammenarbeit verloren.
- Test-Skripte sind selbst wie Software – sie **können Fehler enthalten** und müssen geprüft werden.

Christian Kupfer erzählte, dass sein Team vor etwa elf Jahren genau vor diesen Problemen stand. Die Tests waren ineffizient. Auf der Suche nach einer besseren Lösung sind sie beim Keyword-Driven Testing gelandet.

## Die Grundidee: Logik und Code trennen

Die zentrale Idee von KDT ist einfach und für Entwickler sofort vertraut: Man teilt einen automatisierten Test in zwei Teile auf.

1. **Die Testlogik** – das ist die Beschreibung, *was* der Test tut. Sie besteht aus lesbaren Bausteinen, den sogenannten **Keywords** (auf Deutsch: Schlüsselwörter).
2. **Der Testcode** – das ist der technische Teil, *wie* es im Hintergrund umgesetzt wird. Diesen Code braucht man erst, wenn man den Test wirklich automatisieren möchte.

Verbunden werden beide Teile über die Keywords.

> **Vergleich zum Programmieren:** Ein Keyword ist im Grunde wie eine **benannte Funktion**. Statt überall denselben Code zu wiederholen, gibt man einer Aktion einen Namen – zum Beispiel `Nutzer anmelden` – und ruft sie immer wieder auf. Rolf Glunz hat es im Webinar genau so gesagt: KDT ist eigentlich nur **prozedurales Denken**, angewendet auf das Testen.

Ein schöner Nebeneffekt dieser Trennung: Weil der Code nur ganz unten steckt, funktioniert die Idee auch **ohne Code** – also im manuellen Test. Aber dazu kommen wir gleich.

Die Ziele, die man damit erreichen will, sind:

- einfachere Automatisierung,
- hohe **Wiederverwendbarkeit** (man baut einen Baustein einmal und nutzt ihn oft),
- gute **Wartbarkeit** (man kann die Tests leicht pflegen),
- **Arbeitsteiligkeit** (mehrere Personen können gut zusammenarbeiten),
- **Klarheit** (jeder versteht, was der Test tut).

Interessant ist: Bis auf die Automatisierung sind das alles Ziele, die auch im manuellen Test wichtig sind.

## Was genau ist ein Keyword?

Ein Keyword ist ein wiederverwendbarer Baustein für einen Testschritt. Es besteht aus mehreren Teilen – auch hier hilft der Vergleich mit einer Funktion:

- eine kurze **Anweisung**, also der Name des Schritts (z. B. „gültiger Login"),
- **Parameter**, die man übergibt (z. B. der Benutzername „Tina Tester" und das Passwort) – genau wie die Argumente einer Funktion,
- die eigentliche **Funktion**, also der Teil, der die Aktion ausführt,
- eine **Beschreibung**, die in einer zentralen **Keyword-Bibliothek** gespeichert wird, damit jeder weiß, was der Baustein macht.

## Die drei Reifegrade der Testautomatisierung

Christian Kupfer hat drei Stufen gezeigt, wie „reif" eine Testautomatisierung sein kann:

**1. Capture & Replay (Aufzeichnen und Abspielen).**
Man zeichnet seine Klicks auf der Oberfläche auf, und das Werkzeug spielt sie später wieder ab. Das ist der einfachste Einstieg und liefert schnell ein Ergebnis. Der Nachteil: Sobald sich auch nur eine Kleinigkeit ändert, muss man alles neu aufzeichnen. *(Für Entwickler: ähnlich wie ein aufgenommenes Makro.)*

**2. Direkte Implementierung.**
Man schreibt die Tests direkt als Code, zum Beispiel in Python. Das ist mächtig, aber man kann schlecht zu mehreren am selben Skript arbeiten, und ohne Disziplin entsteht schnell ein **Wildwuchs**. Christian nannte ein gutes Beispiel: In einem Projekt hatten sie einmal **100 verschiedene Varianten**, um dieselbe Wartezeit (`wait`) zu programmieren. *(Für Entwickler: das ist Code ohne das DRY-Prinzip – „Don't Repeat Yourself".)*

**3. Keyword-Driven Testing.**
Erst diese Stufe bringt alle oben genannten Ziele unter einen Hut: Lesbarkeit, Wiederverwendbarkeit und Wartbarkeit zusammen. Und ein Keyword kann andere Keywords enthalten – Schritte lassen sich also ineinander verschachteln. Das führt direkt zum nächsten, sehr wichtigen Punkt.

## Das Schichtenmodell (Layer-Konzept)

Das Herzstück von KDT ist der Aufbau in **Schichten**. Diese Idee löst ein interessantes Problem: Verschiedene Leser brauchen unterschiedlich viele Details. Ein erfahrener Tester braucht nur einen groben Schritt. Der Computer dagegen braucht jede winzige Einzelanweisung. Die Lösung ist eine **mehrschichtige Beschreibung**, bei der jeder auf „seiner" Ebene liest:

| Schicht | Was steht dort? | Beispiel |
|---|---|---|
| **High-Level** | zusammengefasste, fachliche Schritte | „neues Konto anlegen" |
| **Intermediate-Level** | einzelne, technische Arbeitsschritte | „als Administrator anmelden", „Seite öffnen" |
| **Low-Level** | atomare (kleinste) Anweisungen | `fill text`, `click`, Vorname, E-Mail … |

Nur die unterste Schicht wird in einer echten Programmiersprache umgesetzt – und auch nur als einzelne, kleine Funktionen. Deshalb spricht man von einem **Low-Code-Ansatz**: Man muss kaum noch „drumherum" programmieren, sondern setzt die fertigen Bausteine zusammen.

> **Vergleich zum Programmieren:** Wer mit Komponenten arbeitet (zum Beispiel in Angular), kennt das Prinzip. Eine große Seiten-Komponente besteht aus kleineren Funktions-Komponenten, und die wiederum aus ganz kleinen Bausteinen. Genau dieselbe Verschachtelung gibt es bei den Keyword-Schichten.

Ein wichtiger Hinweis kam von Rolf Glunz: Man sollte sich **disziplinieren** und vorher einen Plan machen. Es gibt Projekte mit 50 Schichten – und dann findet niemand mehr etwas. Drei bis vier Schichten reichen normalerweise. Diese Schichten passen oft auch zu den Ebenen im Unternehmen: Die Geschäftsprozesse sind die **fachliche** Schicht, die Einzelaktionen die **technische**, und die direkte Interaktion mit dem System die **technologische** Schicht.

## Keyword-Driven Testing ist auch im manuellen Test stark

Das war für mich einer der überraschendsten Punkte. Viele glauben, Keywords lohnen sich nur, wenn man programmiert. Das ist ein **Trugschluss** (also ein Denkfehler).

Die High-Level- und Intermediate-Keywords sind nämlich schon von sich aus perfekte, gut verständliche **Arbeitsanweisungen** für einen menschlichen Tester. Das Schöne daran: Der manuelle Tester und der Automatisierer **sprechen dieselbe Sprache**. Der manuelle Tester schreibt einen Testfall aus den Bausteinen zusammen, und der Automatisierer legt später einfach den Code darunter. So verschwinden die typischen Missverständnisse an der **Schnittstelle** zwischen Fachbereich und Technik. Außerdem werden die manuellen Tests dadurch einheitlich: keine schwammigen, frei formulierten Schritte mehr, sondern klare, wiederverwendbare Bausteine.

## Best Practices aus der Praxis

Der Ansatz klingt einfach, aber – wie immer – steckt der Teufel im Detail. Christian Kupfer nannte drei goldene Regeln für den nachhaltigen Einsatz:

1. **Schichten-Disziplin.** Nicht zu viele Ebenen bauen. Drei bis vier reichen, sonst wird alles unwartbar.
2. **Namenskonventionen und klare Regeln (Governance).** Keywords müssen eindeutig benannt sein. Wenn einer „Nutzer einloggen" schreibt und ein anderer „Anmeldung User", entsteht **Redundanz** (unnötige Doppelung), und der Vorteil der Wiederverwendung ist weg. Es muss klar sein, wer Keywords anlegen und ändern darf.
3. **Regelmäßiges Refactoring.** Eine Keyword-Bibliothek altert wie normaler Code. Man muss regelmäßig aufräumen, alte Keywords entfernen oder zusammenfassen. Sonst bricht das System irgendwann unter seinem eigenen Gewicht zusammen.

## Werkzeuge und Frameworks

Zum Schluss noch ein kurzer Blick auf die Werkzeuge:

- **Robot Framework** – das bekannteste Open-Source-Werkzeug. Es basiert auf Python und ist genau nach dem Keyword-Prinzip aufgebaut. Man schreibt die Testfälle in Tabellenform.
- **imbus TestBench** – das eigene Werkzeug der Referenten. Es deckt den ganzen Ablauf ab: von der manuellen Beschreibung über die Keyword-Struktur bis zur Anbindung an Automatisierungs-Werkzeuge.
- **QF-Test** – unterstützt ebenfalls modulare, keyword-ähnliche Strukturen und macht Aufzeichnungen wartbarer.

Eine wichtige Aussage von Rolf Glunz: Am Ende kommt es **weniger auf das Werkzeug an** als darauf, dass man das Konzept dahinter konsequent durchzieht.

## Fazit: Königsweg oder Sackgasse?

Und damit zurück zur Leitfrage. Die Antwort von Christian Kupfer war: **„Ja, aber mit Bedingungen."**

KDT ist der Königsweg, wenn man auf **Skalierung** und langfristige Wartbarkeit setzt. Aber – und das ist ehrlich gesagt der Kern – die Wirtschaftlichkeit stellt sich **nicht am ersten Tag** ein. Am Anfang hat man sogar einen höheren Aufwand (den **Initialaufwand**), weil man die Bibliothek und die Schichten erst aufbauen muss.

Der eigentliche Gewinn kommt später, in der **Wartungsphase**: Wenn sich die Anwendung ändert, passt man **ein einziges Low-Level-Keyword an einer einzigen Stelle** an – und Hunderte Testfälle sind sofort wieder korrekt. Das ist die wahre Wirtschaftlichkeit.

Wer aber keinen **langen Atem** hat oder die Disziplin bei den Best Practices schleifen lässt, für den wird aus dem Königsweg schnell eine **Sackgasse**.

> **Vergleich zum Programmieren:** Das ist genau der bekannte DRY-Gedanke. Man investiert einmal mehr Zeit, um gemeinsame Logik an einer Stelle zu bündeln. Kommt dann eine Änderung, ändert man nur diese eine Stelle – nicht hundert.

## Was ich aus den Fragen am Ende gelernt habe

In der Fragerunde kamen zwei sehr praktische Punkte:

- **Wie aufwendig ist das Refactoring?** Es funktioniert meistens **nicht „nebenher"**, weil es im Alltag untergeht. Besser ist es, feste Zeiten einzuplanen – zum Beispiel ein bis zwei Stunden am Ende jedes Sprints. Macht man das laufend, ist der Aufwand klein. Ignoriert man es ein Jahr lang, braucht man danach eine ganze Woche zum Aufräumen.
- **Muss man ein bestehendes System komplett neu schreiben?** Nein. Man muss das **Rad nicht neu erfinden**. Vorhandene Skripte (zum Beispiel in Python) kann man wunderbar als Basis für die Low-Level-Keywords verwenden – man legt sozusagen eine logische Hülle darum. Der technische Kern bleibt; man definiert nur die fachliche Ebene darüber.

## Mein persönliches Fazit

Als Entwickler hatte ich beim Zuhören oft ein vertrautes Gefühl. Keyword-Driven Testing ist im Kern nichts anderes, als **gute Prinzipien aus der Softwareentwicklung auf das Testen zu übertragen**: Aufgaben trennen, Dinge wiederverwenden, sauber benennen und regelmäßig aufräumen. Vieles, was ich beim Programmieren ganz selbstverständlich mache, gilt hier eben für die Tests. Genau das macht das Thema für mich als Einsteiger gut zugänglich – und überraschend logisch.
