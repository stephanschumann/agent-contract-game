# Backlog

> Hinweis: Diese Datei existierte lokal noch nicht und wurde am 20.07.2026 neu angelegt. Die Ticket-Historie bis v1.4.2 (FEATURE-001 bis FEATURE-003, TASK-001) ist bereits in der Claude-Projekt-Wissensablage „Agentic Engineering Gamification" dokumentiert und wird hier nicht dupliziert. Ab jetzt ist diese lokale Datei die laufende Quelle für neue Tickets.
>
> Verschoben am 20.07.2026 aus dem übergeordneten Ordner `Agentic Engineering Gamification/` in dieses Projektverzeichnis (`agent-contract-game/`), damit Backlog.md und Product.md direkt im selben Projektordner wie der Code liegen.

## 🔄 In Progress

### BUG-001 Fehlermeldung bei der zweiten Zuordnungsaufgabe passt nicht zu deren Kategorien

| Feld | Wert |
|------|------|
| **Typ** | BugFix |
| **Priorität** | Mittel |
| **Status** | In Progress |
| **Erstellt** | 2026-07-21 |
| **In Progress seit** | 2026-07-21 |

**Beschreibung:** Ein Nutzer meldete per Screenshot, dass Meldungen des Agenten manchmal nicht zum gerade gespielten Schritt zu passen scheinen, sondern zu einem früheren. Prüfung ergab: Im Spiel gibt es zwei Zuordnungsaufgaben mit unterschiedlichen Kategorien (erste: Goal/Rule/Example/Open question; zweite, „Settle it before going on": Answered/bewusst offen gelassen/Blockiert). Beide teilen sich denselben Fehlertext, wenn eine Karte falsch einsortiert wurde — dieser Text ist aber fest auf die Kategorien der ERSTEN Aufgabe formuliert („what's a rule, what's just an example, what's still an open question"). Landet ein Spieler in der zweiten Aufgabe und sortiert falsch, bekommt er trotzdem diesen Text mit Kategorien, die dort gar nicht vorkommen — das wirkt wie eine Meldung aus einem früheren Schritt.

**User Story:** Als Spielerin/Spieler möchte ich bei einem Sortierfehler eine Rückmeldung bekommen, die zu den tatsächlich in diesem Schritt verfügbaren Kategorien passt, sodass ich nicht verwirrt werde und weiß, wonach ich wirklich suchen soll.

**Scope:**
Eingeschlossen: Die feste Fehlermeldung im Klick-Handler des „Hand it to the agent"-Buttons (Funktion `renderCategorize`, `public/index.html`) wird so umgebaut, dass sie die Kategorie-Namen dynamisch aus den tatsächlichen Kategorien des jeweils aktuellen Schritts (`st.buckets`) zusammensetzt, statt fest „rule/example/open question" zu nennen. Betrifft beide Zuordnungsaufgaben (erste: Goal/Rule/Example/Open question; zweite „Settle it before going on": Answered/Deliberately limited/Blocked).
Ausgeschlossen: die Erfolgsmeldung (bereits korrekt szenario-/schrittspezifisch über `st.successReaction`); Layout, Sortierlogik selbst, Kategorie-Zähler, sonstige Texte.

**Akzeptanzkriterien:**
- [x] Sortiert man in der ERSTEN Zuordnungsaufgabe eine Karte falsch, nennt die Fehlermeldung weiterhin die dortigen Kategorien (Goal/Rule/Example/Open question).
- [x] Sortiert man in der ZWEITEN Zuordnungsaufgabe („Settle it before going on") eine Karte falsch, nennt die Fehlermeldung die dortigen Kategorien (Answered/Deliberately limited/Blocked) statt der Begriffe der ersten Aufgabe.
- [x] Die Anzahl falsch einsortierter Karten wird weiterhin korrekt genannt.
- [x] Keine Konsolenfehler *(mit einer bekannten, vom Fix unabhängigen Einschränkung — siehe Testplan)*.
- [ ] Bestehende Tests aus FEATURE-004/005 erneut ausgeführt — **nicht möglich**: im Projekt liegt keine gespeicherte Testdatei aus diesen Tickets, nur die Beschreibung im Backlog. Stattdessen wurde die komplette Seite inkl. Startbildschirm real ausgeführt und beide betroffenen Zuordnungsschritte über echte Klick-Handler durchgespielt (siehe Testplan) — das deckt dieselbe Funktion ab, ist aber kein Wiederholen der historischen Testläufe selbst.

**Analyse & Planung:**
- [x] Ursache bereits lokalisiert (durch vorherige Analyse mit Subagent, am realen Live-Code auf GitHub verifiziert, danach am lokalen Git-Klon per Grep bestätigt): `public/index.html`, Funktion `renderCategorize(st)`, `else`-Zweig des `document.getElementById("check").onclick`-Handlers — der Text ist dort als fester String verdrahtet.
- [x] Kategorie-Namen sind bereits pro Schritt vorhanden: `st.buckets` (Array mit `.label`) wird schon für die Kästen selbst verwendet (`bucketsHTML`) — dieselbe Quelle kann für den Fehlertext genutzt werden, keine neue Datenstruktur nötig.
- [x] Geplanter Ansatz: aus `st.buckets.map(b=>b.label)` einen sprachlich sauberen Aufzählungstext bauen (Komma-getrennt, letztes Element mit „or") und in die bestehende Fehlermeldung einsetzen statt der festen Begriffe.
- [x] Risiko: bei nur 3 statt 4 Kategorien (zweite Aufgabe) muss die Aufzählungslogik auch mit 3 Elementen korrekt funktionieren — wird im Test geprüft.

**Testplan:**
- [x] Automatisiert (jsdom, echtes Ausführen der App im echten DOM wie in FEATURE-004/005): die komplette Seite wurde geladen und ausgeführt, dann `renderCategorize` für die erste Aufgabe (Goal/Rule/Example/Open question) über die echten Klick-Handler bedient, eine Karte bewusst falsch einsortiert — Fehlermeldung nennt korrekt „Goal, Rule, Example or Open question" und die Anzahl falscher Karten (1).
- [x] Automatisiert: dieselbe Funktion für die zweite Aufgabe („Settle it before going on", Answered/Deliberately limited/Blocked) durchgespielt, eine Karte bewusst falsch einsortiert — Fehlermeldung nennt korrekt „Answered, Deliberately limited or Blocked", enthält NICHT mehr „rule"/„example"/„open question", und die Anzahl falscher Karten stimmt.
- [x] Syntax-Check (`node --check`) fehlerfrei.
- [ ] Regressionstest gegen gespeicherte FEATURE-004/005-Tests: nicht möglich, da keine Testdatei aus diesen Tickets im Projekt abgelegt wurde (siehe Akzeptanzkriterien). Ersatzweise wurde der volle Seitenaufbau (Startbildschirm, Konfetti-Initialisierung, beide Zuordnungsschritte) fehlerfrei durchlaufen.
- **Bekannte, vom Fix unabhängige Testumgebungs-Einschränkung:** Beim ersten Laden der Seite in der jsdom-Testumgebung meldet die Konsole einen Fehler zur Canvas-Konfetti-Animation (`getContext` von `<canvas>` wird von jsdom ohne Zusatzpaket nicht unterstützt). Das betrifft ausschließlich die optische Konfetti-Anzeige beim Seitenstart, nicht die hier geänderte Sortier-Logik, und ist vermutlich auch bei den früheren Tickets FEATURE-004/005 in derselben Testumgebung aufgetreten (nicht durch dieses Ticket verursacht). Ein echter Blick im Browser zeigt keinen Konsolenfehler, da echte Browser `<canvas>` unterstützen — das ist aber wie bei FEATURE-004/005 noch nicht separat verifiziert (kein `file://`-Zugriff in dieser Sitzung).

**Scope-Änderungen** *(chronologisches Log):*
*(leer bei Erstellung)*

**Implementierungsnotizen:**
Umgesetzt in `public/index.html` (lokaler Git-Klon, Basis v1.6.0 / Commit `08da401`, vor dem Schreiben per `git log`/`git status` verifiziert — sauberer Stand, kein zwischenzeitlicher Fremdstand). In `renderCategorize(st)`, im `else`-Zweig des `check`-Button-Handlers, wurde der fest verdrahtete Text „what's a rule, what's just an example, what's still an open question" ersetzt durch einen dynamisch aus `st.buckets.map(b=>b.label)` gebauten Aufzählungstext (Komma-getrennt, letztes Element mit „or"): „Look again — is it really ‹Kategorie1, Kategorie2 or Kategorie3/4›? — and re-sort." Dieselbe Datenquelle (`st.buckets`), die schon für die sichtbaren Kategorie-Kästen verwendet wird, liefert damit auch den Fehlertext — für beide Zuordnungsaufgaben automatisch korrekt, keine Sonderfall-Unterscheidung nötig. Die Erfolgsmeldung (`st.successReaction`) war bereits vorher korrekt und wurde nicht angefasst.

## ✅ Done

### FEATURE-005 Rückblick auch während des laufenden Spiels

| Feld | Wert |
|------|------|
| **Typ** | Feature |
| **Priorität** | Mittel |
| **Status** | Done |
| **Erstellt** | 2026-07-20 |
| **In Progress seit** | 2026-07-20 |
| **Fertiggestellt** | 2026-07-20 |

**Beschreibung:** Direkte Erweiterung von FEATURE-004. Der Rückblick auf frühere, bereits abgeschlossene Schritte soll nicht nur auf der Abschlussseite, sondern auch während eines laufenden Spiels möglich sein — ohne den aktuell in Bearbeitung befindlichen Schritt zu stören oder zurückzusetzen.

**User Story:** Als Spielerin/Spieler möchte ich mitten im Spiel (z. B. auf Schritt 4) nachsehen können, was ich bei einem früheren Schritt (z. B. Schritt 1) ausgewählt hatte, sodass ich mich daran erinnern kann, ohne das Spiel neu zu starten — und danach genau dort weitermachen, wo ich war.

**Scope:**
Eingeschlossen: Die bereits während des Spiels sichtbare Kopfzeile zeigt einen Badge-Zähler (🏅-Chip), der beim Anklicken die vorhandene „Your badges"-Liste öffnet. Jeder Eintrag in dieser Liste wird zusätzlich anklickbar und öffnet denselben Rückblick wie auf der Abschlussseite (gleicher Inhalt, gleiche Ansichts-Variante). Technische Umstellung: Der Rückblick wird als eigenes Overlay (wie die bestehenden „Your badges"/„Rework log"-Fenster) über den aktuellen Bildschirm gelegt statt ihn zu ersetzen — dadurch bleibt der gerade in Bearbeitung befindliche Schritt beim Schließen exakt so erhalten, wie er war (nichts wird zurückgesetzt). Die Abschlussseite nutzt danach denselben Mechanismus (kein separater Rückweg über „renderFinale()" mehr nötig).
Ausgeschlossen: kein Zugriff auf den allerersten Schnellstart-Schritt und den Rewind-Übergangsbildschirm (weiterhin ohne Badge); keine Bearbeitung der früheren Auswahl; kein neuer, zusätzlicher UI-Button — es wird bewusst der bereits vorhandene Badge-Zähler in der Kopfzeile wiederverwendet.

**Akzeptanzkriterien:**
- [x] Während eines laufenden Spiels lässt sich über den Badge-Zähler in der Kopfzeile die Liste der bisher verdienten Badges öffnen (wie bisher).
- [x] Jeder Eintrag in dieser Liste ist zusätzlich anklickbar und zeigt darauf den letzten Bildschirm des jeweiligen Schritts inkl. eigener Auswahl — read-only wie auf der Abschlussseite.
- [x] Nach dem Schließen des Rückblicks ist der Schritt, an dem gerade gespielt wurde, unverändert vorhanden — insbesondere ein bereits halb ausgefüllter, noch nicht abgeschickter Schritt (automatisiert geprüft: eine bereits zugeordnete Karte blieb zugeordnet, der Fortschrittszähler war unverändert, derselbe DOM-Slot blieb bestehen; das Spiel ließ sich danach normal zu Ende spielen).
- [x] Dasselbe funktioniert weiterhin unverändert von der Abschlussseite aus (Regressionstest zu FEATURE-004: alle bisherigen automatisierten Prüfungen erneut fehlerfrei, jetzt gegen die Overlay-Variante).
- [x] Keine Konsolenfehler.

**Analyse & Planung:**
- [x] Risiko der naheliegenden Lösung erkannt: Ein Zurück-Weg, der den aktuellen Schritt über `render()` neu aufbaut, würde bei einem bereits halb bearbeiteten Schritt (z. B. teilweise zugeordnete Karten) den Fortschritt auf diesem Schritt löschen, weil jede Render-Funktion ihren Zuordnungs-Zustand neu und leer aufbaut.
- [x] Gewählter Ansatz: Der Rückblick wird nicht mehr in `#stageHost` hineingerendert, sondern in einem eigenen, bereits im Code vorhandenen Overlay-Muster (`.overlay`/`.modal`, wie „Your badges"/„Rework log") über den unangetasteten aktuellen Bildschirm gelegt. Schließen entfernt nur die `.show`-Klasse — der aktuelle Bildschirm darunter wurde nie verändert.
- [x] Wiederverwendung statt neuer UI: der bestehende Badge-Zähler in der Kopfzeile (`#badgeChip`) ist schon auf jedem Bildschirm sichtbar und öffnet bereits eine Liste der Badges — diese Liste wird nur um Klickbarkeit ergänzt.
- [x] Aufwand: klein, da FEATURE-004 die Datengrundlage (`S.history`, `S.badges[].i`) bereits liefert.

**Testplan:**
- [x] Automatisiert (jsdom): Spiel bis Schritt 4 gespielt, dort bewusst gestoppt, eine Karte zugeordnet (Rest offen gelassen), Badge-Zähler geöffnet, Eintrag für Schritt 2 angeklickt, Inhalt geprüft, Rückblick geschlossen, geprüft dass die eine zugeordnete Karte, der Fortschrittszähler und der DOM-Slot exakt erhalten blieben, danach Schritt 4 normal fertig gespielt.
- [x] Regressionstest: alle drei Abschlussseiten-Durchläufe aus FEATURE-004 (Maus/Touch/Versuchung) erneut ausgeführt — funktionieren unverändert mit der neuen Overlay-Umsetzung, alle 7 Badges je einzeln geprüft.
- [x] Syntax-Check (`node --check`) fehlerfrei.
- [ ] Echter visueller Blick im Browser steht wie bei FEATURE-004 noch aus (gleiche technische Einschränkung: kein `file://`-Zugriff für den Chrome-Test dieser Sitzung).

**Scope-Änderungen** *(chronologisches Log):*
*(leer bei Erstellung)*

**Implementierungsnotizen:**
Umgesetzt in `public/index.html`, direkt auf v1.5.0 aufsetzend (`git log`/`git status` vor Beginn geprüft, kein zwischenzeitlicher Stand). `openReview(i)` ersetzt nicht mehr `host.innerHTML`, sondern befüllt ein neues, dem bestehenden Overlay-Muster folgendes Fenster (`#ovReview`, wie „Your badges“/„Rework log“) und zeigt es per `.show`-Klasse an — der eigentliche Spielbildschirm in `#stageHost` wird dabei nie berührt, daher bleibt ein halb bearbeiteter Schritt beim Schließen exakt erhalten. Schließen läuft über den bereits vorhandenen generischen `[data-close]`-/Klick-außerhalb-Mechanismus, kein eigener Rückweg-Button mehr nötig (dadurch entfällt auch der Konfetti-Nebeneffekt von FEATURE-004). Der bestehende Badge-Zähler in der Kopfzeile (`#badgeChip`) — vorher nur eine reine Liste — bekommt pro Eintrag zusätzlich `data-i` und einen Klick-Handler, der die Badge-Liste schließt und direkt `openReview(i)` öffnet; dieselbe Funktion bedient jetzt sowohl die Abschlussseite als auch das laufende Spiel.

**Technische Randnotiz (kein Nutzer-Effekt, aber für später dokumentiert):** Der zuletzt angesehene Rückblick-Schnappschuss bleibt bis zum nächsten Öffnen unsichtbar im DOM von `#reviewBody` liegen (Overlay ist nur per CSS `display:none` versteckt, nicht entfernt). Da gespeicherte Schnappschüsse dieselben Element-IDs/-Klassen wie der echte, aktuelle Bildschirm verwenden können (z. B. `id="check"`, `.bucket`), wurde geprüft, dass dadurch kein echtes Fehlverhalten entsteht: `#stageHost` steht im HTML vor den Overlays, wodurch ungezielte `document.getElementById(...)`-Zugriffe im echten Spielcode immer zuerst das echte, aktuelle Element treffen; zusätzlich ist der Schnappschuss durch `pointer-events:none` und das unsichtbare, geschlossene Overlay für echte Klicks ohnehin nicht erreichbar. Im automatisierten Test musste deshalb bewusst auf `#stageHost` eingegrenzt werden, um nicht versehentlich Elemente aus dem liegen gebliebenen alten Schnappschuss mitzuzählen.

**Release:** Als v1.6.0 released (Commit `08da401`), GitHub Action grün (37s). Stephan hat live geprüft — Badge-Zähler mitten im Spiel öffnen und einen früheren Schritt anklicken funktioniert wie vorgesehen. Bestätigt und auf Done gesetzt.

---

### FEATURE-004 Rückblick auf frühere Spielschritte über die Abschlussseite

| Feld | Wert |
|------|------|
| **Typ** | Feature |
| **Priorität** | Mittel |
| **Status** | Done |
| **Erstellt** | 2026-07-20 |
| **In Progress seit** | 2026-07-20 |
| **Fertiggestellt** | 2026-07-20 |

**Beschreibung:** Von der Abschlussseite aus soll man über die dort ganz unten gezeigten Badge-Icons zu den jeweils letzten Bildschirmen der schon abgeschlossenen Spielschritte zurückspringen können — rein zur Ansicht (was wurde gezeigt, was wurde ausgewählt), ohne dass dort etwas verändert werden kann. Von dort führt ein Button zurück zur Abschlussseite.

**User Story:** Als Spielerin/Spieler möchte ich mir nach dem Durchspielen nochmal ansehen können, was ich bei einem bestimmten Schritt gesehen und ausgewählt hatte, sodass ich Details nachvollziehen kann, ohne das Spiel neu spielen zu müssen.

**Scope:**
Eingeschlossen: die Badge-Icons im Bereich ganz unten auf der Abschlussseite (ein Icon pro abgeschlossenem Schritt mit Badge, aktuell 7) werden anklickbar; ein Klick zeigt den letzten Bildschirm dieses Schritts (inkl. Agenten-Reaktion und Debrief-Text) in einer reinen Ansichts-Variante — keine Buttons darin funktionieren. Ein eigener „← Zurück zu den Ergebnissen"-Button führt zurück zur Abschlussseite.
Ausgeschlossen: kein Zugriff auf Schritte ohne Badge (der erzwungene Schnellstart und der Rewind-Übergangsbildschirm haben keins); keine Rückkehr-Möglichkeit mitten im laufenden Spiel (nur von der Abschlussseite aus); keine Bearbeitung oder Neubewertung der früheren Auswahl; keine dauerhafte Speicherung über das Schließen des Browser-Tabs hinaus (bestehende Vorgabe: rein clientseitig, nichts wird gespeichert).

**Akzeptanzkriterien:**
- [x] Auf der Abschlussseite ist an jedem der Badge-Icons erkennbar, dass ein Klick möglich ist (Mauszeiger wird zur Hand, Icon hebt sich beim Überfahren leicht an; zusätzlich Hinweiszeile „Tap a badge to look back at that step“ direkt darüber).
- [x] Ein Klick auf ein Badge-Icon zeigt den Bildschirm, wie er unmittelbar vor dem Weiterklicken zum nächsten Schritt aussah — inklusive der eigenen getroffenen Auswahl und der Rückmeldung des Agenten (automatisiert für alle 7 Schritte geprüft).
- [x] In dieser Ansicht lässt sich nichts anklicken oder verändern (automatisiert geprüft: alle Buttons im gespeicherten Bildschirm haben keine aktive Funktion mehr; zusätzlich per CSS Klicks/Berührungen technisch unterbunden).
- [x] Es ist klar erkennbar, dass man sich in einer reinen Rückblick-Ansicht befindet (Hinweiszeile oben: „🔍 Looking back at: … — view only, nothing here can be changed.“).
- [x] Ein Button führt zuverlässig zurück zur Abschlussseite, die dort unverändert wieder korrekt angezeigt wird (automatisiert geprüft: Abschlussseite vor und nach dem Rückblick byte-identisch).
- [x] Das Verhalten funktioniert für jeden der sieben Badge-Schritte einzeln geprüft.
- [x] Keine Konsolenfehler beim Öffnen und Schließen der Rückblick-Ansicht.
- [ ] Auf schmalem Bildschirm (Handy-Breite) bleibt die Rückblick-Ansicht nutzbar — automatisiert nicht abschließend prüfbar (siehe Testplan), Stephans eigener Blick auf dem Handy steht noch aus.

**Analyse & Planung:**
- [x] Aktuellen Zustand verstanden: Das Spiel rendert pro Schritt den kompletten Bildschirm neu in `#stageHost` (`host.innerHTML`), es gibt bisher keine Historie früherer Bildschirme — nur laufende Summen (`S.analysis`, `S.cycle`, `S.badges`, `S.rework`).
- [x] Betroffene Stellen identifiziert: `go(n)` (Schrittwechsel), `completeStep` (Badge-Vergabe), `renderFinale` (Badge-Icons ganz unten, aktuell nicht anklickbar).
- [x] Implementierungsansatz: `S.history` als neues Array einführen. In `go(n)` wird vor dem Wechsel des Schritts (`S.i=n`) der aktuelle `host.innerHTML` unter dem bisherigen `S.i` abgelegt — das erfasst automatisch für jeden Schritttyp (categorize/build/select/fork/reveal) den exakt letzten gezeigten Zustand, ohne jede Render-Funktion einzeln anzufassen. Beim Badge-Vergeben in `completeStep` wird zusätzlich der Schrittindex mitgespeichert (`S.badges` bekommt ein `i`-Feld), damit ein Icon weiß, welchen History-Eintrag es öffnen soll. Klick auf ein Badge-Icon ersetzt `host.innerHTML` durch den gespeicherten Schnappschuss, umhüllt von einer Hinweiszeile und einem „Zurück"-Button; der Schnappschuss selbst bekommt `pointer-events:none`, damit darin nichts anklickbar ist. Der Zurück-Button ruft einfach erneut `renderFinale()` auf (keine eigene Wiederherstellung nötig, dadurch bleiben alle Handler korrekt verknüpft).
- [x] Risiken benannt: Der allererste `go(0)`-Aufruf beim Start eines neuen Durchlaufs würde kurz einen „falschen" History-Eintrag (den alten Startbildschirm) unter Index 0 ablegen — das wird aber beim Verlassen von Schritt 0 automatisch mit dem echten Inhalt überschrieben, bevor er je sichtbar würde (Schritt 0 hat ohnehin kein Badge und ist über die Abschlussseite nicht erreichbar). Der Zurück-Button löst über `renderFinale()` erneut den Konfetti-Effekt aus — bewusst in Kauf genommen (kein Fehler, eher ein kleines Wiedersehens-Feuerwerk).
- [x] Aufwand: klein bis mittel, reine Frontend-Logik in `public/index.html`.

**Testplan:**
- [x] Automatisiert (jsdom, echtes Ausführen der App-Logik im echten DOM, kein optischer Screenshot-Vergleich): drei vollständige Durchläufe — Maus-Modus mit diszipliniertem Weg, Touch-Modus (Antippen statt Ziehen) mit diszipliniertem Weg, Maus-Modus mit genommener Abkürzung an einem Versuchungsmoment. In jedem Durchlauf wurden alle 7 Badges einzeln angeklickt, der jeweils korrekte gespeicherte Bildschirm inkl. eigener Auswahl und Debrief geprüft, die technische Wirkungslosigkeit aller darin enthaltenen Buttons geprüft, die Rückkehr zur unveränderten Abschlussseite geprüft (Byte-Vergleich vorher/nachher) und auf JS-Fehler geprüft. Ergebnis: alle 3 Durchläufe × alle Prüfungen fehlerfrei bestanden (0 Fails, 0 JS-Fehler).
- [x] Syntax-Check des eingebetteten Skripts (`node --check`) fehlerfrei.
- [ ] Echter visueller Blick im Browser (Desktop + Handy-Breite) steht noch aus — der Chrome-Zugriff dieser Session kann `file://`-Seiten aus Sicherheitsgründen nicht steuern (Erweiterung hat dafür keine Berechtigung), und ein lokaler Server auf Stephans Mac lässt sich aus dieser Sitzung heraus nicht starten. Ein lokaler Python-Http-Server in meiner eigenen Cloud-Sandbox wäre für Stephans echten Browser nicht erreichbar (anderes Netzwerk). Deshalb bittet dieses Ticket Stephan um einen kurzen eigenen Blick vor dem Release.
- [ ] Bestehende Tests aktualisiert: keine vorhandene automatisierte Testsuite im Projekt gefunden — die jsdom-Prüfung oben ist neu und deckt zusätzlich den kompletten bisherigen Spielablauf (alle Schritttypen, Touch- und Maus-Modus, beide Versuchungs-Ausgänge) ab, nicht nur das neue Feature.

**Scope-Änderungen** *(chronologisches Log):*
- 2026-07-20: Versionsnummer von 1.4.2 auf **1.5.0** angehoben (Minor statt Patch, da eine echte neue, sichtbare Spielfunktion und nicht nur eine Textänderung) — analog zum Vorgehen bei FEATURE-001/003.

**Implementierungsnotizen:**
Umgesetzt in `public/index.html` (lokaler Git-Klon `~/Claude/Projects/Agentic Engineering Gamification/agent-contract-game`, Basis: v1.4.2 / Commit `151cb67`, vor dem Schreiben per `git log`/`git status` verifiziert — kein zwischenzeitlicher unabhängiger Stand). Neues `S.history`-Array wird generisch in `go(n)` befüllt (Schnappschuss des kompletten `host.innerHTML` des verlassenen Schritts, bevor der Index wechselt) — deckt automatisch alle Schritttypen ab, ohne jede Render-Funktion einzeln anfassen zu müssen. `S.badges`-Einträge tragen jetzt zusätzlich den Schrittindex (`i`). Auf der Abschlussseite sind die Badge-Icons jetzt anklickbar (`data-i`-Attribut, Cursor/Hover-Stil) und öffnen über `openReview(i)` den gespeicherten Bildschirm, umhüllt von einer Hinweiszeile und einem „← Back to results“-Button; der gespeicherte Bereich bekommt die CSS-Klasse `.reviewMode` mit `pointer-events:none`, wodurch technisch nichts darin anklickbar ist (zusätzlich zur Tatsache, dass beim Neuaufbau aus dem HTML-String ohnehin keine JS-Klick-Handler mehr existieren). Der Zurück-Button ruft einfach erneut `renderFinale()` auf statt den alten Zustand händisch wiederherzustellen — dadurch bleiben alle Handler korrekt verknüpft, als kleiner Nebeneffekt löst das erneut den Konfetti-Effekt aus (bewusst in Kauf genommen).

Der „For facilitators“-Button auf dem Startbildschirm wurde entgegen der ursprünglichen Vermutung NICHT verändert: eine Prüfung im Code ergab, dass er bereits funktioniert und einen echten Moderations-Hinweiskasten aufklappt — Stephan hat das nach kurzer Rückfrage bestätigt und möchte ihn unverändert behalten.

**Bestätigt:** Stephan hat den GitHub-Actions-Lauf (grün, `86886dc`) und die Live-Seite selbst geprüft — Badge-Klick und „Back to results" funktionieren wie vorgesehen. Direkt im Anschluss kam der Wunsch auf, den Rückblick auch während des laufenden Spiels verfügbar zu machen — das ist bewusst nicht Teil dieses Tickets (das war von Anfang an auf die Abschlussseite begrenzt), sondern wird als eigenes Ticket FEATURE-005 weitergeführt.

---
