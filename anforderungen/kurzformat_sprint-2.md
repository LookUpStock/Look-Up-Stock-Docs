# Kurzformat-Tabellen – Sprint 2

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #56 Kurzformat-Tabelle für US-01; Docs #72 PO-Festlegungen zum Simulationsmodell; Docs #58 Anpassung an die umgesetzte Engine |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 3 |
| **Stand** | 11.10.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 2 (Abschnitt 4), API-Design (Stand 02.10.2026, Abschnitt „Auth“), NFRs nach FURPS+ (NF-03, NF-04, NF-16), Akteursbeschreibung v1, Use-Case-Diagramm v1, Glossar; für E15 bis E23 zusätzlich Kurzformat Sprint 1 (US-04, E8), Domänenmodell v3, ADR-02, ADR-03, NF-01, NF-08, NF-14 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt `kurzformat_sprint-1.md` fort. Die Festlegungen E15 bis E23 legen das Rechenmodell der Simulation (US-04) fest; die genauen Rechenregeln dazu stehen in `simulationsmodell.md`. Die Tabellen aus Sprint 1 bleiben als Stand von Sprint 1 unverändert; wie sich US-02 bis US-04 durch die Anmeldung ändern, steht im Abschnitt „Auswirkungen von US-01 auf US-02 bis US-04“.

US-01 wird in zwei Tabellen beschrieben, weil die Story zwei Anwendungsfälle mit unterschiedlichen primären Akteuren umfasst. So ist es auch im Use-Case-Diagramm dargestellt: „Registrieren“ löst der Gast aus, „An- und abmelden“ der registrierte Nutzer.

---

## US-01 (Teil 1) Registrieren

| **Name** | **Registrieren** |
|---|---|
| **Art** | User Story (US-01), Teil 1 von 2 |
| **Kurzbeschreibung** | Ein Gast legt mit seiner E-Mail-Adresse und einem Passwort ein Nutzerkonto an, damit er anschließend eigene Portfolios anlegen, simulieren und speichern kann. |
| **Auslöser** | Der Gast möchte eigene Portfolios anlegen und speichern. |
| **Akteure** | Gast (primär) |
| **Eingehende Information** | E-Mail-Adresse (gültiges Format), Passwort (mindestens 8 Zeichen) |
| **Vorbedingungen** | Die Anwendung ist im Browser geöffnet; der Gast ist nicht angemeldet. |
| **Nachbedingungen** | • Für die E-Mail-Adresse ist ein Nutzerkonto angelegt.<br>• Das Passwort ist nur als Hash gespeichert, nie im Klartext (NF-03).<br>• Der Gast kann sich mit E-Mail-Adresse und Passwort anmelden (Teil 2); er ist nach der Registrierung noch nicht angemeldet (E13).<br>• Ist die E-Mail-Adresse bereits registriert, wird kein weiteres Konto angelegt und der Hinweis „Diese E-Mail ist bereits registriert.“ erscheint. |
| **Essenzielle Schritte** | • E-Mail-Adresse und Passwort eingeben<br>• Eingaben prüfen<br>• Nutzerkonto anlegen<br>• Registrierung bestätigen |

---

## US-01 (Teil 2) An- und abmelden

| **Name** | **An- und abmelden** |
|---|---|
| **Art** | User Story (US-01), Teil 2 von 2 |
| **Kurzbeschreibung** | Ein registrierter Nutzer meldet sich mit E-Mail-Adresse und Passwort an, um mit seinen eigenen Portfolios zu arbeiten, und meldet sich danach wieder ab. |
| **Auslöser** | Der Nutzer möchte auf seine Portfolios zugreifen oder seine Anmeldung beenden. |
| **Akteure** | Registrierter Nutzer (primär) |
| **Eingehende Information** | Anmelden: E-Mail-Adresse und Passwort. Abmelden: keine Eingabe. |
| **Vorbedingungen** | Für den Nutzer besteht ein Nutzerkonto (Teil 1). Zum Abmelden ist der Nutzer angemeldet. |
| **Nachbedingungen** | • Nach erfolgreicher Anmeldung ist der Nutzer 24 Stunden angemeldet (NF-16) und sieht ausschließlich seine eigenen Portfolios (NF-04).<br>• Bei falschen Anmeldedaten erscheint „E-Mail oder Passwort ist falsch.“; der Hinweis verrät nicht, ob die E-Mail-Adresse registriert ist.<br>• Nach dem Abmelden oder nach Ablauf der 24 Stunden ist die Anmeldung ungültig; geschützte Funktionen verlangen eine neue Anmeldung (E14). |
| **Essenzielle Schritte** | • E-Mail-Adresse und Passwort eingeben<br>• Anmeldedaten prüfen<br>• Anmeldung bestätigen und eigene Portfolios bereitstellen<br>• Abmelden |

---

## Auswirkungen von US-01 auf US-02 bis US-04

Laut Sprint-Planning-Protokoll Sprint 2 gilt ab diesem Sprint die Vorbedingung „Nutzer ist angemeldet“, soweit der Endpunkt laut API-Design eine Anmeldung erfordert.

| Story | Feld | Sprint 1 | Ab Sprint 2 | Grundlage |
|---|---|---|---|---|
| US-02 Anlagen suchen | – | Ohne Anmeldung nutzbar | **Unverändert.** Die Suche bleibt auch für Gäste ohne Anmeldung nutzbar. | API-Design: öffentliche Abfragen bleiben GET; Use-Case-Diagramm v1: Gast – Anlagen suchen |
| US-03 Portfolio erstellen | Akteure | Nutzer | Registrierter Nutzer | Akteursbeschreibung v1 |
| | Vorbedingungen | Anlagen wurden über die Anlagensuche gefunden. | Zusätzlich: Der Nutzer ist angemeldet. | Sprint-Planning-Protokoll Sprint 2, Abschnitt 4 |
| | Nachbedingungen | Portfolio ist gespeichert. | Zusätzlich: Das Portfolio ist dem angemeldeten Nutzer zugeordnet und nur für ihn sichtbar. | NF-04; in Sprint 1 bewusst gestrichen, da es noch keine Anmeldung gab |
| US-04 Simulation durchführen | Akteure | Nutzer (primär), Twelve Data (sekundär) | Registrierter Nutzer (primär), Twelve Data (sekundär) | Akteursbeschreibung v1 |
| | Vorbedingungen | Es wurde mindestens ein Portfolio erstellt. | Der Nutzer ist angemeldet und hat mindestens ein eigenes Portfolio erstellt. | Sprint-Planning-Protokoll Sprint 2, Abschnitt 4 |
| | Nachbedingungen | – | Zusätzlich: Simuliert werden kann nur ein eigenes Portfolio; ein fremdes Portfolio wird wie ein nicht vorhandenes behandelt. | NF-04 (fremde Ressourcen liefern 404) |

---

## Festlegungen des Product Owners (07.10.2026)

Anlass: Abgleich der Akzeptanzkriterien von US-01 mit dem API-Design. Die Nummerierung setzt E1 bis E10 aus Sprint 1 fort.

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| E11 | US-01 | Die E-Mail-Adresse wird ohne Unterscheidung von Groß- und Kleinschreibung verglichen (`Anna@example.com` und `anna@example.com` sind dieselbe Adresse). | Verhindert zwei Konten für dieselbe Adresse und eine scheiternde Anmeldung wegen der Schreibweise. |
| E12 | US-01 | Eine E-Mail-Adresse ohne gültiges Format wird mit dem Hinweis „Bitte gib eine gültige E-Mail-Adresse ein.“ abgelehnt. | Das API-Design legt bisher nur den Hinweis zum Passwort fest; für Oberfläche und Abnahmetest wird ein einheitlicher Text benötigt. |
| E13 | US-01 | Nach der Registrierung ist der Nutzer noch nicht angemeldet; er meldet sich anschließend mit seinen Daten an. | `POST /auth/register` liefert laut API-Design keinen Token. |
| E14 | US-01 | Ist die Anmeldung abgelaufen, wird der Nutzer mit dem Hinweis „Deine Anmeldung ist abgelaufen. Bitte melde dich erneut an.“ zur Anmeldung geführt. Ist sie aus einem anderen Grund ungültig (z. B. nach dem Abmelden), lautet der Hinweis „Bitte melde dich an.“ | Der Nutzer versteht, warum er sich erneut anmelden muss. Das Frontend erkennt den Ablauf selbst über `expiresAt` aus der Login-Antwort; das Backend bleibt unverändert (401 mit „Bitte melde dich an.“). |

---

## Festlegungen des Product Owners (08.10.2026) – Simulationsmodell

Anlass: Rückfrage aus der Umsetzung der Monte-Carlo-Engine (BE #40). Die Kurzformat-Tabelle US-04, das Domänenmodell und das API-Design legen fest, *was* die Simulation liefert, aber nicht, *wie* gerechnet wird. Ohne diese Festlegungen müsste der Entwickler fachliche Annahmen selbst treffen. Die genauen Rechenregeln stehen in `simulationsmodell.md`.

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| E15 | US-04 | Die Gewichtungen werden in der Simulation **jeden Monat auf die festgelegten Werte zurückgesetzt** (monatliches Rebalancing). Die Sparrate wird nach denselben Gewichtungen aufgeteilt. | Die vom Nutzer eingegebene Gewichtung gilt über den gesamten Anlagehorizont. Einfache, nachvollziehbare Rechnung und kurze Rechenzeit (NF-01). |
| E16 | US-02, US-04 | Renditen werden in der **Handelswährung** der Anlage verwendet und nicht in Euro umgerechnet. Startkapital, Sparrate und Ergebnis werden in Euro angegeben. Unter dem Ergebnis steht der Hinweis „Wechselkursschwankungen werden nicht berücksichtigt.“ | Wechselkurse würden zusätzliche Abrufe bei der Finanzdaten-API und einen weiteren Cache erfordern. Bei Anlagen in Euro (z. B. EUNL) ist das Währungsrisiko bereits im Kurs enthalten. |
| E17 | US-04 | Der **Schätzzeitraum** umfasst die letzten bis zu 120 Monatswerte (10 Jahre). Er gilt gemeinsam für alle Anlagen und ist so lang wie die kürzeste vorhandene Kurshistorie, höchstens 120 Monatswerte. Mindestens 36 Monatswerte je Anlage bleiben Pflicht (E8). | Längere Zeiträume ergeben stabilere Schätzungen; jüngere Anlagen ab 36 Monatswerten bleiben simulierbar. Ein gemeinsamer Zeitraum macht die Anlagen vergleichbar (Domänenmodell v3). |
| E18 | US-04 | *(geändert am 11.10.2026)* Der **Zusammenhang zwischen den Anlagen** (Korrelation) wird **nicht** berücksichtigt: Rendite und Volatilität werden je Anlage geschätzt, die Monatsrenditen der Anlagen werden unabhängig voneinander simuliert und mit den Gewichtungen zur Monatsrendite des Portfolios zusammengefasst. | Angleichung an die umgesetzte Monte-Carlo-Engine (BE #40). Einfaches, je Anlage nachvollziehbares Modell, das mit den angezeigten Schätzungen je Anlage übereinstimmt. Bekannte Vereinfachung: Bei Anlagen, die sich meist gleichläufig bewegen, wird die Schwankung des Portfolios unterschätzt, bei gegenläufigen überschätzt (siehe `simulationsmodell.md`, Abschnitt 7). Ursprüngliche Fassung (08.10.2026): Korrelation über die historischen Monatsrenditen des Portfolios. |
| E19 | US-04 | Die **Sparrate** wird am Ende jedes Monats eingezahlt, erstmals in Monat 1. Über den Anlagehorizont sind das Anlagehorizont × 12 Raten. Monat 0 enthält nur das Startkapital. | Eindeutige Regel für Berechnung und Test; entspricht dem Beispiel im API-Design (10.000 € + 240 × 200 € = 58.000 € Einzahlungen). |
| E20 | US-04 | Die Kurshistorie wird als **Gesamtrendite** verwendet: Kurse sind um Splits und Dividenden bereinigt, Dividenden gelten als wiederangelegt. | Sonst würden ausschüttende Anlagen schlechter dargestellt als thesaurierende mit gleicher Entwicklung. |
| E21 | US-04 | Die Simulation rechnet **nominal**, ohne Kosten, Steuern und Inflation. Unter dem Ergebnis steht der Hinweis „Die Werte sind nominal und berücksichtigen keine Kosten, Steuern und Inflation. Keine Anlageberatung.“ | Hält das Modell einfach; der Hinweis grenzt die Anwendung von einer Anlageberatung ab (NF-14). |
| E22 | US-04 | Es werden nur **abgeschlossene Monate** verwendet; der laufende Monat zählt nicht zur Kurshistorie. | Ein unvollständiger Monat würde die Schätzung verfälschen. Entspricht dem Beispiel im API-Design (Historie bis 31.08. bei Abruf im September). |
| E23 | US-04 | *(geändert am 11.10.2026)* Die Monatsrenditen der Simulation sind **normalverteilt** (Mittelwert und Standardabweichung aus der Schätzung je Anlage). Fällt der Portfoliowert rechnerisch unter 0, wird er auf 0 gesetzt; danach wird die Sparrate eingezahlt. | Angleichung an die umgesetzte Monte-Carlo-Engine (BE #40). Einfach zu erklären und zu testen; die Untergrenze verhindert negative Portfoliowerte. Ursprüngliche Fassung (08.10.2026): lognormalverteilte Monatsrenditen. |

### Präzisierung der Akzeptanzkriterien US-04

Ergänzt die Kriterien aus `kurzformat_sprint-1.md`; diese bleiben unverändert gültig.

- Unter dem Ergebnis erscheinen die Hinweise aus E16 und E21. *(E16, E21)*
- Die Summe der Einzahlungen beträgt Startkapital + Anlagehorizont × 12 × Sparrate. *(E19)*
- Der Verlaufspunkt für Monat 0 hat in allen drei Perzentilen den Wert des Startkapitals. *(E19)*
- Die angezeigten Schätzungen (Rendite und Volatilität je Anlage) und der Schätzzeitraum stammen aus demselben gemeinsamen Zeitraum von höchstens 120 Monatswerten. *(E17)*

---

## Akzeptanzkriterien

Zur Übernahme in das Story-Issue US-01 (Docs #9). Grundlage sind die Kriterien aus dem Sprint-Planning-Protokoll Sprint 2; Präzisierungen durch die Festlegungen E11 bis E14 sind gekennzeichnet.

**US-01 Registrieren und anmelden**

- Die Registrierung erfolgt mit E-Mail-Adresse und Passwort. Ein Passwort mit weniger als 8 Zeichen wird mit „Das Passwort muss mindestens 8 Zeichen haben.“ abgelehnt.
- Eine E-Mail-Adresse ohne gültiges Format wird abgelehnt. *(E12)*
- Eine bereits registrierte E-Mail-Adresse wird mit dem Hinweis „Diese E-Mail ist bereits registriert.“ abgelehnt; Groß- und Kleinschreibung spielen dabei keine Rolle. *(E11)*
- Passwörter werden nur gehasht gespeichert (NF-03).
- Nach der Registrierung meldet sich der Nutzer mit seinen Daten an. *(E13)*
- Falsche Anmeldedaten führen zur Meldung „E-Mail oder Passwort ist falsch.“, ohne zu verraten, ob die E-Mail-Adresse existiert.
- Eine Anmeldung ist 24 Stunden gültig (NF-16).
- Ein Nutzer sieht nur seine eigenen Portfolios; der Zugriff auf fremde Portfolios liefert 404 (NF-04).
- Ein angemeldeter Nutzer kann sich abmelden; die Anmeldung ist danach ungültig.
- Ist die Anmeldung abgelaufen, wird der Nutzer mit „Deine Anmeldung ist abgelaufen. Bitte melde dich erneut an.“ zur Anmeldung geführt; bei sonst ungültiger Anmeldung mit „Bitte melde dich an.“ *(E14)*

---

## Änderungshistorie

| Version | Datum | Story | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 07.10.2026 | US-01 | Neu: Kurzformat-Tabellen „Registrieren“ und „An- und abmelden“, Akzeptanzkriterien | Docs #56, Sprint-Planning-Protokoll Sprint 2 |
| 1 | 07.10.2026 | US-03, US-04 | Vorbedingung „Nutzer ist angemeldet“, Akteur „Registrierter Nutzer“, Zuordnung der Portfolios zum Nutzer | US-01, NF-04 |
| 1 | 07.10.2026 | US-02 | Ausdrücklich ohne Anmeldung nutzbar | API-Design (öffentliche GET-Abfrage) |
| 1 | 07.10.2026 | US-01 | Festlegungen E11 bis E14 | Abgleich mit dem API-Design |
| 2 | 08.10.2026 | US-04 | Festlegungen E15 bis E23 zum Simulationsmodell (Rebalancing, Währung, Schätzzeitraum, Korrelation, Sparrate, Gesamtrendite, nominale Rechnung, abgeschlossene Monate, Verteilungsmodell); Präzisierung der Akzeptanzkriterien US-04; Verweis auf `simulationsmodell.md` | Rückfrage aus BE #40 (Monte-Carlo-Engine), Docs #72 |
| 3 | 11.10.2026 | US-04 | E18 (keine Korrelation) und E23 (normalverteilte Monatsrenditen mit Untergrenze 0) an die umgesetzte Monte-Carlo-Engine angepasst | PO-Entscheidung vom 11.10.2026 nach Abgleich mit BE #40 und BE #49 |
