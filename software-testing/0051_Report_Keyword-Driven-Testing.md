# Was ich aus dem Webinar „Keyword-Driven Testing" gelernt habe

*Ein verständlicher Lernbericht für alle, die – so wie ich – aus der Entwicklung kommen und mit dem Thema Testen bisher wenig zu tun hatten.*

Am **18.06.2026 um 10 Uhr** habe ich am Webinar „Keyword-Driven Testing" von **imbus** teilgenommen, gehalten von **Rolf Glunz** und **Christian Kupfer**. Im Folgenden fasse ich zusammen, was ich aus diesem Webinar mitgenommen habe – nicht als Protokoll, sondern als persönliche Lernzusammenfassung. Ich erkläre alles so einfach wie möglich, denn viele Begriffe waren für mich als Entwickler ohne Testerfahrung komplett neu.

## Mein Hintergrund

Als Full-Stack-Entwickler mit Schwerpunkt auf **C#/.NET** und **Angular** ist eine hohe Code-Qualität für mich das Fundament jeder stabilen Anwendung. In meinen bisherigen Projekten habe ich bereits mit automatisierten Tests gearbeitet, um die Zuverlässigkeit meiner Software sicherzustellen.

Einen Einblick in meine Arbeit mit Unit-Tests (xUnit), API-Testing sowie End-to-End-Tests mit Cypress bieten folgende Projekte:
* **[CatchEleven](https://github.com/jahanalem/CatchEleven/tree/master/tests/CatchEleven.Tests):** Umfangreiche Test-Suiten für Core-Logik.
* **[SmartSupervisorBot](https://github.com/jahanalem/SmartSupervisorBot/blob/main/SmartSupervisorBot.Test.Core/BotServiceTests.cs):** Tests für Bot-Services und Datenzugriff.
* **[RectanglesCalculator](https://github.com/jahanalem/RectanglesCalculator/tree/main/Nineteen.Rectangle.Test):** Mathematische Validierung und parallele Verarbeitung.
* **[LiliShop](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-unit-tests-xunit.md):** Unit-Tests in einer E-Commerce-Architektur.
* **[Postman REST API Testing](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/postman-certification-completion.md):** Zertifizierte Erfahrung im Bereich API-Automatisierung.
* **[Discount System Unit Tests](https://github.com/jahanalem/LiliShop-backend-dotnet-test/tree/main#discount-system-unit-tests--technical-documentation)**
* **[Cypress End‑to‑End Testing](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0049_Cypress-Ent-to-End-Testing.md)**

## Die Leitfrage des Webinars

Im Mittelpunkt stand die Frage, ob Keyword-Driven Testing (kurz: KDT) der **„Königsweg"** zum wirtschaftlichen Testen ist – also der beste und elegantste Weg, um Software effizient zu testen. Am Ende habe ich darauf eine klare, aber auch ehrliche Antwort gelernt, dazu komme ich am Schluss.

## Das Grundproblem: Warum braucht man so etwas überhaupt?

Zuerst habe ich gelernt, welche Schwächen die klassische, „normale" Testautomatisierung hat (also Tests, die der Computer automatisch ausführt, statt dass ein Mensch klickt):

- Man braucht **Programmierkenntnisse**. Tests werden oft in Python geschrieben, und das kann nicht jeder im Team.
- Solche Tests sind aufwendig in der **Wartung** (also im Pflegen und Anpassen), besonders in großen Teams.
- Wenn man in ein Test-Skript hineinschaut, ist oft **nicht klar, was der Test eigentlich prüfen soll**.
- Manuelle und automatisierte Tests werden meist **getrennt verwaltet**, in unterschiedlichen Werkzeugen. Dadurch geht viel mögliche Zusammenarbeit verloren.
- Test-Skripte sind selbst wie Software – sie **können Fehler enthalten** und müssen geprüft werden.

Diese Probleme sind genau der Grund, warum Keyword-Driven Testing überhaupt entstanden ist.

## Die Grundidee: Logik und Code trennen

Das Wichtigste, was ich gelernt habe, ist eigentlich ganz einfach – und für Entwickler sofort vertraut. Man teilt einen automatisierten Test in zwei Teile auf:

1. **Die Testlogik** – das ist die Beschreibung, *was* der Test tut. Sie besteht aus lesbaren Bausteinen, den sogenannten **Keywords** (auf Deutsch: Schlüsselwörter).
2. **Der Testcode** – das ist der technische Teil, *wie* es im Hintergrund umgesetzt wird. Diesen Code braucht man erst, wenn man den Test wirklich automatisieren möchte.

Verbunden werden beide Teile über die Keywords.

> **Vergleich zum Programmieren:** Ein Keyword ist im Grunde wie eine **benannte Funktion**. Statt überall denselben Code zu wiederholen, gibt man einer Aktion einen Namen – zum Beispiel `Nutzer anmelden` – und ruft sie immer wieder auf. KDT ist im Kern nichts anderes als **prozedurales Denken**, einfach angewendet auf das Testen.

Ein schöner Nebeneffekt dieser Trennung: Weil der Code nur ganz unten steckt, funktioniert die Idee auch **ohne Code** – also im manuellen Test. Dazu komme ich weiter unten noch.

Die Ziele dahinter sind:

- einfachere Automatisierung,
- hohe **Wiederverwendbarkeit** (man baut einen Baustein einmal und nutzt ihn oft),
- gute **Wartbarkeit** (man kann die Tests leicht pflegen),
- **Arbeitsteiligkeit** (mehrere Personen können gut zusammenarbeiten),
- **Klarheit** (jeder versteht, was der Test tut).

Interessant fand ich: Bis auf die Automatisierung sind das alles Ziele, die auch im manuellen Test wichtig sind.

## Was genau ist ein Keyword?

Ein Keyword ist ein wiederverwendbarer Baustein für einen Testschritt. Es besteht aus mehreren Teilen – auch hier hilft der Vergleich mit einer Funktion:

- eine kurze **Anweisung**, also der Name des Schritts (z. B. „gültiger Login"),
- **Parameter**, die man übergibt (z. B. ein Benutzername und ein Passwort) – genau wie die Argumente einer Funktion,
- die eigentliche **Funktion**, also der Teil, der die Aktion ausführt,
- eine **Beschreibung**, die in einer zentralen **Keyword-Bibliothek** gespeichert wird, damit jeder weiß, was der Baustein macht.

## Die drei Reifegrade der Testautomatisierung

Ich habe drei Stufen kennengelernt, wie „reif" eine Testautomatisierung sein kann:

**1. Capture & Replay (Aufzeichnen und Abspielen).**
Man zeichnet seine Klicks auf der Oberfläche auf, und das Werkzeug spielt sie später wieder ab. Das ist der einfachste Einstieg und liefert schnell ein Ergebnis. Der Nachteil: Sobald sich auch nur eine Kleinigkeit ändert, muss man alles neu aufzeichnen. *(Für Entwickler: ähnlich wie ein aufgenommenes Makro.)*

**2. Direkte Implementierung.**
Man schreibt die Tests direkt als Code, zum Beispiel in Python. Das ist mächtig, aber man kann schlecht zu mehreren am selben Skript arbeiten, und ohne Disziplin entsteht schnell ein **Wildwuchs**. Ein anschauliches Beispiel dazu: In einem realen Projekt gab es einmal **100 verschiedene Varianten**, um dieselbe Wartezeit (`wait`) zu programmieren. *(Für Entwickler: das ist Code ohne das DRY-Prinzip – „Don't Repeat Yourself".)*

**3. Keyword-Driven Testing.**
Erst diese Stufe bringt alle oben genannten Ziele unter einen Hut: Lesbarkeit, Wiederverwendbarkeit und Wartbarkeit zusammen. Und ein Keyword kann andere Keywords enthalten – Schritte lassen sich also ineinander verschachteln. Das führt direkt zum nächsten, sehr wichtigen Punkt.

## Das Schichtenmodell (Layer-Konzept)

Das war für mich das Herzstück des ganzen Konzepts. Die Idee löst ein interessantes Problem: Verschiedene Leser brauchen unterschiedlich viele Details. Ein erfahrener Tester braucht nur einen groben Schritt. Der Computer dagegen braucht jede winzige Einzelanweisung. Die Lösung ist eine **mehrschichtige Beschreibung**, bei der jeder auf „seiner" Ebene liest:

| Schicht | Was steht dort? | Beispiel |
|---|---|---|
| **High-Level** | zusammengefasste, fachliche Schritte | „neues Konto anlegen" |
| **Intermediate-Level** | einzelne, technische Arbeitsschritte | „als Administrator anmelden", „Seite öffnen" |
| **Low-Level** | atomare (kleinste) Anweisungen | `fill text`, `click`, Vorname, E-Mail … |

Nur die unterste Schicht wird in einer echten Programmiersprache umgesetzt – und auch nur als einzelne, kleine Funktionen. Deshalb spricht man von einem **Low-Code-Ansatz**: Man muss kaum noch „drumherum" programmieren, sondern setzt die fertigen Bausteine zusammen.

> **Vergleich zum Programmieren:** Wer mit Komponenten arbeitet (zum Beispiel in Angular), kennt das Prinzip. Eine große Seiten-Komponente besteht aus kleineren Funktions-Komponenten, und die wiederum aus ganz kleinen Bausteinen. Genau dieselbe Verschachtelung gibt es bei den Keyword-Schichten.

Eine wichtige Lektion dabei: Man sollte sich bei der Anzahl der Schichten **disziplinieren** und vorher einen Plan machen. Es gibt Projekte mit 50 Schichten – und dann findet niemand mehr etwas. Drei bis vier Schichten reichen normalerweise völlig aus. Diese Schichten passen oft auch zu den Ebenen im Unternehmen: Die Geschäftsprozesse sind die **fachliche** Schicht, die Einzelaktionen die **technische**, und die direkte Interaktion mit dem System die **technologische** Schicht.

## Keyword-Driven Testing ist auch im manuellen Test stark

Das war für mich einer der überraschendsten Punkte. Viele glauben, Keywords lohnen sich nur, wenn man programmiert. Das ist ein **Trugschluss** (also ein Denkfehler).

Die High-Level- und Intermediate-Keywords sind nämlich schon von sich aus perfekte, gut verständliche **Arbeitsanweisungen** für einen menschlichen Tester. Das Schöne daran: Der manuelle Tester und der Automatisierer **sprechen dieselbe Sprache**. Der manuelle Tester schreibt einen Testfall aus den Bausteinen zusammen, und der Automatisierer legt später einfach den Code darunter. So verschwinden die typischen Missverständnisse an der **Schnittstelle** zwischen Fachbereich und Technik. Außerdem werden die manuellen Tests dadurch einheitlich: keine schwammigen, frei formulierten Schritte mehr, sondern klare, wiederverwendbare Bausteine.

## Best Practices aus der Praxis

Der Ansatz klingt einfach, aber – wie immer – steckt der Teufel im Detail. Drei goldene Regeln für den nachhaltigen Einsatz habe ich mitgenommen:

1. **Schichten-Disziplin.** Nicht zu viele Ebenen bauen. Drei bis vier reichen, sonst wird alles unwartbar.
2. **Namenskonventionen und klare Regeln (Governance).** Keywords müssen eindeutig benannt sein. Wenn ein Keyword „Nutzer einloggen" heißt und ein anderes „Anmeldung User" für dieselbe Sache, entsteht **Redundanz** (unnötige Doppelung), und der Vorteil der Wiederverwendung ist weg. Es muss klar geregelt sein, wer Keywords anlegen und ändern darf.
3. **Regelmäßiges Refactoring.** Eine Keyword-Bibliothek altert wie normaler Code. Man muss regelmäßig aufräumen, alte Keywords entfernen oder zusammenfassen. Sonst bricht das System irgendwann unter seinem eigenen Gewicht zusammen.

Dazu kam noch eine sehr praktische Erkenntnis: Refactoring funktioniert meistens **nicht „nebenher"**, weil es im Alltag untergeht. Besser ist es, feste Zeiten einzuplanen – zum Beispiel ein bis zwei Stunden am Ende jedes Sprints. Macht man das laufend, ist der Aufwand klein. Ignoriert man es ein Jahr lang, braucht man danach eine ganze Woche zum Aufräumen.

## Werkzeuge und Frameworks

Zum Schluss noch ein kurzer Überblick über die gängigen Werkzeuge:

- **Robot Framework** – das bekannteste Open-Source-Werkzeug. Es basiert auf Python und ist genau nach dem Keyword-Prinzip aufgebaut. Man schreibt die Testfälle in Tabellenform.
- **imbus TestBench** – ein kommerzielles Werkzeug, das den ganzen Ablauf abdeckt: von der manuellen Beschreibung über die Keyword-Struktur bis zur Anbindung an Automatisierungs-Werkzeuge.
- **QF-Test** – unterstützt ebenfalls modulare, keyword-ähnliche Strukturen und macht Aufzeichnungen wartbarer.

Eine wichtige Erkenntnis dabei: Am Ende kommt es **weniger auf das konkrete Werkzeug an** als darauf, dass man das Konzept dahinter konsequent durchzieht.

Außerdem habe ich gelernt, dass man ein bestehendes System dafür **nicht komplett neu schreiben muss**. Man muss das **Rad nicht neu erfinden**. Vorhandene Skripte (zum Beispiel in Python) lassen sich wunderbar als Basis für die Low-Level-Keywords verwenden – man legt sozusagen eine logische Hülle darum. Der technische Kern bleibt; man definiert nur die fachliche Ebene darüber.

## Fazit: Königsweg oder Sackgasse?

Und damit zurück zur Leitfrage. Die Antwort, die ich mitgenommen habe, lautet: **„Ja, aber mit Bedingungen."**

KDT ist der Königsweg, wenn man auf **Skalierung** und langfristige Wartbarkeit setzt. Aber – und das ist ehrlich gesagt der Kern – die Wirtschaftlichkeit stellt sich **nicht am ersten Tag** ein. Am Anfang hat man sogar einen höheren Aufwand (den **Initialaufwand**), weil man die Bibliothek und die Schichten erst aufbauen muss.

Der eigentliche Gewinn kommt später, in der **Wartungsphase**: Wenn sich die Anwendung ändert, passt man **ein einziges Low-Level-Keyword an einer einzigen Stelle** an – und Hunderte Testfälle sind sofort wieder korrekt. Das ist die wahre Wirtschaftlichkeit.

Wer aber keinen **langen Atem** hat oder die Disziplin bei den Best Practices schleifen lässt, für den wird aus dem Königsweg schnell eine **Sackgasse**.

> **Vergleich zum Programmieren:** Das ist genau der bekannte DRY-Gedanke. Man investiert einmal mehr Zeit, um gemeinsame Logik an einer Stelle zu bündeln. Kommt dann eine Änderung, ändert man nur diese eine Stelle – nicht hundert.

## Mein persönliches Fazit

Als Entwickler hatte ich beim Zuhören oft ein vertrautes Gefühl. Keyword-Driven Testing ist im Kern nichts anderes, als **gute Prinzipien aus der Softwareentwicklung auf das Testen zu übertragen**: Aufgaben trennen, Dinge wiederverwenden, sauber benennen und regelmäßig aufräumen. Vieles, was ich beim Programmieren ganz selbstverständlich mache, gilt hier eben für die Tests. Genau das macht das Thema für mich als Einsteiger gut zugänglich – und überraschend logisch.


# 📘 Deutscher Fachwortschatz – Webinar „Keyword-Driven Testing"

> **Berufsdeutsch B2–C1** · Vokabelliste aus dem imbus-Webinar (Rolf Glunz & Christian Kupfer)
> Spalten: **Deutsch · English · فارسی · Beispielsatz (Deutsch)**

---

## 1. Substantive – Arbeit, Firma & Projekt

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| die Vertriebsleitung | sales management | مدیریت فروش | Die Vertriebsleitung stellt das neue Produkt vor. |
| die Geschäftsstellenleitung | branch office management | مدیریت شعبه | Er hat die Geschäftsstellenleitung in Frankfurt übernommen. |
| die Veranstaltungsreihe | event series | مجموعه رویدادها | Dieses Webinar ist Teil einer Veranstaltungsreihe. |
| die Qualitätssicherung | quality assurance | تضمین کیفیت | Die Qualitätssicherung prüft jede neue Version der Software. |
| die Teamleitung | team leadership | سرپرستی تیم | Sie übernimmt die Teamleitung des Projekts. |
| die Einführung | introduction / rollout | پیاده‌سازی، معرفی | Die Einführung des neuen Systems dauerte mehrere Monate. |
| der Lösungsansatz | solution approach | رویکرد حل مسئله | Wir brauchen einen neuen Lösungsansatz für dieses Problem. |
| die Zielsetzung | objective / goal-setting | هدف‌گذاری | Die Zielsetzung des Projekts ist klar definiert. |
| der Geschäftsprozess | business process | فرایند کسب‌وکار | Jeder Geschäftsprozess wird durch eigene Tests abgedeckt. |
| die Toollandschaft | tool landscape / tool ecosystem | اکوسیستم ابزارها | Die Toollandschaft im Unternehmen ist sehr komplex. |
| die Synergie | synergy | هم‌افزایی | Durch die Zusammenarbeit entstehen wertvolle Synergien. |
| die Nachverfolgbarkeit | traceability | قابلیت ردیابی | Die Nachverfolgbarkeit der Testfälle ist sehr wichtig. |
| der Implementierungsaufwand | implementation effort | میزان تلاش لازم برای پیاده‌سازی | Der Implementierungsaufwand für diese Funktion ist hoch. |
| der Reifegrad | maturity level | سطح بلوغ | Der Reifegrad der Testautomatisierung steigt mit der Zeit. |
| die Arbeitsanweisung | work instruction | دستورالعمل کاری | Die Keywords dienen als klare Arbeitsanweisung für den Tester. |
| der Fachbereich | department / specialist area | بخش تخصصی شرکت | Der Fachbereich versteht die Testfälle, ohne Code zu lesen. |
| die Schnittstelle | interface | رابط، نقطه اتصال | An der Schnittstelle zwischen Technik und Fachbereich entstehen oft Missverständnisse. |
| der Baustein | building block / module | بلوک سازنده | Jeder Testfall besteht aus wiederverwendbaren Bausteinen. |
| die Hierarchie | hierarchy | سلسله‌مراتب | Eine zu tiefe Hierarchie macht das System unwartbar. |
| die Namenskonvention | naming convention | استاندارد نام‌گذاری | Ohne klare Namenskonventionen entstehen schnell Redundanzen. |
| die Governance | governance | حاکمیت، قوانین مدیریتی | Es braucht eine klare Governance, wer Keywords ändern darf. |
| die Redundanz | redundancy | افزونگی، تکرار غیرضروری | Wir müssen die Redundanzen im Code eliminieren. |
| das Refactoring | refactoring | بازسازی و بهبود ساختار کد | Regelmäßiges Refactoring hält die Keyword-Bibliothek sauber. |
| die Wartungsphase | maintenance phase | مرحله نگهداری | Der eigentliche Nutzen zeigt sich erst in der Wartungsphase. |
| der Initialaufwand | initial effort | هزینه و تلاش اولیه | Der Initialaufwand ist hoch, aber langfristig lohnt es sich. |
| die Skalierung | scaling / scalability | مقیاس‌پذیری | Keyword-Driven Testing eignet sich gut für die Skalierung. |
| die Investition | investment | سرمایه‌گذاری | Gute Tests sind eine Investition in die Zukunft. |
| die Sackgasse | dead end | بن‌بست | Ohne Disziplin wird der Ansatz schnell zur Sackgasse. |
| die Aufmerksamkeit | attention | توجه | Vielen Dank für Ihre Aufmerksamkeit. |
| die Teilnahme | participation | مشارکت | Wir danken für die rege Teilnahme am Webinar. |

---

## 2. Verben

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| umtreiben | to preoccupy / be on one's mind | ذهن کسی را مشغول کردن | Dieses Thema treibt uns schon lange um. |
| preisgeben | to reveal / disclose | افشا کردن، در اختیار قرار دادن | Wir geben gerne einen Teil unseres Wissens preis. |
| reingrätschen | to butt in / interrupt *(coll.)* | وسط صحبت پریدن | Wenn Sie eine Frage haben, grätschen Sie einfach rein. |
| aufgeschmissen sein | to be stuck / helpless *(coll.)* | درمانده بودن | Ohne die Technik bin ich völlig aufgeschmissen. |
| sich kümmern um | to take care of | رسیدگی کردن به | Wir kümmern uns gleich um den ersten Punkt. |
| ansteuern | to control / drive / target | کنترل کردن، هدایت کردن | Das Skript steuert bestimmte Hardware-Zustände an. |
| betreuen | to support / look after | پشتیبانی کردن، رسیدگی کردن | Wir betreuen eine große Lebensversicherung. |
| sich umhören | to ask around / look around | جست‌وجو و پرس‌وجو کردن | Ich wollte mich umhören, was es Neues am Markt gibt. |
| angehen | to tackle / address | پرداختن به، حل کردن | Keyword-Driven Testing geht diese Probleme an. |
| nachvollziehen | to comprehend / follow | درک کردن، دنبال کردن | Es ist nicht immer leicht nachzuvollziehen, was der Test macht. |
| rausziehen | to extract / pull out *(coll.)* | استخراج کردن | Daraus kann man wertvolle Synergien rausziehen. |
| verschlagworten | to tag / index with keywords | برچسب‌گذاری کردن | Die Grundidee ist, Testschritte zu verschlagworten. |
| kombinieren | to combine | ترکیب کردن | Wir kombinieren mehrere Keywords zu einem Testfall. |
| aufführen | to list / itemize | فهرست کردن | Die Ziele sind hier noch einmal aufgeführt. |
| verschachteln | to nest | تو در تو کردن | Man kann Keywords beliebig ineinander verschachteln. |
| sich disziplinieren | to discipline oneself / restrain | منظم و محدود کردن | Man sollte sich bei der Anzahl der Schichten disziplinieren. |
| umsetzen | to implement / put into practice | پیاده‌سازی کردن | Die Logik wird technisch über eine API umgesetzt. |
| ausspielen | to leverage / play out (a strength) | به‌خوبی نشان دادن (توانایی) | Hier spielt der Ansatz seine Stärken voll aus. |
| glauben | to believe / think | تصور کردن | Häufig wird geglaubt, Keywords lohnen sich erst beim Programmieren. |
| eliminieren | to eliminate | حذف کردن | Damit eliminieren wir klassische Missverständnisse. |
| standardisieren | to standardize | استانداردسازی کردن | Der manuelle Test wird dadurch standardisiert. |
| ausformulieren | to formulate in detail / spell out | با جزئیات نوشتن | Wir haben keine schwammig ausformulierten Testschritte mehr. |
| betreiben | to operate / run / pursue | اجرا کردن، به کار بردن | Wer Keyword-Driven Testing nachhaltig betreiben will, braucht Disziplin. |
| sich beschränken auf | to limit / restrict oneself to | محدود کردن | Beschränken Sie sich auf maximal vier Schichten. |
| benennen | to name | نام‌گذاری کردن | Keywords müssen eindeutig benannt sein. |
| verändern | to change / modify | تغییر دادن | Wer darf die Keywords erstellen und verändern? |
| entfernen | to remove | حذف کردن | Man muss veraltete Keywords regelmäßig entfernen. |
| zusammenfassen | to summarize / merge / combine | ادغام و خلاصه کردن | Ähnliche Keywords kann man zusammenfassen. |
| vernachlässigen | to neglect | نادیده گرفتن | Wenn man das Refactoring vernachlässigt, kollabiert das System. |
| kollabieren | to collapse | فروپاشیدن | Irgendwann kollabiert das System unter seinem eigenen Gewicht. |
| auslegen auf | to design (for a purpose) | طراحی کردن برای هدفی مشخص | Die TestBench ist genau auf diesen Lifecycle ausgelegt. |
| abdecken | to cover | پوشش دادن | Die TestBench deckt den gesamten Lifecycle ab. |
| sich einstellen | to materialize / occur | محقق شدن | Die Wirtschaftlichkeit stellt sich nicht am ersten Tag ein. |
| schleifen lassen | to let slide / neglect | سهل‌انگاری کردن | Wenn man die Disziplin schleifen lässt, wird es eine Sackgasse. |

---

## 3. Adjektive & Adverbien

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| ungünstig | unfavorable / awkward | نامناسب | Der blaue Hintergrund ist hier ziemlich ungünstig. |
| umgehend | immediately / promptly | فوراً | Technische Details werden umgehend geklärt. |
| tiefgreifend | profound / far-reaching | عمیق | Das war eine tiefgreifende Veränderung im Unternehmen. |
| ineffizient | inefficient | ناکارآمد | Unsere alte Testautomatisierung war ineffizient. |
| nachvollziehbar | comprehensible / understandable | قابل درک | Das Testziel ist nicht immer sofort nachvollziehbar. |
| eindeutig | clear / unambiguous | واضح، بدون ابهام | Keywords müssen eindeutig benannt sein. |
| fehleranfällig | error-prone | مستعد خطا | Handgeschriebene Testskripte sind oft fehleranfällig. |
| nachhaltig | sustainable / lasting | پایدار | Wie setzt man Keyword-Driven Testing nachhaltig ein? |
| wirtschaftlich | economical / cost-effective | مقرون‌به‌صرفه | Ist das wirklich der wirtschaftlichste Weg? |
| wiederverwendbar | reusable | قابل استفاده مجدد | Wir bauen präzise, wiederverwendbare Bausteine. |
| mäßig | moderate / mediocre | متوسط | Bei Capture & Replay ist die Klarheit nur mäßig. |
| atomar | atomic / very granular | اتمی، بسیار جزئی | Im Low-Level stehen die atomaren Anweisungen. |
| fachlich | technical (domain) / professional | تخصصی، حوزه کاری | Die Geschäftsprozesse bilden die fachliche Schicht. |
| technologisch | technological | فناورانه | Die Interaktion mit dem System ist die technologische Schicht. |
| massiv | massive / huge | بسیار زیاد | Das ist ein massiver Vorteil für die Automatisierung. |
| strukturiert | structured | ساختارمند | High-Level-Keywords sind gut strukturierte Arbeitsanweisungen. |
| verständlich | understandable | قابل فهم | Die Testfälle sind für jeden verständlich. |
| schwammig | vague / wishy-washy | مبهم | Früher hatten wir schwammige Testschritte. |
| präzise | precise | دقیق | Wir arbeiten mit präzisen Bausteinen. |
| handfest | concrete / solid / tangible | ملموس، عملی | Jetzt kommen die handfesten Erfahrungen aus der Praxis. |
| unwartbar | unmaintainable | غیرقابل نگهداری | Zu viele Ebenen machen das System unwartbar. |
| veraltet | outdated / obsolete | قدیمی | Veraltete Keywords muss man entfernen. |
| langfristig | long-term | بلندمدت | Langfristig lohnt sich die Investition. |
| simpel | simple | ساده | Eine simple direkte Programmierung ist am Anfang schneller. |
| kontinuierlich | continuous(ly) | پیوسته | Wenn Sie kontinuierlich aufräumen, bleibt der Aufwand klein. |
| dediziert | dedicated | اختصاصی | Man sollte dafür dedizierte Zeiten einplanen. |
| proprietär | proprietary | اختصاصی (متعلق به یک شرکت) | Wir nutzen ein proprietäres Testsystem. |

---

## 4. Redewendungen & feste Wendungen

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| einen Überblick haben | to have an overview | دید کلی داشتن | Wir wollen einen kurzen Überblick über das Thema haben. |
| in Interaktion treten | to interact / engage | وارد تعامل شدن | Im Live-Webinar können wir in Interaktion treten. |
| einen Blick auf etwas werfen | to take a look at sth | نگاهی به چیزی انداختن | Werfen wir einen Blick auf die Tools und Frameworks. |
| unter einen Hut bringen | to reconcile / bring together | هماهنگ و یکپارچه کردن | Keyword-Driven Testing bringt alle Ziele unter einen Hut. |
| sich Gedanken machen | to think carefully / consider | فکر کردن، برنامه‌ریزی کردن | Man muss sich vorher ein paar Gedanken machen. |
| auf etwas Wert legen | to value sth / attach importance | برای چیزی اهمیت قائل شدن | Wir legen großen Wert auf Wiederverwendbarkeit. |
| den Überblick behalten | to keep track / stay on top | کنترل و اشراف داشتن | In großen Teams ist es schwer, den Überblick zu behalten. |
| eine niedrige Einstiegshürde haben | to have a low barrier to entry | مانع ورود کمی داشتن | Capture & Replay hat eine niedrige Einstiegshürde. |
| ein Dilemma lösen | to solve a dilemma | یک دوراهی یا مشکل پیچیده را حل کردن | Die mehrschichtige Spezifikation löst dieses Dilemma. |
| seine Stärken ausspielen | to play to one's strengths | نقاط قوت خود را نشان دادن | Im manuellen Test spielt der Ansatz seine Stärken aus. |
| ein Trugschluss sein | to be a fallacy / misconception | یک برداشت اشتباه بودن | Das ist ein Trugschluss. |
| im Detail stecken | (the devil) is in the details | در جزئیات نهفته بودن | Der Teufel steckt wie immer im Detail. |
| das Rad neu erfinden | to reinvent the wheel | دوباره اختراع کردن چیزی که از قبل وجود دارد | Sie müssen das Rad nicht neu erfinden. |
| unter seinem eigenen Gewicht kollabieren | to collapse under its own weight | زیر بار پیچیدگی خودش فروپاشیدن | Sonst kollabiert das System unter seinem eigenen Gewicht. |
| in der Praxis | in practice | در عمل | Wie groß ist der Aufwand in der Praxis? |
| den langen Atem haben | to have staying power / persevere | صبر و پشتکار بلندمدت داشتن | Man braucht einen langen Atem für solche Projekte. |
| sich nicht am ersten Tag einstellen | not to happen on day one | فوراً حاصل نشدن | Die Wirtschaftlichkeit stellt sich nicht am ersten Tag ein. |
| sich perfekt ergänzen | to complement each other perfectly | کاملاً مکمل هم بودن | Python und Keywords ergänzen sich perfekt. |

---

## 5. Konnektoren & Signalwörter

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| grundsätzlich | basically / in principle | اساساً | Grundsätzlich ist es ein Live-Webinar. |
| dementsprechend | accordingly / therefore | بر این اساس، در نتیجه | Es ist ein Live-Webinar; dementsprechend können wir interagieren. |
| häufig | often / frequently | اغلب | Das ist häufig ein Problem in der Testautomatisierung. |
| üblicherweise | usually | معمولاً | Die Dokumentation führt man üblicherweise zentral. |
| entsprechend | correspondingly / accordingly | متناسب با آن | Die Skripte müssen entsprechend gewartet werden. |
| letzten Endes | ultimately / in the end | در نهایت | Letzten Endes ist das mein Testfall. |
| letztendlich | ultimately / after all | در نهایت، بالأخره | Letztendlich kommt es auf das Konzept an, nicht auf das Tool. |
| beispielsweise | for example | برای مثال | Beispielsweise ist das Anlegen eines Nutzers ein Keyword. |
| hinsichtlich | regarding / with respect to | از نظر، در خصوص | Hinsichtlich der Wartbarkeit ist der Ansatz überlegen. |
| durchaus | quite / absolutely / indeed | کاملاً، واقعاً | Das kann durchaus sehr stark sein. |

---

## 6. ⭐ Bonus – weitere nützliche Begriffe aus dem Transkript

> Diese Wörter standen nicht in den ChatGPT-Listen, sind aber im Webinar gefallen und sehr lohnenswert.

| Deutsch | English | فارسی | Beispielsatz |
| --- | --- | --- | --- |
| der Königsweg | the royal road / ideal solution | راه‌حل ایده‌آل | Ist Keyword-Driven Testing der Königsweg zum wirtschaftlichen Testen? |
| der Wildwuchs | uncontrolled growth / sprawl | رشد بی‌نظم و افسارگسیخته | Ohne Konventionen entsteht schnell ein großer Wildwuchs. |
| die Wartbarkeit | maintainability | قابلیت نگهداری | Eine gute Wartbarkeit ist eines der Hauptziele. |
| der Wartungsaufwand | maintenance effort | هزینه نگهداری | Handgeschriebener Code verursacht hohen Wartungsaufwand. |
| das Schlüsselwort | keyword | کلیدواژه | Ein Schlüsselwort ist die deutsche Bezeichnung für ein Keyword. |
| die Schicht | layer | لایه | Wir empfehlen maximal drei oder vier Schichten. |
| die Wiederverwendbarkeit | reusability | قابلیت استفاده مجدد | Die Wiederverwendbarkeit ist ein großer Vorteil des Ansatzes. |
| der Übergabeparameter | parameter / argument | پارامتر ورودی | Benutzername und Passwort sind Übergabeparameter. |
| das Schmankerl | a (special) treat / bonus *(coll.)* | یک چیز ویژه و جذاب | Als kleines Schmankerl zeigen wir noch einige Best Practices. |

---

### 📝 Lerntipp

Beginne mit **Abschnitt 5 (Konnektoren)** und den **Redewendungen** – diese hörst du in fast jedem Meeting. Bilde danach mit jedem neuen Wort einen eigenen Satz aus deinem Arbeitsalltag (z. B. über LiliShop oder Angular). So bleibt der Wortschatz besser hängen.

