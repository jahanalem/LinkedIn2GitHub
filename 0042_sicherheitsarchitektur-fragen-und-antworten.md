### 1. Refresh Token und Login-Prozess
**Frage:** Können Sie erklären, wie Ihr Login-System die Benutzer schützt und gleichzeitig eine reibungslose Benutzererfahrung (Smooth Experience) bietet?
**Antwort:** Wenn sich ein Benutzer anmeldet, erstellt mein Backend zwei Tokens: ein kurzlebiges Access Token und ein langlebiges Refresh Token. Das Access Token wird normal an das Frontend zurückgegeben, aber das Refresh Token wird sicher in einem `HttpOnly`-Cookie gespeichert. Das verhindert, dass Hacker es per JavaScript stehlen können. Wenn das Access Token abläuft, fängt mein Angular-Interceptor den 401-Fehler ab, pausiert den Request und sendet das `HttpOnly`-Cookie lautlos an das Backend. Das Backend rotiert die Tokens (Token Rotation), indem es ein brandneues Refresh Token und Access Token ausstellt. Danach wird der ursprüngliche Request einfach fortgesetzt, ohne dass der Benutzer etwas davon merkt.

### 2. Registrierung und E-Mail-Bestätigung (Email Confirmation)
**Frage:** Wie handhaben Sie die Benutzerregistrierung, um sicherzustellen, dass E-Mail-Adressen gültig und sicher sind?
**Antwort:** Wenn sich ein Benutzer registriert, wird sein Account erstellt, aber der `EmailConfirmed`-Status ist noch "false". Mein System generiert ein sicheres Token und speichert das genaue Erstellungsdatum. Ich wickle den E-Mail-Versand in einer `try-catch`-Schleife mit einem "Exponential Backoff" ab. Das bedeutet: Falls der SendGrid-E-Mail-Dienst kurzzeitig ausfällt, wartet das System und versucht es automatisch erneut. Die E-Mail enthält einen Link, der direkt auf meine ASP.NET Core API zeigt. Wenn der Benutzer darauf klickt, prüft das Backend, ob das Token gültig und weniger als 24 Stunden alt ist, bestätigt die E-Mail in der Datenbank und leitet den Benutzer sicher zum Angular-Frontend weiter.

### 3. Rollen- und Richtlinienbasierte Autorisierung (RBAC)
**Frage:** Was ist der Unterschied zwischen Rollen (Roles) und Richtlinien (Policies) in Ihrem Autorisierungssystem, und warum haben Sie diese getrennt?
**Antwort:** Eine Rolle ist einfach ein statisches Label für einen Benutzer, wie "Standard" oder "SuperAdmin". Eine Richtlinie (Policy) ist eine flexible Regel, die ein Endpoint erzwingt. Ich habe sie getrennt, um eine saubere Hierarchie zu schaffen. Zum Beispiel akzeptiert meine Policy `RequireAtLeastAdministratorRole` sowohl die Rolle "Administrator" als auch "SuperAdmin". Dadurch kann ich meine Controller mit einem einzigen Policy-Namen schützen, anstatt überall komplexe Logik schreiben zu müssen. Das macht den Code extrem sauber und leicht skalierbar.

### 4. Sichere Passwort-Wiederherstellung (Password Recovery)
**Frage:** Wie haben Sie die "Passwort vergessen"-Funktion implementiert, um Sicherheitslücken wie E-Mail-Enumeration zu verhindern?
**Antwort:** Sicherheit ist hier absolut kritisch. Wenn jemand eine E-Mail in das "Passwort vergessen"-Formular eingibt, prüft mein Backend, ob der Benutzer existiert. Es gibt jedoch immer eine generische Erfolgsmeldung an das Frontend zurück, unabhängig davon, ob die E-Mail echt ist oder nicht. Das hindert Hacker komplett daran, gültige E-Mail-Adressen zu erraten. Für echte Benutzer generiert das System ein sicheres, zeitlich begrenztes Token. Sobald der Benutzer sein neues Passwort absendet, wird dieses Token genau einmal verwendet und sofort zerstört, um "Replay"-Angriffe zu verhindern.

### 5. Google Single Sign-On (SSO)
**Frage:** Wie handhaben Sie den Google Single Sign-On, und warum nutzen Sie das Google-Token nicht für den API-Zugriff?
**Antwort:** Wenn ein Benutzer auf "Mit Google anmelden" klickt, erhält das Frontend ein Google ID-Token und sendet es an das Backend. Ich nutze den `JwtSecurityTokenHandler`, um E-Mail und Namen zu extrahieren. Wenn es sich um einen neuen Benutzer handelt, erstelle ich lautlos einen Account für ihn und weise die "Standard"-Rolle zu. Danach verwerfe ich das Google-Token und stelle mein eigenes, lokales Access Token und Refresh Token aus. Ich mache das, weil Google meine internen Rollen oder meine Hintergrund-Refresh-Logik nicht kennt. Indem ich eigene Tokens ausstelle, behalte ich die volle Kontrolle über die Sicherheit der Sitzung (Session).

### 6. Globaler Logout (Von allen Geräten abmelden)
**Frage:** Wie funktioniert Ihre "Von allen Geräten abmelden"-Funktion hinter den Kulissen?
**Antwort:** Immer wenn ich ein Refresh Token erstelle, generiere ich eine eindeutige Device-ID und hänge sie an den Namen des Tokens in der Datenbank an. Dadurch kann ein Benutzer mehrere aktive Sitzungen haben. Wenn ein Benutzer einen globalen Logout auslöst, aktualisiert mein Backend seinen ASP.NET Core `SecurityStamp` als eine Art Not-Aus-Schalter (Kill-Switch). Dann durchsucht es die Datenbank und löscht jedes einzelne Refresh Token, das diesem Benutzer gehört. Wenn seine anderen Geräte versuchen, ihre abgelaufenen Access Tokens lautlos zu erneuern, weist das Backend die Anfrage ab, und die Angular-App wirft den Benutzer automatisch auf den Login-Bildschirm zurück.

***
***
***

# 🔐 1. Refresh Token & Login Prozess

### ❓ Q1: Wie funktioniert dein Login-System?

**Antwort:**
In meinem System erstellt das Backend beim Login zwei Tokens:
ein Access Token und ein Refresh Token.

Das Access Token ist kurzlebig und wird für API-Anfragen verwendet.
Das Refresh Token ist langlebig und wird in einem HttpOnly-Cookie gespeichert.

So bleibt der Benutzer eingeloggt, ohne sich erneut anmelden zu müssen.

---

### ❓ Q2: Was ist der Unterschied zwischen Access Token und Refresh Token?

**Antwort:**
Das Access Token wird für API-Zugriffe verwendet und läuft schnell ab.
Das Refresh Token wird benutzt, um ein neues Access Token zu bekommen.

Das Refresh Token ist sensibler, deshalb speichere ich es sicher in einem HttpOnly-Cookie.

---

### ❓ Q3: Was passiert, wenn das Access Token abläuft?

**Antwort:**
Wenn das Access Token abläuft, gibt das Backend einen 401-Fehler zurück.

Der Angular-Interceptor erkennt diesen Fehler automatisch
und sendet eine Anfrage, um das Token zu erneuern.

Wenn das Refresh Token gültig ist, bekommt der Client ein neues Access Token
und die ursprüngliche Anfrage wird wiederholt.

---

### ❓ Q4: Was ist Token Rotation?

**Antwort:**
Token Rotation bedeutet, dass bei jeder Erneuerung
ein neues Refresh Token erstellt wird
und das alte ungültig wird.

Das erhöht die Sicherheit und reduziert das Risiko bei Token-Diebstahl.

---

### ❓ Q5: Warum verwendest du HttpOnly-Cookies?

**Antwort:**
HttpOnly-Cookies können nicht von JavaScript gelesen werden.

Das schützt das Refresh Token vor XSS-Angriffen
und macht das System sicherer.

---

# 📧 2. Registrierung & E-Mail-Bestätigung

### ❓ Q6: Warum bestätigst du E-Mail-Adressen?

**Antwort:**
Wir bestätigen E-Mails, um sicherzustellen,
dass die Adresse echt ist und dem Benutzer gehört.

Das hilft auch beim Zurücksetzen des Passworts
und verhindert Fake-Accounts.

---

### ❓ Q7: Wie funktioniert die E-Mail-Bestätigung?

**Antwort:**
Nach der Registrierung erstellt das Backend ein sicheres Token
und einen Bestätigungslink.

Dieser Link wird per E-Mail gesendet.

Wenn der Benutzer darauf klickt, überprüft das Backend das Token
und bestätigt die E-Mail.

---

### ❓ Q8: Warum zeigt der Link auf das Backend?

**Antwort:**
Weil das Backend das Token überprüfen muss.

Nach der Prüfung leitet das Backend den Benutzer
zum Frontend weiter und zeigt Erfolg oder Fehler an.

---

### ❓ Q9: Wie gehst du mit Fehlern beim E-Mail-Versand um?

**Antwort:**
Ich verwende einen Retry-Mechanismus mit Exponential Backoff.

Wenn das Senden fehlschlägt, wird es nach 2, 4 und 8 Sekunden erneut versucht.

---

### ❓ Q10: Wie behandelst du abgelaufene Tokens?

**Antwort:**
Ich speichere den Zeitpunkt der Erstellung
und erlaube nur 24 Stunden.

Wenn das Token älter ist, wird es abgelehnt.

---

# 🔑 3. RBAC (Autorisierung)

### ❓ Q11: Was ist der Unterschied zwischen Authentifizierung und Autorisierung?

**Antwort:**
Authentifizierung bedeutet: Wer bist du?

Autorisierung bedeutet: Was darfst du tun?

---

### ❓ Q12: Was ist RBAC?

**Antwort:**
RBAC bedeutet Role-Based Access Control.

Jeder Benutzer hat eine Rolle, zum Beispiel Standard oder Administrator,
und diese bestimmt den Zugriff.

---

### ❓ Q13: Was ist der Unterschied zwischen Rolle und Policy?

**Antwort:**
Eine Rolle ist ein Label für den Benutzer.

Eine Policy ist eine Regel, die festlegt,
welche Rollen Zugriff auf einen Endpunkt haben.

---

### ❓ Q14: Warum verwendest du Policy-basierte Autorisierung?

**Antwort:**
Policies machen das System flexibler und leichter skalierbar.

Wir definieren Regeln einmal
und können sie überall wiederverwenden.

---

### ❓ Q15: Was passiert, wenn ein Benutzer keine Berechtigung hat?

**Antwort:**
Wenn er nicht eingeloggt ist, bekommt er 401 Unauthorized.

Wenn er eingeloggt ist, aber keine Rechte hat, bekommt er 403 Forbidden.

---

# 🔁 4. Passwort zurücksetzen

### ❓ Q16: Wie funktioniert dein Forgot-Password-Flow?

**Antwort:**
Der Benutzer gibt seine E-Mail ein.

Das Backend erstellt ein sicheres Token
und sendet einen Reset-Link per E-Mail.

Der Benutzer klickt den Link und setzt ein neues Passwort.

---

### ❓ Q17: Wie verhinderst du Email Enumeration?

**Antwort:**
Das Backend gibt immer die gleiche Antwort zurück,
egal ob die E-Mail existiert oder nicht.

So können Angreifer keine gültigen E-Mails erkennen.

---

### ❓ Q18: Warum ist dein Reset-Token sicher?

**Antwort:**
Das Token wird kryptografisch erzeugt
und ist an den Benutzer gebunden.

Es ist zeitlich begrenzt und nur einmal gültig.

---

### ❓ Q19: Was passiert bei einem ungültigen oder abgelaufenen Token?

**Antwort:**
Das Backend lehnt die Anfrage ab
und gibt einen Fehler zurück.

Der Benutzer muss ein neues Token anfordern.

---

# 🔓 5. Google Login (SSO)

### ❓ Q20: Wie funktioniert Google Login in deinem System?

**Antwort:**
Das Frontend bekommt ein Token von Google
und sendet es an das Backend.

Das Backend liest die Benutzerdaten
und meldet den Benutzer an.

---

### ❓ Q21: Warum verwendest du das Google-Token nicht direkt?

**Antwort:**
Weil Google unsere Rollen und Berechtigungen nicht kennt.

Wir erstellen eigene Tokens,
um Sicherheit und Zugriff selbst zu kontrollieren.

---

### ❓ Q22: Was passiert bei neuen Benutzern?

**Antwort:**
Wenn der Benutzer noch nicht existiert,
wird automatisch ein Konto erstellt
und eine Standardrolle vergeben.

---

# 🚪 6. Global Logout

### ❓ Q23: Was ist Global Logout?

**Antwort:**
Global Logout bedeutet,
dass der Benutzer auf allen Geräten gleichzeitig ausgeloggt wird.

---

### ❓ Q24: Wie implementierst du Global Logout?

**Antwort:**
Wir aktualisieren den SecurityStamp des Benutzers
und löschen alle Refresh Tokens aus der Datenbank.

Außerdem löschen wir die Cookies auf dem aktuellen Gerät.

---

### ❓ Q25: Was passiert auf anderen Geräten?

**Antwort:**
Andere Geräte haben noch kurz ein gültiges Access Token.

Wenn es abläuft, versuchen sie ein neues Token zu bekommen.

Da das Refresh Token gelöscht wurde, schlägt das fehl
und der Benutzer wird ausgeloggt.

***
***
***

## 1. Refresh Token & Login-Prozess

### F1: Warum verwendest du sowohl einen Access‑Token als auch einen Refresh‑Token? Warum nicht einfach einen einzigen, langlebigen Token?
**Antwort:**  
Der Access‑Token ist kurzlebig (z. B. 15 Minuten). Falls er gestohlen wird, kann der Angreifer ihn nur kurz nutzen.  
Der Refresh‑Token wird in einem `HttpOnly`-Cookie gespeichert – JavaScript kann ihn nicht lesen, daher ist er sicher vor XSS-Angriffen.  
Zusammen bieten sie Sicherheit (kurzer Access‑Token) und eine gute Benutzererfahrung (stiller Refresh, ohne dass der Benutzer sich erneut anmelden muss).

### F2: Was passiert, wenn der Access‑Token abläuft und der Benutzer eine API-Anfrage macht?
**Antwort:**  
Das Backend antwortet mit `401 Unauthorized`.  
Der **Error Interceptor** von Angular fängt diesen Fehler ab, pausiert die Anfrage und ruft automatisch den `/refresh-token`-Endpunkt auf. Der Browser sendet dabei das `HttpOnly`-Cookie mit dem Refresh‑Token mit.  
Ist der Refresh‑Token gültig, erzeugt das Backend einen neuen Access‑Token und **auch einen neuen** Refresh‑Token (Token‑Rotation). Dann wiederholt der Interceptor die ursprüngliche Anfrage – der Benutzer merkt nichts.

### F3: Was ist Token‑Rotation und warum ist sie wichtig?
**Antwort:**  
Token‑Rotation bedeutet, dass wir bei jeder Auffrischung einen **völlig neuen** Refresh‑Token erstellen und den alten ungültig machen.  
Wird ein Refresh‑Token gestohlen, kann er nur einmal verwendet werden. Der Angreifer kann ihn nicht erneut nutzen, weil der Server ihn bereits ersetzt hat. Das reduziert den Schaden durch gestohlene Refresh‑Tokens erheblich.

---

## 2. Registrierung & E‑Mail‑Bestätigung

### F1: Wie verhindert dein System, dass gefälschte oder inaktive E‑Mail‑Adressen verwendet werden?
**Antwort:**  
Nach der Registrierung senden wir eine Bestätigungs‑E‑Mail mit einem eindeutigen, kryptografisch sicheren Token. Der Benutzer muss den Link in dieser E‑Mail anklicken.  
Erst nach dem Klick wird das Konto vollständig aktiviert. So stellen wir sicher, dass die E‑Mail‑Adresse wirklich dem Benutzer gehört.

### F2: Was passiert, wenn der Benutzer mehr als 24 Stunden wartet, bevor er den Bestätigungslink anklickt?
**Antwort:**  
Unser Code prüft das **Ausstellungsdatum des Tokens**. Wenn der Unterschied zwischen `DateTime.UtcNow` und dem Ausstellungsdatum mehr als 24 Stunden beträgt, lehnen wir die Bestätigung ab.  
Der Benutzer sieht eine Meldung, dass der Token abgelaufen ist, und muss eine neue Bestätigungs‑E‑Mail anfordern.

### F3: Wie gehst du mit E‑Mail‑Zustellungsfehlern um (z. B. wenn der E‑Mail‑Server gerade nicht erreichbar ist)?
**Antwort:**  
Wir verwenden einen **Wiederholungsmechanismus mit exponentiellem Backoff**.  
Wir versuchen die E‑Mail bis zu 3‑mal zu senden. Die erste Wiederholung nach 2 Sekunden, dann nach 4 Sekunden, dann nach 8 Sekunden. Wenn alle Versuche fehlschlagen, loggen wir den Fehler und der Benutzer wird informiert, es später erneut zu versuchen. Das macht das System fehlertolerant.

---

## 3. Rollenbasierte & richtlinienbasierte Autorisierung (RBAC)

### F1: Was ist der Unterschied zwischen einer Rolle und einer Richtlinie (Policy) in deinem System?
**Antwort:**  
Eine **Rolle** ist eine einfache Bezeichnung, die einem Benutzer zugeordnet wird, z. B. `Standard`, `Administrator` oder `SuperAdmin`.  
Eine **Richtlinie (Policy)** ist eine Regel, die ein API‑Endpunkt durchsetzt. Zum Beispiel erlaubt die Richtlinie `RequireAtLeastAdministratorRole` sowohl `Administrator` als auch `SuperAdmin` den Zugriff.  
Richtlinien sind flexibler, weil wir eine **Hierarchie** einmal definieren und dann überall den Richtliniennamen verwenden – wir müssen nicht in jedem Controller komplexe `if`-Abfragen schreiben.

### F2: Wenn ein Benutzer mit der Rolle `Standard` auf einen Endpunkt zugreifen will, der mit `RequireAtLeastAdministratorRole` geschützt ist, welchen HTTP‑Statuscode bekommt er und warum?
**Antwort:**  
Er bekommt `403 Forbidden`.  
Er ist authentifiziert (er hat einen gültigen Token), also ist es nicht `401 Unauthorized`. Aber seine Rolle erfüllt nicht die Richtlinie, daher verweigert der Server korrekt den Zugriff.

### F3: Wie stellst du sicher, dass Rollen- und Richtliniennamen im gesamten Projekt einheitlich sind?
**Antwort:**  
Wir definieren alle Rollennamen und Richtliniennamen als **Konstanten** in einem gemeinsamen `Domain`-Projekt (z. B. `Role.SuperAdmin`, `PolicyType.RequireAtLeastAdministratorRole`).  
Diese Konstanten verwenden wir in den Controllern und in der `Startup`-Konfiguration. Das verhindert Tippfehler und macht Umbenennungen einfach – Änderung an einer Stelle, und das gesamte Projekt wird aktualisiert.

---

## 4. Sicheres Passwort‑Zurücksetzen (Passwort vergessen)

### F1: Wie verhindert dein System, dass ein Angreifer herausfindet, welche E‑Mail‑Adressen in deiner Datenbank registriert sind?
**Antwort:**  
Wir verwenden eine Technik namens **„No Email Enumeration“** (keine Aufzählung von E‑Mails).  
Wenn ein Benutzer das „Passwort vergessen“-Formular absendet, geben wir immer die gleiche allgemeine Erfolgsmeldung zurück – selbst wenn die E‑Mail nicht in unserer Datenbank existiert.  
Der Angreifer kann nicht unterscheiden, ob eine E‑Mail echt ist oder nicht. Das verbirgt unsere Benutzerliste.

### F2: Was passiert, wenn ein Benutzer versucht, einen sehr alten Passwort‑Zurücksetzungs‑Token zu verwenden (z. B. von vor einer Woche)?
**Antwort:**  
Der Token wird abgelehnt.  
Unsere `ResetPasswordAsync`-Methode verwendet die eingebaute Validierung von ASP.NET Core Identity, die auch das Ablaufdatum prüft.  
Selbst wenn der Token‑String korrekt ist, gibt das System einen „ungültiger oder abgelaufener Token“-Fehler zurück. Der Benutzer muss eine neue Zurücksetzungs‑E‑Mail anfordern.

### F3: Warum erzeugst du den Zurücksetzungslink, der auf das Angular‑Frontend zeigt, statt direkt auf das Backend?
**Antwort:**  
Das Backend ist nur für die Token‑Validierung und die Passwortänderung zuständig. Das Frontend ist dafür da, das Formular „Neues Passwort eingeben“ anzuzeigen.  
Wir übergeben den Token und die E‑Mail als URL‑Parameter an die Angular‑Route. Angular extrahiert sie und sendet sie dann in einer `POST /reset-password`-Anfrage an das Backend. Das sorgt für eine saubere Benutzererfahrung und trennt die Zuständigkeiten.

---

## 5. Google Externer Login (Single Sign‑On / SSO)

### F1: Wenn sich ein Benutzer mit Google anmeldet, warum erzeugt dein System dann eigene Access‑ und Refresh‑Tokens, anstatt den Google‑Token zu verwenden?
**Antwort:**  
Der Google‑Token sagt uns nur, wer der Benutzer ist (E‑Mail und Name). Er kennt **nicht** unsere internen Rollen wie `Administrator` oder `Standard`. Außerdem funktioniert der Google‑Token nicht mit unserem stillen Refresh‑Mechanismus.  
Deshalb „übersetzen“ wir den Google‑Token in unser eigenes Token‑Paar. So behalten wir die volle Kontrolle über die Sitzungsdauer und die Berechtigungen.

### F2: Was passiert, wenn sich ein Benutzer zum ersten Mal mit Google anmeldet? Muss er sich separat registrieren?
**Antwort:**  
Nein. Unsere `GoogleLoginAsync`-Methode prüft, ob die E‑Mail bereits in unserer Datenbank existiert.  
Wenn nicht, **erstellt sie automatisch** ein neues Benutzerkonto, weist die Rolle `Standard` zu und meldet den Benutzer an – alles in einem Schritt. Der Benutzer muss kein Registrierungsformular ausfüllen.

### F3: Wie stellst du sicher, dass der Google‑Login-Benutzer in dein Refresh‑Token‑System eingebunden bleibt?
**Antwort:**  
Nachdem wir den Google‑ID‑Token erhalten haben, erzeugen wir unseren eigenen Refresh‑Token – genau wie beim normalen Login. Dieser Refresh‑Token wird in einem `HttpOnly`-Cookie gespeichert.  
Somit profitiert der Google‑Benutzer vom gleichen stillen Refresh‑Prozess: Wenn der Access‑Token abläuft, sendet der Browser automatisch das Refresh‑Token‑Cookie mit, um einen neuen Access‑Token zu erhalten.

---

## 6. Globales Logout (Von allen Geräten abmelden)

### F1: Wie verfolgt dein System verschiedene Geräte, damit du von allen gleichzeitig abmelden kannst?
**Antwort:**  
Bei der Anmeldung erzeugen wir eine eindeutige **Geräte‑ID** (eine GUID). Wir speichern den Refresh‑Token in der Tabelle `AspNetUserTokens` mit einem Namen wie `RefreshToken_<GeräteId>`.  
Wenn sich derselbe Benutzer von drei verschiedenen Browsern anmeldet, haben wir drei separate Refresh‑Token‑Datensätze, jeder mit einer anderen Geräte‑ID.

### F2: Was passiert genau, wenn ein Benutzer auf „Von allen Geräten abmelden“ klickt?
**Antwort:**  
Drei Dinge passieren:
1. Wir erzeugen einen **neuen SecurityStamp** für den Benutzer. Das macht alle vorhandenen Access‑Tokens sofort ungültig.
2. Wir löschen **jeden** Refresh‑Token aus der Datenbank, der zu diesem Benutzer gehört (alle Geräte‑IDs).
3. Wir löschen die `refreshToken`- und `deviceId`-Cookies im aktuellen Browser.  
Der Benutzer wird auf dem aktuellen Gerät abgemeldet, und jedes andere Gerät wird gezwungen, sich erneut anzumelden, sobald dessen Access‑Token abläuft oder es versucht, aufzufrischen.

### F3: Angenommen, du klickst auf dem Laptop auf „Von allen Geräten abmelden“. Dein Handy ist noch angemeldet – wie wird das Handy automatisch abgemeldet?
**Antwort:**  
Das Handy weiß nicht sofort Bescheid. Aber der Access‑Token auf dem Handy ist kurzlebig (z. B. 15 Minuten).  
Wenn dieser Token abläuft, versucht das Handy, sich still aufzufrischen, indem es seinen Refresh‑Token verwendet. Das Backend sucht diesen Refresh‑Token in der Datenbank – aber wir haben während des globalen Logouts alle Refresh‑Tokens gelöscht.  
Die Auffrischung schlägt fehl, und der Error‑Interceptor auf dem Handy bekommt eine `401`. Das Handy löscht seinen lokalen Zustand und leitet den Benutzer zum Anmeldebildschirm weiter. Somit wird das Handy innerhalb weniger Minuten abgemeldet.

***

