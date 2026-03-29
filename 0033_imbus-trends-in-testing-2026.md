# [Trends in Testing 2026](https://www.imbus.de/events/trends-in-testing/): Meine Insights von der [imbus](https://www.imbus.de/) Roadshow

## Inhaltsverzeichnis

* [Teil 1: Eröffnung und die Grundlagen der Agentic AI](#teil-1-eröffnung-und-die-grundlagen-der-agentic-ai)
* [Teil 2: Von Sprachmodellen zu intelligenten Agenten](#teil-2-von-sprachmodellen-zu-intelligenten-agenten)
* [Teil 3: Die Balance zwischen Mensch und Maschine (Human in the Loop)](#teil-3-die-balance-zwischen-mensch-und-maschine-human-in-the-loop)
* [Teil 4: Systemarchitektur und die Orchestrierung autonomer Agenten](#teil-4-systemarchitektur-und-die-orchestrierung-autonomer-agenten)
* [Teil 5: Multi-Agenten-Systeme und die Orchestrierung eines digitalen Teams](#teil-5-multi-agenten-systeme-und-die-orchestrierung-eines-digitalen-teams)
* [Teil 6: Showcases – Die Agenten in der Praxis und mein Entwickler-Blickwinkel](#teil-6-showcases--die-agenten-in-der-praxis-und-mein-entwickler-blickwinkel)
* [Teil 7: Testing AI – Wie wir die Künstliche Intelligenz selbst testen](#teil-7-testing-ai--wie-wir-die-künstliche-intelligenz-selbst-testen)
* [Teil 8: "Agentic Thinking" und die Neudefinition von Produktivität](#teil-8-agentic-thinking-und-die-neudefinition-von-produktivität)
* [Teil 9: Prompt Engineering und die Bedeutung von Fachkompetenz](#teil-9-prompt-engineering-und-die-bedeutung-von-fachkompetenz)
* [Teil 10: Der komplette Lebenszyklus – Agenten-Teams in der Praxis](#teil-10-der-komplette-lebenszyklus--agenten-teams-in-der-praxis)
* [Teil 11: Fazit und Ausblick – Bereit für die Zukunft (Conclusion & Outlook)](#teil-11-fazit-und-ausblick--bereit-für-die-zukunft-conclusion--outlook)

---

## Teil 1: Eröffnung und die Grundlagen der Agentic AI

Alles begann in einer ganz besonderen Atmosphäre: Im Filmpalast Hofheim. Zur Begrüßung durch die imbus-Manager [Frank Schmeißner](https://www.imbus.de/karriere/vorstellung-unserer-standorte/rhein-main#c19315) und [Rolf Glunz](https://www.imbus.de/events/trends-in-testing-2015/referenten#c3510) gab es eine riesige Kinoleinwand und perfekten Sound – die ideale Kulisse, um die beeindruckende Kraft der Künstlichen Intelligenz zu demonstrieren. Ein kurzer Rückblick auf die rasante Entwicklung der KI seit dem Start von ChatGPT im Jahr 2022 bildete den perfekten Einstieg in den Tag.

### Das Motto: "Licence to execute"

Das zentrale Thema der Konferenz war raffiniert gewählt: **"Licence to execute – Agentic AI kontrolliert einsetzen"**. 

Die Speaker zogen eine sehr treffende Parallele zu James Bond (007). So wie Bond die "Lizenz zum Töten" (Licence to Kill) hat, geben wir der Künstlichen Intelligenz heute die "Lizenz zum Ausführen". Das bedeutet: Wir erlauben der KI, Aufgaben völlig automatisch zu erledigen, behalten aber stets die Kontrolle.

### Die Evolution des Software-Testens

Laut imbus haben wir in der Qualitätssicherung drei große Entwicklungsstufen durchlaufen:
1. **Testen von KI:** Wir prüfen, ob die KI-Modelle selbst richtig funktionieren.
2. **Testen mit KI:** Wir nutzen die KI als reines Hilfswerkzeug (zum Beispiel als Copilot beim Programmieren).
3. **Testen mit Agenten (Heute):** Wir arbeiten mit autonomen "KI-Agenten", die Aufgaben selbstständig lösen.

### Die wichtigsten Fachbegriffe der neuen Ära

Um diese neue Welt zu verstehen, müssen wir einige Kernbegriffe kennen, die uns den ganzen Tag über begleitet haben:

* **Agentic AI (KI-Agenten):** Im Gegensatz zu normalen Chatbots sind diese Systeme autonom. Man gibt ihnen nur ein Ziel ("Goal") vor, und sie entscheiden selbst, "wie" sie dieses Ziel mit verschiedenen Werkzeugen erreichen.
* **Human in the Loop (HiTL):** Das bedeutet, dass der Mensch immer im Entscheidungsprozess bleibt. Je mächtiger die KI wird, desto wichtiger wird unsere Überwachung, um fatale Fehler (wie unautorisierte Geldüberweisungen) zu verhindern.
* **LLM (Large Language Model):** Der Motor dieser Systeme. Ein Modell verarbeitet Text in kleinen Einheiten, den sogenannten **Tokens**. Beeindruckend ist das **Context Window** (das Kurzzeitgedächtnis): Moderne Modelle wie Gemini fassen 1 bis 3 Millionen Tokens. Das reicht aus, um alle "Herr der Ringe"-Bücher auf einmal zu lesen und zu verstehen! Zudem sind sie **multimodal**, können also Text, Bild und Ton gleichzeitig verarbeiten.
* **RAG (Retrieval Augmented Generation):** Eine Methode, um die KI mit dem internen Wissen einer Firma (wie PDF-Dokumenten oder Jira-Datenbanken) zu verbinden. Das macht die Antworten genau und verhindert, dass die KI Dinge erfindet ("Halluzinationen").
* **MCP (Model Context Protocol):** Das wurde sehr passend als der "USB-Standard für KI" bezeichnet. Es ist ein offenes Protokoll, damit die KI sicher mit verschiedenen Werkzeugen und Daten kommunizieren kann.
* **Orchestrierung:** Die Kunst, mehrere KI-Agenten zu koordinieren, um gemeinsam ein komplexes Problem zu lösen.
* **Extended Thinking:** Neue Modelle (wie Claude 3.5) haben die Fähigkeit, erst in mehreren Schritten "nachzudenken" und zu planen, bevor sie eine Antwort geben.

### Werkzeuge und die Rolle der Entwickler

In der Praxis wurden bereits spannende Projekte und Tools erwähnt: Neben der *imbus TestBench* gab es Einblicke in *iS²* (ein Projekt, das Testfälle direkt aus Anforderungen extrahiert) und *OpenClaw*, ein stark wachsendes Open-Source-System für KI-Agenten auf GitHub.

Die wichtigste strategische Erkenntnis für uns Entwickler und Tester am Ende dieses ersten Teils: **Die KI ersetzt uns nicht.** Unsere Rolle verändert sich lediglich. Wir werden zu Managern und Aufsehern dieser intelligenten Systeme. Das ultimative Ziel der gesamten Technologie ist es, die Produktivität zu steigern und die Kosten in der Qualitätssicherung (QA) zu senken.

## Teil 2: Von Sprachmodellen zu intelligenten Agenten

Nach der Eröffnung ging es tief in den "Maschinenraum" der neuen Systeme. In diesem Teil der Konferenz drehte sich alles um das Gehirn der Agenten: die Large Language Models (LLMs) und wie wir effizient mit ihnen interagieren.

### 1. Ein tieferes Verständnis von LLMs
Zunächst wurde ein wichtiger Mythos aufgeräumt: LLMs "denken" nicht wie wir Menschen. Im Kern basieren sie auf Wahrscheinlichkeiten und versuchen lediglich, das nächste Wort (oder Token) logisch vorherzusagen. 

Besonders spannend fand ich das Konzept der **Emergenz**: Wenn diese Modelle eine bestimmte Größe überschreiten, entwickeln sie plötzlich Fähigkeiten, für die sie nie explizit programmiert wurden – wie zum Beispiel das Lösen komplexer Logikrätsel oder das Schreiben von Programmcode.

### 2. Das "Context Window" (Kontextfenster)
Das Kontextfenster ist quasi das Kurzzeitgedächtnis der KI. Je größer dieses Fenster ist (bei Google Gemini beispielsweise 1 bis 3 Millionen Tokens), desto mehr riesige Dokumente kann das Modell auf einmal "sehen" und analysieren. 

Aber es gibt einen Haken, den wir als Entwickler beachten müssen: Das Phänomen **"Lost in the Middle"**. Auch wenn das Kontextfenster riesig ist, neigen die Modelle dazu, Informationen, die sich in der Mitte eines sehr langen Textes befinden, zu vergessen oder zu übersehen. Sie fokussieren sich oft auf den Anfang und das Ende.

### 3. Chat vs. Agent: Der entscheidende Unterschied
Das war für mich einer der absoluten Schlüsselmomente der Konferenz. Warum ist "Agentic AI" ein so massiver Schritt nach vorne im Vergleich zu ChatGPT?

* **Chat (Das Interface):** Ist ein linearer Prozess. Du stellst eine Frage, das Modell antwortet. Fertig.
* **Agent (Der Ausführende):** Ein Agent verfügt über **Werkzeuge (Tools)**. Er kann das Internet durchsuchen, Code ausführen oder auf Datenbanken zugreifen.
* **Reasoning Loop (Die Denk-Schleife):** Der Agent arbeitet in Zyklen. Er *denkt* nach (Thought), führt eine *Aktion* aus (Action) und *beobachtet* das Ergebnis (Observation). Wenn das Ziel noch nicht erreicht ist, beginnt die Schleife von vorn. Das macht ihn autonom.

### 4. Das Vokabular für Entwickler und Tester
Um diese Systeme zu steuern, müssen wir ihre Sprache sprechen. Hier sind die wichtigsten Parameter:
* **Tokens vs. Wörter:** Ein Token ist nicht immer ein ganzes Wort, sondern oft nur ein Wortteil (1000 Tokens entsprechen grob 750 Wörtern).
* **Temperature (Temperatur):** Dieser Wert steuert die "Kreativität" der KI. In der Software-Entwicklung und im Testing stellen wir die Temperatur in der Regel auf `0`. Wir wollen keine kreativen, sondern *wiederholbare (deterministische)* Ergebnisse.
* **System Prompt:** Die Grundanweisung, die der KI ihre Rolle gibt (z.B. "Du bist ein Senior QA Engineer").
* **Few-Shot Prompting:** Wir geben der KI direkt im Prompt ein paar Beispiele mit, damit sie das gewünschte Antwortformat lernt.
* **Chain of Thought (CoT):** Eine Technik, bei der wir das Modell zwingen, "Schritt für Schritt" zu denken. Das erhöht die Genauigkeit bei logischen Problemen enorm.

### 5. Praxis-Einsatz im Software-Testing
Was bedeutet das nun konkret für unseren Alltag? Auf der Bühne wurden hervorragende Use Cases gezeigt:
* **Testdaten generieren:** LLMs eignen sich perfekt, um realistische, hochkomplexe Testdaten für Formulare und Datenbanken zu erzeugen.
* **Log-Analyse:** Man kann gigantische Mengen an Server-Logs in das Modell kippen, um versteckte Fehlermuster zu finden.
* **Von Sprache zu Code:** Man schreibt eine Test-Anforderung in normalem Deutsch oder Englisch, und die KI generiert daraus vollautomatisch fertige `Playwright`- oder `Selenium`-Skripte.

### Mein Fazit aus Teil 2: Vom "Schreiber" zum "Reviewer"
Für mich zeigt dieser Teil deutlich, wie sich unser Job verändert. Anstatt stundenlang Testskripte von Hand zu schreiben, wird unsere Hauptaufgabe in Zukunft das **Reviewen** und Validieren des KI-generierten Codes sein. Zudem wird die Optimierung unserer Prompts immer wichtiger – nicht nur, um die Genauigkeit zu erhöhen, sondern auch um *Tokens* und damit bares Geld (Cloud-Kosten) zu sparen.


## Teil 3: Die Balance zwischen Mensch und Maschine (Human in the Loop)

Nachdem wir uns den technischen "Maschinenraum" der KI-Agenten angesehen haben, widmete sich der dritte Teil der Konferenz der wohl wichtigsten Frage: Wie können wir der KI die "Licence to execute" (die Erlaubnis zum Handeln) geben, ohne die Kontrolle zu verlieren?

### 1. Aufgabenverteilung statt Ersetzung
Ein zentraler Punkt der Vorträge war, dass das Ziel von Agentic AI nicht darin besteht, uns Menschen zu ersetzen. Es geht vielmehr um eine **Umverteilung der Aufgaben** (Task Balance):
* **Die Aufgaben der Maschine:** Sie übernimmt das Durchsuchen gigantischer Datenmengen, das Schreiben von Entwürfen für Test-Skripte und die Ausführung von monotonen, repetitiven Szenarien.
* **Die Aufgaben des Menschen:** Wir übernehmen die strategischen Entscheidungen, bewerten komplexe geschäftliche Zusammenhänge (Business Context) und geben die finale Freigabe für sensible Bereiche.

### 2. Die Stufen der Autonomie
Die Zusammenarbeit mit der KI lässt sich in verschiedene Autonomie-Stufen unterteilen:
1. **KI als Assistent (Copilot):** Die KI schlägt vor, der Mensch führt aus.
2. **Überwachter Agent:** Die KI führt aus, aber der Mensch muss jedes Ergebnis manuell überprüfen und bestätigen.
3. **Autonome KI mit Leitplanken (Guardrails):** Die KI handelt völlig selbstständig innerhalb streng definierter technischer und ethischer Grenzen. Nur wenn eine Ausnahme auftritt, bittet sie den Menschen um Hilfe.

### 3. Vertrauen vs. Kontrolle
Ein KI-System ist nicht fehlerfrei. Wir müssen akzeptieren, dass LLMs gelegentlich "halluzinieren". Daher ist blindes Vertrauen gefährlich. 
* **Feedback Loops:** Als Tester müssen wir der KI kontinuierlich Feedback geben, damit das Modell lernt, die spezifischen Bugs unseres Projekts besser zu erkennen.
* **Strikte Verifikation:** Die eiserne Regel lautet: *Kein KI-generierter Output darf jemals ungeprüft in die Produktionsumgebung (Live-Umgebung) übernommen werden.* Es braucht immer einen menschlichen Reviewer oder ein zweites, unabhängiges Prüf-Tool.

### 4. Wichtiges Vokabular für QA-Architekten
In diesem Zusammenhang fielen einige Begriffe, die für unsere tägliche Arbeit essenziell werden:
* **Human in the Loop (HiTL):** Der Prozess, bei dem die KI an kritischen Punkten auf die Freigabe des Menschen warten muss.
* **Guardrails (Leitplanken):** Hardcodierte technische Grenzen, die verhindern, dass die KI beispielsweise versehentlich eine Datenbank löscht oder sensible Kundendaten preisgibt.
* **Delegation:** Die Kunst zu entscheiden, welche Tests man der KI überlassen *kann* und welche man ihr aus Sicherheitsgründen *nicht* überlassen *darf*.
* **Audit Trail:** Eine lückenlose Aufzeichnung aller Schritte, die der Agent gemacht hat. Wenn ein Fehler passiert, müssen wir genau nachvollziehen können, *warum* die KI so entschieden hat.
* **Supervisor Agent:** Ein spannender Ansatz, bei dem wir einen zweiten KI-Agenten bauen, der nur die Aufgabe hat, die Arbeit des ersten Agenten zu überwachen.

### 5. Praxis-Szenarien
Wie sieht das in der Praxis aus?
* **Requirements Review:** Die KI liest die Anforderungen und markiert logische Lücken. Der Mensch entscheidet dann, welche dieser Lücken wirklich kritisch sind und behoben werden müssen.
* **Test-Priorisierung:** Die KI berechnet das Risiko und schlägt vor, welche Tests zuerst laufen sollen. Der QA-Manager trifft die finale Entscheidung.

### Mein Fazit aus Teil 3: Vom Ausführenden zum Manager
Hier zeigt sich, warum ein starker technischer Hintergrund in der Softwareentwicklung aktuell so wertvoll ist. Wer den Code und die Architektur versteht, bringt genau die analytischen Fähigkeiten mit, die man braucht, um intelligente Systeme zu überwachen und die richtigen "Guardrails" zu setzen. 

Wir transformieren uns von "Test-Ausführenden" zu **"Test-Prozess-Managern"**. Am Ende hat die KI nämlich keine rechtliche Verantwortung – die Verantwortung für die Qualität des Produkts liegt immer bei uns Menschen.

## Teil 4: Systemarchitektur und die Orchestrierung autonomer Agenten

Dieser Teil der Konferenz war das absolute technische Herzstück. Hier ging es tief in die Architektur und die Frage, wie man autonome Agenten baut und orchestriert. Für mich als Entwickler war der Vergleich zwischen klassischer Softwareentwicklung und dem neuen KI-Ansatz besonders aufschlussreich.

### 1. Klassische Entwicklung vs. KI-basierte Entwicklung
Der Unterschied lässt sich auf zwei Begriffe herunterbrechen: Deterministisch vs. Probabilistisch.
* **Klassische Entwicklung (Deterministisch):** Hier programmieren wir die gesamte *Business Logic* bis ins kleinste Detail aus. Der Prozess ist starr; die Anwendung verlässt niemals den vordefinierten Pfad.
* **KI-Entwicklung (Probabilistisch):** Hier übergeben wir der KI die Entscheidungsgewalt. Der Prozess ist dynamisch. Die KI entscheidet selbstständig, *welche* Funktionen oder Werkzeuge sie *in welcher Reihenfolge* und *wie* aufruft, um ein Ziel zu erreichen.

### 2. Prompting ist Software-Architektur
Prompting ist nicht einfach nur Text. Technisch gesehen ist ein Prompt die Bereitstellung von Kontext-Informationen, die die Vorhersage der nächsten Tokens durch das LLM beeinflussen. Es gibt hierbei eine klare Hierarchie, ähnlich der klassischen Software-Architektur:
1. **Prompt Techniques:** Die übergreifende Strategie und Grobstruktur.
2. **Prompt Patterns:** Wiederverwendbare Lösungsmuster für ähnliche Probleme (vergleichbar mit Design Patterns in der objektorientierten Programmierung).
3. **Building Blocks (Bausteine):** Die kleinsten Einheiten wie Wörter, die zugewiesene Rolle (Persona), das gewünschte Output-Format und die reinen Kontext-Informationen.

### 3. Die verschiedenen Rollen eines Prompts
Um zu verstehen, wie ein Agent unter der Haube funktioniert, muss man die vier technischen Rollen eines Prompts kennen:
* **System Prompt:** Das sind die Meta-Anweisungen, die das Grundverhalten der KI definieren (z. B. "Du bist ein Senior Software Tester"). Wir Entwickler schreiben diesen Prompt; der Endnutzer sieht ihn in der Regel nicht.
* **User Prompt:** Die eigentliche Anfrage oder Aufgabe, die der menschliche Nutzer eingibt.
* **Assistant Prompt:** Die Antwort, die das KI-Modell generiert und zurückgibt.
* **Tool Prompt:** Das ist extrem wichtig für Agenten! Das sind die Ergebnisse aus aufgerufenen Werkzeugen (z. B. ein Datenbank-Ergebnis), die an das Modell zurückgespielt werden, damit es weiß, was in der realen Welt passiert ist.

### 4. Fortgeschrittene technische Konzepte für Entwickler
Einige weitere Fachbegriffe, die in diesem Zusammenhang unverzichtbar sind:
* **Few-Shot Prompting:** Wir geben der KI direkt im Prompt ein paar Lösungsbeispiele mit, um Format und Tonalität exakt vorzugeben.
* **Chain of Thought (CoT):** Die "Gedankenkette". Das Modell wird gezwungen, große Probleme in kleine Teilprobleme zu zerlegen und diese Schritt für Schritt zu lösen.
* **Lost in the Middle:** Ein bekanntes Problem bei LLMs. Selbst bei einem riesigen Context Window ignoriert das Modell oft Informationen, die genau in der *Mitte* eines langen Textes stehen, und fokussiert sich auf Anfang und Ende.
* **Prompt Injection:** Ein massives Sicherheitsrisiko (Security Threat)! Hierbei versuchen Angreifer durch irreführende Prompts, die KI dazu zu bringen, ihre System-Regeln zu ignorieren und sensible Daten (wie Passwörter oder SSH-Keys) preiszugeben. Das kann auch *indirekt* passieren (Indirect Prompt Injection), beispielsweise wenn der Agent den präparierten HTML-Code einer fremden Website liest.

### 5. Die moderne Lösung: Sub-Agenten (Modularisierung)
Wie lösen wir diese Komplexität? Die Antwort von imbus war klar: Anstatt einem einzigen Agenten hundert Werkzeuge zu geben (was das Modell verwirrt und das Context Window sprengt), nutzen wir **spezialisierte Sub-Agenten**. 
Die Vorteile dieser Modularisierung liegen auf der Hand: Eine deutlich höhere Testbarkeit, besseres Management von Sicherheitsrisiken (Guardrails) und effizientere Ressourcennutzung.

### Fun Fact aus dem Q&A: Höflichkeit zahlt sich aus
Auf eine Frage aus dem Publikum gab es eine interessante Antwort: Soll man zur KI "Bitte" und "Danke" sagen? Laut einem Paper von Meta aus dem Jahr 2023 erhöht eine höfliche Sprache im Prompt tatsächlich die Qualität der Antworten. Der Grund? In den gigantischen Trainingsdaten der Modelle korrelieren höfliche Formulierungen meist mit inhaltlich hochwertigeren Texten.

## Teil 5: Multi-Agenten-Systeme und die Orchestrierung eines digitalen Teams

Nachdem wir in Teil 4 die grundlegende Architektur eines Agenten betrachtet haben, gehen wir nun einen Schritt weiter: Was passiert, wenn wir ein großes Projekt – wie den kompletten Test einer Applikation – umsetzen wollen? Die Antwort lautet: Wir bauen ein digitales Team aus mehreren KI-Agenten. 

Dieser Teil der Konferenz war für mich als Entwickler besonders greifbar, da die Konzepte stark an moderne Software-Architektur und Microservices erinnern.

### 1. Das Konzept der Orchestrierung
Orchestrierung bedeutet hier das Management des Workflows zwischen verschiedenen Modellen. Dabei gibt es zwei Hauptansätze:
* **Chaining (Verkettung):** Der Output des einen Agenten dient direkt als Input für den nächsten (wie bei einer Pipeline).
* **Routing (Bedingte Weiterleitung):** Ein zentraler "Router"-Agent nimmt die Anfrage entgegen und entscheidet anhand des Inhalts, an welchen spezialisierten Agenten er die Aufgabe weiterleitet.

### 2. Strukturen der Zusammenarbeit
Wie in menschlichen Teams gibt es auch bei Agenten unterschiedliche Organisationsformen:
* **Hierarchisch:** Ein übergeordneter "Manager-Agent" teilt die Aufgaben auf seine "Sub-Agenten" auf, sammelt deren Ergebnisse ein und führt das finale Review durch.
* **Kollaborativ (Peer-to-Peer):** Die Agenten kommunizieren direkt miteinander und tauschen völlig autonom Informationen aus, um ein Problem gemeinsam zu lösen.

### 3. Agent Memory: Das Gedächtnis der KI
Eine der größten Herausforderungen bei langen Prozessen ist der Erhalt von Informationen. Hierfür nutzt man zwei Arten von Gedächtnis:
* **Short-term Memory (Kurzzeitgedächtnis):** Das ist das "Context Window", in dem die Informationen für die *aktuelle* Operation liegen.
* **Long-term Memory (Langzeitgedächtnis):** Hier kommen **Vektor-Datenbanken (Vector DBs)** ins Spiel. Sie speichern das Wissen und die vergangenen Erfahrungen des Agenten. Wir können diese Funktion hervorragend nutzen, um die KI aus alten, bereits behobenen Bugs unseres Projekts lernen zu lassen!

### 4. Neues Vokabular für Entwickler
Einige weitere Begriffe, die den Umgang mit Multi-Agenten-Systemen prägen:
* **Agent Handoff:** Die reibungslose Übergabe einer Aufgabe samt dem kompletten Kontext von einem Agenten an den anderen.
* **State Management:** Die Verwaltung des aktuellen Systemzustands in jeder Phase der Testausführung.
* **Planning Agent:** Ein spezialisierter Agent, der nur dafür da ist, eine Roadmap zur Problemlösung zu entwerfen.
* **Execution Agent:** Das "Arbeitstier", das ausschließlich dafür zuständig ist, Code oder Werkzeuge auszuführen.

### 5. Ein konkretes Praxis-Beispiel für die Testautomatisierung
Wie sieht so ein digitales Team in Aktion aus? Auf der Konferenz wurde folgendes Szenario skizziert:
1. Der **Analyst-Agent** liest die Anforderungen (Requirements).
2. Der **Planner-Agent** übernimmt und entwirft daraus logische Test-Szenarien.
3. Der **Coder-Agent** schreibt das eigentliche Test-Skript (z.B. in *Playwright*).
4. Der **Reviewer-Agent** prüft den Code abschließend auf Einhaltung der unternehmensinternen Standards.

### Mein Fazit aus Teil 5: Separation of Concerns für KI
Dieser Ansatz bringt zwei massive strategische Vorteile mit sich. Erstens: **Kosten und Effizienz**. Es ist meistens deutlich günstiger und präziser, mehrere kleine, spezialisierte Modelle zu nutzen, als ein einziges, riesiges Modell für alles einzusetzen. 
Zweitens: **Fehlerreduktion**. Durch die strikte Aufgabentrennung (in der Entwicklung kennen wir das als *Separation of Concerns*) wird die Wahrscheinlichkeit von KI-Halluzinationen drastisch gesenkt.

## Teil 6: Showcases – Die Agenten in der Praxis und mein Entwickler-Blickwinkel

Nach viel Theorie ging es in diesem Teil der Konferenz endlich in die Praxis. Die Speaker zeigten echte "Showcases" – also reale Beispiele, wie KI-Agenten bereits heute bei imbus und deren Kunden eingesetzt werden. Für mich als Entwickler war das der spannendste Teil, denn hier schließt sich der Kreis zu meiner täglichen Arbeit.

### 1. Von der Anforderung zum Testfall (Requirements to Test Cases)
Eines der beeindruckendsten Beispiele war die Automatisierung der Test-Design-Phase:
* **Der Input:** Eine einfache User Story in natürlicher Sprache (zum Beispiel auf Deutsch).
* **Die Arbeit des Agenten:** Der KI-Agent analysiert den Text, deckt logische Lücken und Unklarheiten auf und schlägt direkt eine Liste mit sinnvollen Testfällen vor.
* **Der Vorteil:** Die Erstellung von Testfällen kostet oft viel Zeit. Dieser Prozess wird hier massiv beschleunigt.

### 2. Intelligentes exploratives Testen
Ein Agent kann sich durch eine Applikation bewegen wie ein echter Mensch. Man gibt ihm ein Ziel, zum Beispiel: *"Kaufe ein Produkt im Shop."* Der Agent betrachtet die Benutzeroberfläche (UI), identifiziert die Buttons und probiert verschiedene Wege aus, um das Ziel zu erreichen.

Der massive Unterschied zu klassischen Tools? **Self-Healing (Selbstheilung)**. Wenn ich in meinem Code einen Button verschiebe oder eine ID ändere, bricht ein altes Automatisierungs-Skript einfach ab. Der Agent hingegen sucht nicht nach starren IDs, sondern versteht die UI semantisch. Er findet den Button trotzdem und repariert das Skript im Hintergrund.

### 3. Live-Testing und direktes Feedback
Der Agent führt den Test nicht nur aus, er übernimmt auch das Reporting. Tritt ein Fehler auf, macht die KI automatisch einen Screenshot, analysiert die Server-Logs und erstellt einen kompletten, detaillierten Bug-Report direkt in Systemen wie Jira.

### 4. Automatisierte Security-Tests mit iSecTest
Ein weiteres, extrem spannendes Praxisbeispiel aus dem imbus-Portfolio ist [**iSecTest**](https://www.imbus.de/security-test/isec-test). Als Web-Entwickler weiß ich, wie komplex und zeitaufwendig Vulnerability Assessments (Schwachstellenanalysen) und Penetrationstests sein können. 
Dieses Tool nutzt KI, um mit nur wenigen Klicks gezielte Security-Testfälle für Webanwendungen zu generieren. Die Künstliche Intelligenz deckt dabei selbstständig potenzielle Sicherheitslücken auf und hilft dabei, Systeme gegen komplexe Cyberangriffe abzusichern. Das beschleunigt den Prozess des Security-Testings enorm und zeigt eindrucksvoll, wie Agenten uns dabei helfen, unsere Web-Apps sicher zu machen.

### 5. Wichtiges Vokabular für die Praxis
Um diese Showcases zu verstehen, wurden folgende Begriffe geprägt:
* **Test Oracle (Test-Orakel):** Das Konzept, bei dem die KI als "Schiedsrichter" entscheidet, ob ein Testergebnis "Pass" (bestanden) oder "Fail" (fehlgeschlagen) ist.
* **Semantic Understanding:** Die KI versteht die *Bedeutung* eines Feldes (z. B. "Das ist ein Passwort-Feld") und sucht nicht nur nach technischen Bezeichnern in HTML.
* **Validation Agent:** Ein spezieller Agent, dessen einzige Aufgabe es ist, die Ergebnisse der anderen Agenten zu überprüfen.

### Meine Perspektive: Die perfekte Symbiose mit .NET und Angular
Genau bei diesen Praxisbeispielen sehe ich den enormen Wert meiner technischen Erfahrung. Agentic AI ist keine isolierte Insel. Die wahre Stärke entsteht durch **Integration**. 

Moderne KI-Agenten (und KI-Lösungen wie iSecTest) können den **Angular**-Code direkt analysieren, um Komponenten besser zu verstehen, und gleichzeitig intelligente Integrationstests für Backend-Services in **.NET** schreiben. Diese Tools ersetzen uns nicht. Sie nehmen uns die stundenlange, repetitive Fleißarbeit ab, damit wir den Test-Prozess einer gesamten Web-Applikation um das Zehnfache beschleunigen können.

## Teil 7: Testing AI – Wie wir die Künstliche Intelligenz selbst testen

Bisher haben wir darüber gesprochen, wie wir KI nutzen können, um unsere Software zu testen ("Testing with AI"). In diesem Teil der Konferenz drehte sich der Spieß jedoch um. Die entscheidende Frage lautete: **Wie stellen wir sicher, dass die KI selbst richtig funktioniert?**

### 1. Die große Herausforderung: Non-Determinismus
Als .NET-Entwickler bin ich es gewohnt, dass Code deterministisch ist. Wenn ich in einem `xUnit`-Test denselben Input gebe, erwarte ich jedes Mal exakt denselben Output. Bei KI-Modellen ist das anders: Bei gleichem Input können wir unterschiedliche Antworten erhalten. 

Die Lösung? Ein einfacher "Wort-für-Wort"-Vergleich funktioniert in der traditionellen Automatisierung nicht mehr. Wir müssen auf **semantische Vergleiche** (Bedeutungsvergleiche) und statistische Metriken umsteigen.

### 2. Metriken zur KI-Bewertung (Evaluation Metrics)
Um die Qualität der Antworten eines Large Language Models (LLM) objektiv zu messen, nutzen Experten spezielle Metriken:
* **Faithfulness (Treue):** Basiert die Antwort der KI wirklich *nur* auf unseren bereitgestellten Dokumenten, oder hat das Modell sich Dinge ausgedacht?
* **Relevance (Relevanz):** Beantwortet die KI wirklich die gestellte Frage oder schweift sie ab?
* **Answer Correctness (Richtigkeit):** Sind die genannten Fakten in der Antwort objektiv korrekt?

### 3. KI als Schiedsrichter (LLM-as-a-Judge)
Ein sehr mächtiges Konzept, das vorgestellt wurde, ist **LLM-as-a-Judge**. Wenn wir hunderte Agenten-Antworten testen müssen, können wir das nicht manuell tun. Stattdessen nutzen wir ein stärkeres Modell (zum Beispiel GPT-4o) als "Schiedsrichter". Dieser Schiedsrichter bekommt ein klares Regelwerk (Rubric) und bewertet damit automatisch die Qualität der Outputs unseres schwächeren Test-Agenten.

### 4. Neues Vokabular für den QA-Alltag
In der Welt des "Testing AI" gibt es neue Begriffe, die wir kennen müssen:
* **Hallucination Rate:** Die Fehlerquote der KI – also der Prozentsatz der Antworten, in denen die KI falsche Informationen liefert.
* **Ground Truth:** Unsere "Wahrheit". Das ist ein von Menschen verifizierter Datensatz, den wir als Referenz nutzen, um die KI-Ausgaben damit zu vergleichen.
* **Prompt Robustness:** Wie stabil ist der Output? Ändert sich die Qualität der Antwort massiv, wenn wir nur ein Wort oder den Tonfall im Prompt leicht anpassen?
* **Adversarial Testing:** Das bewusste "Hacken" oder Stören der KI (Penetration Testing). Wir versuchen absichtlich, die KI auszutricksen, damit sie gefährliche oder falsche Dinge ausgibt, um ihre Sicherheitslücken zu finden.

### 5. Werkzeuge für automatisierte KI-Tests
Auf der Konferenz wurden auch praktische Tools genannt, die uns Entwicklern helfen:
* **Giskard:** Ein starkes Framework, um Schwachstellen und Bias (Voreingenommenheit) in KI-Modellen zu identifizieren.
* **DeepEval:** Ein großartiges Tool, um echte Unit-Tests für LLM-Outputs zu schreiben.

### Mein Fazit: Der Paradigmenwechsel zum Evaluation Engineer
In der Welt der Künstlichen Intelligenz hat sich die Definition eines "Bugs" komplett verändert. Ein Bug ist nicht mehr nur eine falsche Codezeile. Ein Bug kann heute eine "sprachliche Voreingenommenheit" (Bias) oder eine "logische Halluzination" sein.

Das zeigt mir ganz deutlich: Die Notwendigkeit für uns Tester verschwindet nicht, sie wird nur viel technischer und spezialisierter. Mit meiner Erfahrung im Schreiben von Unit-Tests kann ich diese Konzepte nahtlos übernehmen. Wir klicken in Zukunft nicht mehr durch Menüs, wir werden zu **Evaluation Engineers**, die automatisierte Bewertungssysteme für KIs programmieren.

## Teil 8: "Agentic Thinking" und die Neudefinition von Produktivität

In diesem Teil der Konferenz ging es um das richtige "Mindset" (die Denkweise). Wenn wir mit KI-Agenten arbeiten, müssen wir unsere klassischen Prozesse in der Softwareentwicklung anpassen.

### 1. Was ist "Agentic Thinking"?
Es bedeutet den Wechsel von "Befehlen" zu "Zielen". 
* **Der klassische Weg:** Wir sagen der Maschine genau, was sie tun soll: *"Klicke auf diesen Button, dann prüfe diesen Text."*
* **Der Agentic-Weg:** Wir geben der KI nur noch das Ziel vor: *"Teste den Kaufprozess und prüfe, ob der Rabatt richtig berechnet wird."* Der Agent findet den Weg dorthin selbstständig.

### 2. Die perfekte Balance zwischen Mensch und Maschine
Wann sind wir wirklich produktiv? Die Speaker machten klar:
* Die **Maschine** ist extrem schnell bei sich wiederholenden Aufgaben und der Erstellung von Inhalten.
* Der **Mensch** ist unersetzlich, wenn es um ethische Entscheidungen und das Verständnis für das Geschäft (Business Context) geht.

Echte Produktivität entsteht, wenn die KI die monotonen Aufgaben übernimmt. So haben wir Menschen mehr Zeit für kreative und strategische Arbeit.

### 3. KI im Requirements Engineering (Anforderungsanalyse)
Viele Bugs in der Software entstehen schon ganz am Anfang, weil die Anforderungen unvollständig sind. KI-Agenten können diese Anforderungen analysieren und auf logische Fehler oder fehlende Testbarkeit prüfen. Sie schlagen sogar automatisch sogenannte "Edge Cases" (Grenzfälle) vor. So bringen wir die Qualität schon an den Ursprung des Projekts, nicht erst ans Ende!

### 4. Wichtiges Vokabular für Entwickler
* **Traceability (Rückverfolgbarkeit):** Wir müssen immer nachvollziehen können, welchen Weg die KI von der ursprünglichen Anforderung bis zum Testergebnis gegangen ist.
* **Trust Calibration (Vertrauens-Kalibrierung):** Die Kunst zu wissen, in welchen Situationen man der KI vertrauen kann und wann man ihre Ergebnisse strikt überprüfen muss.
* **Productivity Paradox (Produktivitäts-Paradoxon):** Eine wichtige Warnung! KI bedeutet nicht automatisch bessere Qualität. Wenn wir die KI nicht richtig überwachen, produzieren wir vielleicht nur "falschen Code", aber dafür in Rekordgeschwindigkeit.
* **Orchestration Logic:** Die Logik, wie wir verschiedene KI-Agenten anordnen, um ein Problem zu lösen.

### Mein Weg vom Entwickler zum "Quality Architect"
Dieser Vortrag hat mir gezeigt, dass Prompt-Engineering mehr ist als nur eine technische Fähigkeit – es ist ein komplett neues Paradigma. 

In Zukunft sind wir nicht mehr nur "Tester", die fertige Skripte ausführen. Wir werden zu **Architekten der Qualitätssicherung**. Wir entwerfen, steuern und orchestrieren Agenten-Systeme. Genau hier ist mein Hintergrund in der Softwareentwicklung (mit **.NET** und **Angular**) entscheidend. Um die Architektur und die Logik dieser autonomen Agenten zu verstehen und sicher zu steuern, braucht man tiefgreifendes Programmierwissen.

## Teil 9: Prompt Engineering und die Bedeutung von Fachkompetenz

In diesem Abschnitt ging es um eine sehr wichtige Erkenntnis: Einen guten Prompt zu schreiben, ist nicht nur eine Frage der Sprache. Es ist eine echte Ingenieurskunst, die extrem stark vom Fachwissen (Domain Knowledge) des Testers abhängt.

### 1. Der Prompt als Arbeitsauftrag (Work Order)
Wir müssen einen Prompt wie einen detaillierten Arbeitsauftrag für einen hochspezialisierten Mitarbeiter betrachten. 
* Wenn unsere Anweisungen zu allgemein sind, ist auch das Ergebnis der KI nur oberflächlich.
* Ein erfahrener Tester weiß genau, welche technischen Details (wie Randbedingungen oder Sicherheitsvorgaben) er in den Prompt schreiben muss, damit der Agent wirklich nützliche Test-Szenarien entwickelt.

### 2. Wissenstransfer (Context Injection)
Die KI hat ein riesiges Allgemeinwissen, aber sie kennt unsere spezifische Software-Logik nicht. Es ist unsere Aufgabe als Tester, den nötigen "Kontext" zu liefern. Wir müssen dem Agenten erklären, worauf es in unserem Projekt ankommt – zum Beispiel, ob die schnelle Antwortzeit oder die absolute Datensicherheit im Fokus steht.

### 3. Fortgeschrittene Prompt-Muster (Patterns)
Für uns Tester gibt es spezielle Muster, die die Arbeit enorm verbessern:
* **Persona Pattern:** Wir geben der KI eine genaue Rolle. (Zum Beispiel: *"Du bist ein White-Hat-Hacker und suchst nach Sicherheitslücken."*)
* **Chain of Verification (CoV):** Wir weisen die KI an, ihre eigene Antwort nach der Erstellung noch einmal kritisch zu überprüfen und mögliche Fehler selbst zu korrigieren.
* **Few-Shot Learning:** Wir geben der KI direkt im Prompt echte Beispiele aus alten Testfällen, damit sie die Struktur und unsere Erwartungen genau versteht.

### 4. Wichtiges Vokabular
* **Fachkompetenz:** Das tiefe Wissen über Software und Prozesse, das nötig ist, um die KI richtig zu steuern.
* **Prompt Refinement:** Der Prozess, bei dem wir unseren Prompt immer wieder anpassen und verbessern, bis das Ergebnis perfekt ist.
* **Hallucination Mitigation:** Techniken im Prompt, die verhindern sollen, dass die KI sich Dinge ausdenkt (indem wir ihr enge Grenzen setzen).
* **Instruction Following:** Die Fähigkeit des Modells, auch sehr komplexe Anweisungen mit vielen Schritten genau zu befolgen.

### 5. Praxis-Tipp: Negative Testdaten generieren
Ein tolles Praxisbeispiel war die Generierung von "Negativen Testdaten". Mit unserem Fachwissen können wir die KI anweisen, gezielt Daten zu erzeugen, die das System zum Absturz bringen sollen (z. B. Sonderzeichen in einem Zahlenfeld oder Text in einem Nummernfeld). 

### Mein Fazit: Warum Entwickler die besten Prompter sind
Auf der Konferenz fiel ein schöner Vergleich: *Eine KI ohne einen fachkundigen Tester ist wie ein Chirurg ohne Skalpell.* Genau hier wird mein Hintergrund als Softwareentwickler zum größten Vorteil. Weil ich die Datenstrukturen in **.NET** und das Verhalten von **Angular**-Komponenten kenne, kann ich Prompts schreiben, die die KI viel tiefer in den Code und die Architektur führen als jemand ohne Programmiererfahrung. Das kontinuierliche Lernen dieser Prompting-Techniken ist heute genauso wichtig wie früher das Lernen von Automatisierungs-Tools wie Selenium.

## Teil 10: Der komplette Lebenszyklus – Agenten-Teams in der Praxis

In diesem Showcase wurde demonstriert, wie ein komplettes Multi-Agenten-System zusammenarbeitet, um eine komplexe Testaufgabe von Anfang bis Ende durchzuführen. Es war faszinierend zu sehen, wie aus Theorie echte Praxis wird.

### 1. Das Praxis-Szenario: Testing eines neuen Features
Das Ziel war es, ein neu entwickeltes Feature in einer Web-Applikation zu testen. Der Ablauf wurde auf drei spezialisierte Agenten aufgeteilt:
* **Analyst Agent:** Er liest den neuen Code und die Dokumentation, um genau zu verstehen, was geändert wurde.
* **Designer Agent:** Basierend auf diesen Änderungen entwirft er neue Testfälle (Test Cases), die alle kritischen Pfade abdecken.
* **Automation Agent:** Er nimmt diese Entwürfe und programmiert daraus fertige, ausführbare Skripte (zum Beispiel mit dem Framework *Playwright*).

### 2. Der Traum jedes Testers: "Self-Healing Tests"
Jeder, der schon einmal Tests automatisiert hat, kennt den Schmerz von "Brittle Tests" (zerbrechlichen Tests). Wenn ein Entwickler die ID eines Buttons ändert, bricht der Test sofort ab. 

Hier zeigte die KI ihre wahre Stärke: Wenn sich die Oberfläche (UI) ändert, bricht der Agentic-Workflow nicht ab. Der Agent sucht visuell und semantisch nach dem Button, findet ihn trotzdem und repariert das Test-Skript einfach "on the fly" (im laufenden Betrieb).

### 3. Intelligente Analyse und Bug-Reporting
Wenn ein Test fehlschlägt, gibt uns die KI nicht nur ein rotes "Fail" zurück. Sie geht viel tiefer:
* **Root Cause Analysis (Ursachenanalyse):** Der Agent durchsucht automatisch die Konsolen-Logs des Browsers und die API-Antworten, um den genauen Fehlergrund zu finden.
* **Automatisches Jira-Ticket:** Er öffnet selbstständig ein Fehler-Ticket, hängt einen Screenshot des Fehlers an und schreibt die "Steps to Reproduce" (Schritte zur Reproduktion) in klarem, menschlichem Text.

### 4. Wichtiges Vokabular für moderne Teams
* **Agentic Workflow:** Ein Arbeitsablauf, bei dem KI-Agenten Aufgaben autonom untereinander aufteilen und ausführen.
* **Cross-Agent Communication:** Die Fähigkeit der Agenten, Informationen auszutauschen (z. B. wenn der Test-Agent eine Fehlermeldung zur Auswertung an den Analyst-Agenten schickt).
* **Visual Regression Testing:** Die KI erkennt ungewollte grafische Veränderungen auf der Benutzeroberfläche.
* **Dynamic Test Execution:** Tests werden nicht mehr nach einer starren Liste ausgeführt, sondern passen sich dynamisch dem aktuellen Zustand des Systems an.

### Mein Entwickler-Blickwinkel: Das Ende der Wartungshölle
Dieser Showcase hat bewiesen, dass Agentic AI keine Zukunftsmusik mehr ist. Für Projekte mit **Angular** und **.NET** ist das ein gigantischer Sprung. Ein intelligenter Agent kann meine Angular-Komponenten analysieren, ihre Struktur verstehen und smarte Integrationstests für meine .NET-Services schreiben. 

Wir Entwickler und Tester können uns endlich wieder auf die **Test-Strategie** konzentrieren. Die nervige operative Arbeit und die ständige Wartung von kaputten Skripten übernimmt das digitale Agenten-Team.


## Teil 11: Fazit und Ausblick – Bereit für die Zukunft (Conclusion & Outlook)

Jede gute Konferenz braucht einen starken Abschluss. In diesem letzten Teil fassten die Speaker (darunter Niels und sein Team) die wichtigsten Erkenntnisse zusammen und zeigten uns die Roadmap für die kommenden Jahre auf. Es wurde klar: Wer diese Welle der "Agentic AI" jetzt nutzt, hat in der Zukunft einen massiven Vorteil.

### 1. Die Kernkonzepte des Tages auf einen Blick
Die beiden wichtigsten Botschaften, die ich aus Hofheim mitnehme, sind:
* **Von der Automatisierung zur Autonomie:** Wir verlassen die Ära der starren, fehleranfälligen Test-Skripte. Wir betreten das Zeitalter der autonomen Agenten, die wir nicht mehr programmieren, sondern *steuern*.
* **Qualität an der Quelle (Quality at Source):** KI ermöglicht es uns, Qualität nicht erst am Ende des Entwicklungszyklus zu testen, sondern bereits ganz am Anfang – beim Schreiben der Anforderungen.

### 2. Eine Roadmap für Tester und Organisationen
Wie fangen wir jetzt an? Die Experten gaben klare Handlungsempfehlungen:
* **Interne Labore (Sandboxes):** Unternehmen müssen sichere, isolierte Testumgebungen schaffen. Wir Entwickler und Tester brauchen einen Ort, an dem wir mit KI-Agenten experimentieren können, ohne Angst vor Datenlecks (Data Leaks) haben zu müssen.
* **Fokus auf Soft Skills:** Die wichtigsten technischen Fähigkeiten der Zukunft sind "Prompt Engineering" (die klare Kommunikation mit der Maschine) und "kritisches Denken", um die Ergebnisse der KI professionell zu bewerten.

### 3. Q&A Highlights: Die drängendsten Fragen
In der abschließenden Fragerunde wurden zwei Themen angesprochen, die uns alle beschäftigen:
* **Frage zur Sicherheit:** *"Wie verhindern wir, dass sensible Unternehmensdaten an öffentliche Modelle abfließen?"*
  * **Die Antwort:** Der Trend geht ganz klar zu lokalen Modellen (**Local LLMs**) oder sicheren **Enterprise-Versionen**, die unsere Eingaben garantiert nicht für ihr eigenes Training verwenden.
* **Die ultimative Frage:** *"Brauchen wir im Jahr 2026 überhaupt noch menschliche Tester?"*
  * **Die Antwort:** Ein klares Ja! Aber nicht für wiederkehrende, langweilige Klick-Aufgaben. Der Tester der Zukunft ist ein "Quality Supervisor" (Qualitätsaufseher), der die rechtliche und ethische Verantwortung für das Endprodukt trägt.

### 4. Das Vokabular für die Zukunft
Wer in den nächsten Jahren im Testing ganz vorne mitspielen will, muss diese Begriffe leben:
* **AI Transformation:** Der Prozess, unsere QA-Teams und -Strukturen fit für Künstliche Intelligenz zu machen.
* **Upskilling:** Die kontinuierliche Weiterbildung von uns Entwicklern und Testern, um Agenten-Systeme zu beherrschen.
* **Human-Centric AI:** Die Philosophie, dass die KI dem Menschen dient – und der Mensch immer die finale Kontrolle behält.
* **Continuous Testing 2.0:** Die nächste Generation von CI/CD-Pipelines, in denen KI-Agenten unsere Tests bei jeder Code-Änderung automatisch anpassen und aktualisieren.

### Das letzte Wort: "Licence to Execute"
Das Motto der Konferenz ist für mich zum Leitgedanken geworden. Wir geben der Künstlichen Intelligenz die "Lizenz zum Ausführen". Wir lassen sie Code analysieren, Testdaten generieren und Bugs finden. Aber wir – die Entwickler und Qualitätsarchitekten – behalten die Aufsicht. Diese Balance ist der Schlüssel zum Erfolg. 

Die Werkzeuge sind da. Die Agenten sind bereit. Es wird Zeit, loszulegen!


