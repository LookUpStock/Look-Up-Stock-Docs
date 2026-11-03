# Abnahmeprotokoll – Sprint 5

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 5 (26.10.2026 – 01.11.2026), letzter Sprint |
| **Issue** | Docs #170 Abnahmetests Sprint 5, Abnahmeprotokoll Sprint 5 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Datum der Abnahme** | 03.11.2026, 09:20 bis 11:45 Uhr |
| **Stories** | Neu aus dem Change Request CR-01: US-11 Nutzungskennzahlen anzeigen (Docs #156), US-12 Nutzungskennzahlen exportieren (Docs #157). Technische Stories: ADR-07 getrennte Dienste und Queue (Docs #158), GraphQL schreibend (Docs #159), SPA-Touchpoint nach ADR-08 (Docs #160). Nachtest der Hinweise H2 bis H7 aus Sprint 4, Übernahme NF-09, Wiederholung der Messung NF-01 und E33 |
| **Grundlagen** | [Abnahmetestfälle Sprint 5](abnahmetestfaelle-sprint-5.md), [Abnahmeprotokoll Sprint 4](abnahmeprotokoll-sprint-4.md), [Bedienbarkeitstest NF-09](bedienbarkeitstest-nf-09.md), [Testkonzept](testkonzept.md) Version 5, `anforderungen/kurzformat_sprint-4.md` (US-11, US-12, E36 bis E43), `anforderungen/change-request.md` Version 2, ADR-07, ADR-08, API-Design (Stand 01.11.2026) |
| **KI-Nutzung** | Durchführung mit KI-Unterstützung vorbereitet und ausgewertet, Tests vom Product Owner selbst durchgeführt, Datenbankabfragen und Heroku-Prüfungen von Carl Stibbe, Entscheidungen vom Product Owner getroffen, siehe KI-Log |

Die Status der einzelnen Testfälle stehen in der Spalte „Status“ von `abnahmetestfaelle-sprint-5.md`, die Ergebnisse des Nachtests in derselben Datei in der Spalte „Ergebnis“ des Abschnitts „Nachtest der Hinweise aus Sprint 4“. Dieses Protokoll fasst das Ergebnis je Story zusammen, begründet die Abnahmeentscheidung und listet die gefundenen Fehler und Hinweise.

---

## 1. Testumgebung und Durchführung

| | |
|---|---|
| **Frontend** | `https://lookupstockfrontend-96a9998805cb.herokuapp.com`, Stand `main` a839c19 (03.11.2026, 09:03, Merge PR #66 zu FE #51 Dashboard) |
| **core-service** | `https://lookupstock-da2951e42089.herokuapp.com` (Heroku-App `lookupstock`), Stand `main` 5a32df2 (01.11.2026, 16:00, Merge PR #127 zu BE #113) |
| **marktdaten-service** | `https://lookupstock-marktdaten-12ca5b75c510.herokuapp.com` (Heroku-App `lookupstock-marktdaten`), Stand des Ordners `marktdaten-service` seit `main` 3d7953f (01.11.2026, 15:35, Merge PR #125); spätere Commits ändern den Ordner nicht |
| **Datenbank** | Heroku Postgres, Schemas `public` (core-service) und `marktdaten` (marktdaten-service) |
| **Finanzdaten** | echte Twelve-Data-API, nur über den marktdaten-service; am 03.11.2026 um etwa 11:05 Uhr mit neuem API-Key (siehe B10) |
| **Testnutzer** | `abnahme3.a@example.com` (Nutzer A) und `abnahme3.b@example.com` (Nutzer B) aus Sprint 3; `gf@example.com` (Nutzer M, Geschäftsführung), am 03.11.2026 registriert, Rolle `MANAGER` um 10:22 Uhr von Carl Stibbe in der Datenbank gesetzt; für AT-11-03 neu registriert: `zaehltest-0311@example.com` |
| **Testdaten** | Portfolios von Nutzer A aus Sprint 3 und 4: „Mein Weltportfolio“ (id 34), „Mein Aktienportfolio“, „Krypto-Test“, „Messportfolio“; gespeicherte Simulation „Sparplan 20 Jahre“. Nutzer B hat keine Daten. Während der Abnahme angelegt: „GraphQL-Test“ (id 133, wieder gelöscht), „Zähltest“ mit „Zähltest Simulation“ (Nutzer `zaehltest-0311`), „Vorladen“ (Nutzer A, VT 100 %) |
| **Manuelle Tests** | Product Owner im Browser (Brave, Chromium) im normalen und im privaten Fenster; API- und GraphQL-Prüfungen sowie Messungen mit `fetch` in der Konsole der Entwicklertools; Netzwerk-Tab mit Spalte „Domain“; Handyansicht mit der Geräteansicht bei 360 px; CSV-Datei in Windows-Editor und Excel (deutsche Spracheinstellung). Datenbankabfragen in der Postgres-Konsole und Prüfungen im Heroku-Dashboard durch Carl Stibbe, Nachweis per Screenshot |
| **Automatisierte Tests** | „Backend CI (core-service)“ Lauf #94 auf `main` 5a32df2 erfolgreich (01.11.2026, 16:00): 307 Tests, 0 Fehler, 0 übersprungen; die JaCoCo-Prüfung der Kernlogik (Zeilenabdeckung mindestens 80 %, NF-11) ist bestanden („All coverage checks have been met.“). „Backend CI (marktdaten-service)“ Lauf #6 auf `main` 3d7953f erfolgreich (01.11.2026, 15:35): 33 Tests, 0 Fehler. „Frontend CI“ Lauf #68 auf `main` a839c19 erfolgreich (03.11.2026, 09:03): 160 Tests, 0 Fehler. Alle in den Testfällen genannten Testmethoden sind im Code vorhanden; kein Test ist mit `@Disabled` abgeschaltet |

**Bewertungsregel.** Wie in Sprint 3 und 4 (Testkonzept, Abschnitt 8): Ein manueller Testfall ist `bestanden`, wenn Oberfläche, API bzw. Datenbank das erwartete Ergebnis zeigen; ein automatisierter, wenn die zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren. `fehlgeschlagen` heißt, mindestens ein erwartetes Ergebnis trat nicht ein. Priorität der Fehler nach Testkonzept, Abschnitt 11.

**Zeitpunkt.** Die Abnahme fand zwei Tage nach dem Ende des Sprints statt, weil das Dashboard (FE #51) erst am 03.11.2026 um 09:03 Uhr auf `main` lag.

---

## 2. Ergebnis je Story

| Story | Testfälle | bestanden | fehlgeschlagen | offen | Entscheidung |
|---|---|---|---|---|---|
| US-11 Nutzungskennzahlen anzeigen | 7 | 5 | 2 | 0 | **nicht abgenommen** (B8, B9) |
| US-12 Nutzungskennzahlen exportieren | 3 | 3 | 0 | 0 | **abgenommen** |
| ADR-07 getrennte Dienste und Queue | 8 | 8 | 0 | 0 | umgesetzt (B10 während der Abnahme behoben) |
| GraphQL schreibend | 5 | 5 | 0 | 0 | umgesetzt |
| SPA-Touchpoint (ADR-08) | 6 | 6 | 0 | 0 | umgesetzt |
| NF-10 neue Seiten | 1 | 1 | 0 | 0 | – |
| Nachtest H2 bis H7, NF-09, NF-01 | 8 | 7 | 1 | 0 | H2 bis H5 und H7 behoben, H6 nicht behoben (H8) |
| **Gesamt** | 38 | 35 | 3 | 0 | |

---

## 3. Begründung

### US-11 Nutzungskennzahlen anzeigen – nicht abgenommen

Fünf von sieben Testfällen sind bestanden. Zwei Akzeptanzkriterien sind nicht erfüllt; nach Testkonzept, Abschnitt 11, sind das Fehler mit Priorität **hoch**, die eine Abnahme verhindern.

- **Dashboard (AT-11-01, bestanden):** M sieht in der Navigation „Auswertungen“. Das Dashboard zeigt ohne Auswahl „November 2026 (vorläufig)“, „Erfassung seit 01.11.2026“ und die neun Kennzahlen in der Reihenfolge aus E39. Die Sitzung enthält `role` „MANAGER“; `GET /reports/usage` liefert `200`, `month` „2026-11“, `preliminary` `true`, `trackingSince` „2026-11-01“ und 9 Einträge mit denselben Werten wie das Dashboard.
- **Zählung (AT-11-03, fehlgeschlagen, B8):** Ausgangsstand 1 | 6 | 2 | 28 | 2 | 27 | 0 | 3 | 13, danach 2 | 7 | 3 | 30 | 3 | 28 | 2 | 5 | 14. Registrierungen, Anmeldungen, Portfolios, gespeicherte Simulationen, Vergleiche und Nutzerkonten stiegen um 1, die Anlagensuchen um 2 (zwei erfolgreiche Suchanfragen im Netzwerk-Tab), die Beispielsimulationen um 2 (die Beispielsimulation wurde zweimal gestartet, zwei Ergebnisse mit den Seeds 121248 und 995480). Der Vergleich hat die Simulationen nicht erhöht. „Durchgeführte Simulationen“ stieg aber um **2 statt 1**. Ursache: Das Frontend ruft zum Speichern erneut `POST /simulations` mit `save: true` auf (`saved-simulations.js`, `api.saveSimulation`); das Backend rechnet neu und zählt eine zweite Simulation. Die Datenbank bestätigt das: `SIMULATION_RUN` um 09:36:06 (Simulieren), `SIMULATION_RUN` um 09:36:27 und `SIMULATION_SAVED` um 09:36:28 UTC (Speichern). Damit ist das Akzeptanzkriterium „Nach einer erfolgreichen Simulation mit Speichern steigen ‚durchgeführte Simulationen‘ und ‚gespeicherte Simulationen‘ jeweils um 1.“ nicht erfüllt. Die Übertragung über die Queue funktioniert: Alle Werte lagen nach 10 Sekunden vor, keine Aktion war spürbar verzögert.
- **Fehlerfälle zählen nicht (AT-11-04, bestanden):** Falsches Passwort („E-Mail oder Passwort ist falsch.“), Registrierung einer vorhandenen E-Mail („Diese E-Mail ist bereits registriert.“), „Krypto-Test“ simulieren und gegen „Mein Weltportfolio“ vergleichen (je `422`, ETHA genau einmal) und Speichern ohne Namen (Meldung im Editor, keine Anfrage): Nur „Anmeldungen“ stieg um 1 (7 → 8), die erfolgreiche Anmeldung von A.
- **Monate ohne Auswertung (AT-11-05, bestanden):** Oktober 2026 zeigt „Für diesen Monat liegen keine Auswertungen vor.“ ohne Werte und ohne Nullen; Dezember 2026 ist in der Monatsauswahl ausgegraut. `?month=2026-12` und `?month=2026-10` liefern `400 VALIDATION_ERROR` mit dieser Meldung, `?month=11-2026` „Bitte gib den Monat im Format JJJJ-MM an.“ mit `details` zu `month`.
- **Zugriffsschutz (AT-11-06, fehlgeschlagen, B9):** Die API ist korrekt geschützt: Mit dem Token von A `403 FORBIDDEN` „Dafür fehlt die Berechtigung.“, ohne Header `401 UNAUTHORIZED` „Bitte melde dich an.“, jeweils als JSON. Auf den MPA-Seiten sieht A keinen Eintrag „Auswertungen“; `dashboard.html` zeigt A „Dafür fehlt die Berechtigung.“ ohne Kennzahlen; nach dem Abmelden leitet die Seite zur Anmeldung weiter. Auf `dashboard.html` selbst steht „Auswertungen“ aber fest im HTML und ist auch für A sichtbar (für M erscheint der Eintrag dort doppelt). Damit ist das Akzeptanzkriterium „… sieht den Navigationseintrag ‚Auswertungen‘ nicht“ auf dieser Seite nicht erfüllt. Die Daten bleiben geschützt; die Prüfung im Backend (NF-04) ist bestanden.
- **Anonyme Erfassung (AT-11-07, bestanden):** `usage_event` hat nur die Spalten `id`, `event_type` und `occurred_at`, einen Primärschlüssel und einen Index auf `occurred_at`, keinen Fremdschlüssel. Die neuesten 15 Zeilen nach AT-11-03 und AT-11-04 enthalten genau die Ereignisarten des Zähltests (`REGISTRATION`, `LOGIN`, `ASSET_SEARCH`, `PORTFOLIO_CREATED`, `SIMULATION_RUN`, `SIMULATION_SAVED`, `COMPARISON_RUN`, 2 × `DEMO_SIMULATION_RUN`) und die eine Anmeldung aus AT-11-04, ohne Bezug zu Nutzer, Suchbegriff oder Anlage. Die abgelehnten Aufrufe aus AT-11-04 und AT-11-06 haben keine Ereignisse erzeugt.
- **Monatsgrenzen (AT-11-02, bestanden):** automatisiert (`UsageReportServiceTest`).

### US-12 Nutzungskennzahlen exportieren – abgenommen

Alle Testfälle sind bestanden. Für November 2026 bietet der Browser die Datei `lookupstock-nutzung-2026-11` als CSV an. Im Editor stehen `Monat;Kennzahl;Wert` und neun Zeilen `2026-11;<Bezeichnung>;<Wert>` mit denselben Werten wie das Dashboard (AT-12-01). Die API liefert `200`, `text/csv;charset=UTF-8`, `Content-Disposition: attachment; filename="lookupstock-nutzung-2026-11.csv"`; die Datei beginnt mit dem BOM (Bytes 239 187 191). Per Doppelklick öffnet Excel die Datei ohne Importdialog in drei Spalten; „Durchgeführte …“ ist mit korrektem „ü“ dargestellt (AT-12-02). Der Export mit dem Token von A liefert `403`, ohne Header `401`, als M für Oktober `400` „Für diesen Monat liegen keine Auswertungen vor.“, jeweils als JSON und ohne Datei; in der Oberfläche erscheint für Oktober dieselbe Meldung unter dem Knopf, ein Download startet nicht (AT-12-03).

Der Fehler B8 betrifft den Export nur insofern, als die Datei denselben überhöhten Wert enthält wie das Dashboard. Das Akzeptanzkriterium von US-12 (dieselben Werte wie das Dashboard) ist erfüllt.

### ADR-07 getrennte Dienste und Queue – umgesetzt

- **Zwei Dienste (AT-ADR7-01):** Im Heroku-Dashboard sind die Apps `lookupstock`, `lookupstock-marktdaten` und `lookupstockfrontend` vorhanden. `GET /actuator/health` des marktdaten-service liefert `{"groups":["liveness","readiness"],"status":"UP"}` ohne Schlüssel. Im Netzwerk-Tab mit der Spalte „Domain“ gehen alle Anfragen der App (`ping`, `portfolios`, `simulations`, `comparisons`, `assets/search`) an `lookupstock-da2951e42089.herokuapp.com`; keine Anfrage geht an den marktdaten-service oder an Twelve Data. Die einzige andere Domain stammt von einer Browser-Erweiterung (Initiator `content.js`).
- **Schlüssel (AT-ADR7-02):** `GET /internal/search?q=msci` ohne Header liefert `UNAUTHORIZED` „Interner Schlüssel fehlt oder ist ungültig.“ ohne Treffer. Die Konfigurationsvariablen (nur Namen geprüft): `TWELVE_DATA_API_KEY` nur beim marktdaten-service, `INTERNAL_API_KEY` bei beiden Diensten, `MARKTDATEN_URL` nur beim core-service. Im aktuellen Stand des Repositorys stehen nur Platzhalter (`.env.example`) und `ci-dummy-key` (Workflows). In der Git-Historie stand aber der echte Twelve-Data-Key (B10); er wurde während der Abnahme ersetzt. Bestanden nach Behebung.
- **Kurse vorladen (AT-ADR7-06):** Vor dem Test lag für VT nichts im Cache (`0 rows`). A speicherte um 11:36 Uhr das Portfolio „Vorladen“ (VT 100 %) ohne Verzögerung und ohne zu simulieren. Danach lagen fünf Monatswerte von VT mit `fetched_at` 10:36:12 UTC (11:36:12 Uhr) im Cache. Weil das ein frischer Abruf bei Twelve Data war, belegt der Test auch, dass der neue API-Key funktioniert.
- **Queue für Nutzungsereignisse:** manuell über AT-11-03 und AT-11-07 belegt (Werte nach 10 Sekunden im Dashboard, Ereignisse in `usage_event`).
- **Automatisiert:** AT-ADR7-03, AT-ADR7-04, AT-ADR7-05 und AT-ADR7-07 mit den genannten Testklassen; AT-ADR7-08: beide Workflows grün ohne echten Twelve-Data-Key, JaCoCo-Prüfung der Kernlogik bestanden, Pfadfilter `core-service/**` bzw. `marktdaten-service/**` (der Merge von PR #127 löste nur „Backend CI (core-service)“ aus).

### GraphQL schreibend – umgesetzt

Als A per `POST /graphql`: `createPortfolio` „GraphQL-Test“ (id 133), `updatePortfolio` auf „GraphQL-Test 2“, `simulate` mit Seed 42 und `save: true` als „GraphQL Simulation“ (id 67); Portfolio und Simulation erschienen in der MPA, `deleteSavedSimulation` und `deletePortfolio` lieferten `true`, danach waren beide in der MPA verschwunden (AT-GQL-01). Ohne Namen `VALIDATION_ERROR` „Bitte einen Namen mit höchstens 100 Zeichen eingeben.“ mit `details` zu `name`; ohne Header `UNAUTHORIZED` „Bitte melde dich an.“; „Krypto-Test“ `INSUFFICIENT_DATA` mit der Meldung aus REST, ETHA genau einmal; die HTTP-Antwort war jeweils `200` (AT-GQL-02). Als B lieferten `updatePortfolio` und `deletePortfolio` für id 34 jeweils `NOT_FOUND` „Portfolio nicht gefunden.“; B sieht keine Portfolios (AT-GQL-03). `compare` und `POST /comparisons` mit Seed 42 liefern identisch 215.452 € und 2.890.032 € und `pairwise` 0, wie die Vergleichsseite (AT-GQL-04). AT-GQL-05 automatisiert.

### SPA-Touchpoint (ADR-08) – umgesetzt

Der Link „App (SPA)“ lädt das Dokument einmal (`app.html` wird mit `301` auf `/app` umgeleitet, dann `200`); beim Öffnen eines Portfolios, beim Simulieren und beim Wechsel zu den gespeicherten Simulationen folgt keine weitere Anfrage vom Typ „document“. „Mein Weltportfolio“ mit Seed 42 ergibt in der SPA dieselben Werte wie in der MPA (Median 215.452 €, p5 111.918 €, p95 424.998 €, Drawdown 16,37 %). Der Titel des Browser-Tabs wechselt von „Portfolios – LookUpStock (SPA)“ zu „Mein Weltportfolio – LookUpStock (SPA)“ (AT-ADR8-01). „Sparplan 20 Jahre“ zeigt das gespeicherte Ergebnis ohne Neuberechnung (Schätzzeitraum 01.11.2016 bis 01.10.2026, gespeichert am 18.10.); Zurück, Vor und Neuladen behalten die Ansicht; die Adresse `/app#/simulations/3` in einem neuen Tab zeigt die Simulation direkt mit Diagramm und Kennzahlen (AT-ADR8-02). `/app` ohne Hash leitet auf `#/portfolios`, `#/unbekannt` zeigt „Seite nicht gefunden“ mit Link zu den Portfolios, `#/portfolios/999999` „Portfolio nicht gefunden.“, abgemeldet erscheint der Hinweis mit Link zur Anmeldung (AT-ADR8-03). Nach jedem Wechsel liegt der Fokus auf der Überschrift der neuen Ansicht (AT-ADR8-04). Horizont ungültig und „Krypto-Test“ ergeben dieselben Meldungen wie in der MPA (AT-ADR8-06). AT-ADR8-05 automatisiert.

### NF-10 neue Seiten (AT-NF10-02) – bestanden

Bei 360 px sind das Dashboard (Monatsauswahl, Tabelle, Export) und die SPA (Liste, Portfolio mit Ergebnis, gespeicherte Simulation) ohne horizontales Scrollen bedienbar. Die Bezeichnungen der Kennzahlen brechen nur zwischen Wörtern um, der Export-Knopf hat die volle Breite, die Achsenbeschriftung in der SPA ist lesbar.

### Nachtest H2 bis H7, NF-09 und NF-01

- **H2 behoben:** Nach einem erfolgreichen Lauf blenden „Krypto-Test“ (`422`) und Horizont 0 das alte Ergebnis aus, auf der Simulationsseite auch das Speichern-Formular, auf der Vergleichsseite ebenso.
- **H3 behoben:** Die Meldungen der Vergleichsseite sind rot wie auf der Simulationsseite.
- **H4 behoben:** Der Editor zeigt nach dem Laden keine Pflichtfeld-Meldungen; Verlassen des leeren Namens zeigt nur die Meldung am Namen, „Speichern“ alle Meldungen und „Bitte zuerst die markierten Fehler beheben.“, die Meldung am Namen verschwindet beim Tippen. Dazu Hinweis H9.
- **H5 behoben:** Bei 360 px stehen „Bearbeiten“ und „Löschen“ untereinander in voller Breite, ohne Umbruch im Wort.
- **H6 nicht behoben:** Auf der Simulationsseite ist die Achsenbeschriftung bei 360 px lesbar. Auf der Vergleichsseite ist die Y-Achse abgeschnitten (H8), auch am Desktop.
- **H7 behoben:** `POST /portfolios/list` liefert im Browser `405` mit lesbarer Meldung „Diese HTTP-Methode ist hier nicht erlaubt.“. `Access-Control-Allow-Origin` ist im Browser nicht auslesbar, weil der Header nicht für Skripte freigegeben wird; dass der Body lesbar ist, beweist, dass der Header gesendet wird (sonst bricht `fetch` mit einem CORS-Fehler ab). Automatisiert: `CorsConfigTest`.
- **NF-09 erfüllt (AT-NF09-01):** Übernahme aus `bedienbarkeitstest-nf-09.md` (Carl Stibbe): P1 3:42 und P2 4:48 bestanden, P3 5:26 nicht bestanden (Zeitgrenze), 2 von 3 Personen ohne Hilfe unter 5 Minuten. Wiederkehrende Beobachtung: Nach dem Speichern des Portfolios war zwei Personen nicht klar, wie sie zur Simulation gelangen.
- **NF-01 und E33 erfüllt (AT-04-14, AT-07-13):** „Messportfolio“, 10.000 € Startkapital, 200 € Sparrate, 40 Jahre, 10.000 Pfade, je ein Aufwärmlauf und 20 Messläufe; Rechenzeit laut `runtimeMs`: Simulation 1.614 bis 1.840 ms (20 von 20 ≤ 3.000 ms), Vergleich 3.153 bis 3.346 ms (20 von 20 ≤ 6.000 ms). Die Dauer der Anfragen im Netzwerk-Tab betrug 1,72 bis 2,13 s bzw. 3,28 bis 3,51 s. Gegenüber Sprint 4 (Vergleich 2.957 bis 3.045 ms) ist der Vergleich etwa 6 % langsamer; die Kurse kommen jetzt über den marktdaten-service.

---

## 4. Gefundene Fehler und Hinweise

Priorität nach Testkonzept, Abschnitt 11. Die Nummerierung setzt Sprint 3 und 4 fort.

| ID | Repo | Beschreibung | Testfälle | Priorität | Art |
|---|---|---|---|---|---|
| B8 | FE, BE | „Durchgeführte Simulationen“ zählt eine über die Oberfläche gespeicherte Simulation doppelt. Das Speichern sendet erneut `POST /simulations` mit `save: true`; das Backend rechnet neu und erfasst ein zweites `SIMULATION_RUN`. Akzeptanzkriterium „… jeweils um 1“ (E37) nicht erfüllt; die Kennzahl ist um die Zahl der gespeicherten Simulationen zu hoch. Behebung im Team abstimmen, siehe Abschnitt 7. | AT-11-03, AT-11-07 | hoch | Fehler |
| B9 | FE | `dashboard.html` enthält den Navigationseintrag „Auswertungen“ fest im HTML (Zeile 18). Nutzer ohne Rolle sehen ihn auf dieser Seite; für M erscheint er doppelt, weil `main.js` ihn zusätzlich einfügt. Außerdem fehlt dort der Eintrag „App (SPA)“. Akzeptanzkriterium aus E40 nicht erfüllt. | AT-11-06, AT-11-01, AT-NF10-02 | hoch | Fehler |
| B10 | BE | Der echte Twelve-Data-Key war am 11.10.2026 in Commit b63f7d8 eingecheckt und wurde am 28.10.2026 in d2b550e nur entfernt; er stand weiter in der Git-Historie (NF-06). Während der Abnahme hat Carl Stibbe den Key bei Twelve Data neu erzeugt und in `lookupstock-marktdaten` ersetzt; der alte Key ist damit wertlos. Nachweis, dass der neue funktioniert: AT-ADR7-06. | AT-ADR7-02 | hoch | Fehler, **behoben am 03.11.2026** |
| H8 | FE | Die Y-Achsenbeschriftung der Fächerdiagramme auf der Vergleichsseite ist abgeschnitten („.000.000 €“), am Desktop und bei 360 px. Ursache: Der linke Rand des Diagramms (`CHART_SIZE.left` = 96 in `chart.js`) ist für Beschriftungen ab acht Ziffern zu schmal; seit der gemeinsamen Y-Achse (H1) treten sie bei großen Endwerten auf beiden Diagrammen auf. Grund dafür, dass H6 nicht behoben ist. | Nachtest H6, AT-07-02 | niedrig | Hinweis |
| H9 | FE | Der leere Portfolio-Editor zeigt nach dem Laden bei der Gewichtssumme rot „ungültig“, bevor etwas eingegeben wurde. Die Festlegung zu H4 betrifft nur die Meldungen an den Feldern; die Statusanzeige der Summe ist laut Kommentar in `portfolio-ui.js` bewusst immer aktiv. Sie wirkt aber wie ein Fehler vor der Eingabe; vorgeschlagen ist eine neutrale Anzeige, solange nichts eingegeben wurde. | Nachtest H4 | niedrig | Hinweis |
| H10 | FE | Auf dem Dashboard bleibt nach dem erfolgreichen Laden ein leeres blaues Hinweisfeld über der Tabelle stehen (`#dashboard-notice`, der Text wird geleert, das Feld nicht ausgeblendet). | AT-11-01, AT-NF10-02 | niedrig | Hinweis |
| H11 | Docs | Die SPA ist unter `/app#/…` erreichbar; `app.html` wird mit `301` dorthin umgeleitet. ADR-08 und die Testfälle nennen `app.html#/…`. Für die Nutzung ohne Folgen. | AT-ADR8-01 bis AT-ADR8-03 | niedrig | Hinweis |
| H12 | FE | Die Portfolioliste der SPA zeigt „1 Positionen“ statt „1 Position“; die MPA zeigt korrekt „1 Position“. | AT-ADR8-01 | niedrig | Hinweis |

**Hinweise zum Vorgehen (für die Retrospektive und das Portfolio):**

- Für `main` ist in den Repositories kein Branch-Schutz aktiv, obwohl er seit Sprint 4 Teil der Definition of Done ist.
- Docs #157 (US-12), #159 und #160 wurden am 01.11.2026 vor der Abnahme geschlossen.
- B10 zeigt, dass das Entfernen eines Geheimnisses aus dem Code nicht genügt; ein eingecheckter Key muss sofort ersetzt werden.
- In `bedienbarkeitstest-nf-09.md` nennt die Statuszeile den 20., 22. und 24.10.2026, die Ergebnisbögen den 26.10.2026; die Statuszeile verweist auf einen Abschnitt 9, den es nicht gibt.
- Die Rolle `MANAGER` lässt sich nur in der Datenbank setzen (E40, E42); in der README des Backends fehlt dazu eine Anleitung.

---

## 5. Nachweise der manuellen Tests

| Prüfung | Testfälle | Beobachtung |
|---|---|---|
| Simulation und Fehlerfälle | Nachtest H2, H3 | „Mein Weltportfolio“, Basiseingaben, Seed 42: Median 215.452 €, p5 111.918 €, p95 424.998 €, Eingezahlt 58.000 €, VaR 0 €, Drawdown 16,37 %, Kursdaten 01.12.2016 bis 01.10.2026; danach „Krypto-Test“ → `422`, ETHA einmal, Ergebnis und Speichern ausgeblendet; Vergleich „Welt“ gegen „Aktien“ 215.452 € und 2.890.032 €, 0 %; „Krypto-Test“ gegen „Aktien“ und Horizont 0 jeweils rote Meldung, kein Ergebnis |
| Editor und Handyansicht | Nachtest H4, H5, H6 | keine Meldungen nach dem Laden (außer „ungültig“, H9); 360 px: Knöpfe ohne Umbruch im Wort; Vergleich: Y-Achse „.000.000 €“ (H8) |
| CORS | Nachtest H7 | `fetch` auf `POST /portfolios/list` mit Token: Status `405`, Meldung lesbar |
| Messung | AT-04-14, AT-07-13 | Simulation 1.614 bis 1.840 ms, Vergleich 3.153 bis 3.346 ms, je 20 von 20 im Zielwert |
| Netzwerk | AT-ADR7-01 | Spalte „Domain“: alle App-Anfragen an `lookupstock-da2951e42089.herokuapp.com` |
| marktdaten-service | AT-ADR7-01, AT-ADR7-02 | `/actuator/health` → `UP`; `/internal/search?q=msci` ohne Header → `UNAUTHORIZED` |
| Heroku (Carl Stibbe) | AT-ADR7-01, AT-ADR7-02 | drei Apps; Namen der Konfigurationsvariablen wie erwartet; Werte nicht angesehen |
| SPA | AT-ADR8-01 bis AT-ADR8-04, AT-ADR8-06 | ein Dokumentabruf (`app.html` 301 → `app` 200); identische Werte wie MPA; Tab-Titel wechselt; Zurück, Vor, Neuladen, Adresse im neuen Tab; unbekannte Route, unbekannte ID, ohne Hash, abgemeldet; Fokus auf der Überschrift |
| GraphQL | AT-GQL-01 bis AT-GQL-04 | Portfolio 133 und Simulation 67 angelegt, geändert, in der MPA sichtbar, gelöscht; Fehlercodes `VALIDATION_ERROR`, `UNAUTHORIZED`, `INSUFFICIENT_DATA`, `NOT_FOUND`; `compare` = REST |
| Rolle (Carl Stibbe) | Vorbereitung | `UPDATE app_user SET role = 'MANAGER' WHERE email = 'gf@example.com';` → `UPDATE 1`; Kontrolle → `MANAGER` |
| Ohne Rolle | AT-11-06, AT-12-03 | als A: Bericht und Export `403`, ohne Header `401`, JSON; `dashboard.html` „Dafür fehlt die Berechtigung.“, Eintrag „Auswertungen“ sichtbar (B9); abgemeldet → Anmeldeseite |
| Dashboard | AT-11-01, AT-11-05 | 9 Kennzahlen = API; Oktober → Meldung ohne Werte; Dezember ausgegraut; API `400` für `2026-12`, `2026-10`, `11-2026` |
| Zähltest | AT-11-03, AT-11-04 | 1 \| 6 \| 2 \| 28 \| 2 \| 27 \| 0 \| 3 \| 13 → 2 \| 7 \| 3 \| 30 \| 3 \| 28 \| 2 \| 5 \| 14 → nach den Fehlerfällen 2 \| 8 \| 3 \| 30 \| 3 \| 28 \| 2 \| 5 \| 14 |
| Datenbank (Carl Stibbe) | AT-11-07 | `\d usage_event`: drei Spalten, kein Fremdschlüssel; 15 neueste Zeilen nur Ereignisart und Zeitpunkt (UTC) |
| Export | AT-12-01, AT-12-02, AT-12-03 | Dateiname, Inhalt, BOM, Excel in drei Spalten mit „ü“; Oktober → Meldung, kein Download |
| Kurse vorladen | AT-ADR7-06 | VT vorher `0 rows`; „Vorladen“ um 11:36 gespeichert; danach 5 Monatswerte, `fetched_at` 10:36:12 UTC |
| Handyansicht | AT-NF10-02 | Dashboard und SPA bei 360 px ohne seitliches Scrollen |

---

## 6. Abweichungen vom Testkonzept

- **Zeitpunkt:** Abnahme am 03.11.2026 statt am Ende des Sprints (Abschnitt 1).
- **AT-11-03:** Die Beispielsimulation wurde zweimal gestartet; erwartet sind deshalb +2 statt +1 Beispielsimulationen.
- **AT-11-04:** Für die Registrierung mit vorhandener E-Mail wurde `abnahme3.a@example.com` statt `anna@example.com` verwendet. Bei `anna@example.com` war nicht sicher, dass das Konto existiert; eine erfolgreiche Registrierung hätte den Test verfälscht.
- **AT-11-07:** Statt der neuesten zehn wurden die neuesten 15 Zeilen angesehen, damit AT-11-03 und AT-11-04 vollständig enthalten sind. Die Zeitpunkte stehen in UTC (`timestamp without time zone`); die Monatsgrenzen nach deutscher Zeit sind automatisiert belegt (AT-11-02).
- **AT-ADR7-01:** Der Status „running“ war auf dem Screenshot der App-Liste nicht zu sehen; dass beide Dienste laufen, zeigen der Health-Check und alle Tests über den core-service. Die Netzwerkprüfung erfolgte während der Messung (Simulation, Vergleich) und des Zähltests (Suche nach URTH).
- **AT-ADR7-02:** Die Screenshots der Konfigurationsvariablen zeigen den App-Namen nicht; die Zuordnung ergibt sich aus `MARKTDATEN_URL` bzw. `TWELVE_DATA_API_KEY`. Der Statuscode von `/internal/search` war im Browser nicht sichtbar, nur der Code `UNAUTHORIZED` im Body; die `401` ist automatisiert belegt (`withoutKeyReturns401`).
- **Heroku und Datenbank:** Der Product Owner hat keinen Zugang; die Prüfungen hat Carl Stibbe durchgeführt und per Screenshot belegt.
- **API-Prüfungen, GraphQL und Messungen** wurden mit `fetch` in der Konsole der Entwicklertools durchgeführt.

---

## 7. Folgen

Das Team hat am 03.11.2026 beschlossen, die offenen Punkte in einem kurzen Sprint 6 zu bearbeiten. Die Nachtests stehen deshalb nicht in diesem Protokoll, sondern in den Abnahmetestfällen und im Abnahmeprotokoll Sprint 6.

- **US-11 bleibt offen (Docs #156)** und geht in Sprint 6. Für B8 und B9 werden Issues mit Label `bug` angelegt (Testkonzept, Abschnitt 11). Nach dem Merge werden AT-11-03 und AT-11-06 in der Abnahme Sprint 6 wiederholt.
  - **B9 (FE):** Den festen Eintrag „Auswertungen“ aus `dashboard.html` entfernen und „App (SPA)“ wie auf den übrigen Seiten ergänzen.
  - **B8:** Zwei Wege sind möglich: (a) Das Backend erfasst bei `save: true` nur `SIMULATION_SAVED`, weil die Oberfläche vorher immer simuliert; E37 wird dann für Speichern ohne vorherige Simulation (nur über die API) präzisiert. (b) Ein eigener Endpunkt speichert ein bereits berechnetes Ergebnis ohne Neuberechnung; das ist aufwendiger und ändert das API-Design. Empfehlung: (a); die Entscheidung trifft der Product Owner im Sprint Planning Sprint 6.
- **US-12 ist abgenommen;** Docs #157 ist bereits geschlossen.
- **ADR-07, GraphQL schreibend und SPA-Touchpoint sind umgesetzt;** Docs #158 wird geschlossen, #159 und #160 sind bereits geschlossen. B10 ist behoben.
- **H8 bis H12** werden als Issues angelegt (Priorität niedrig) und in Sprint 6 bearbeitet. Mit H8 wird H6 erneut geprüft.
- **NF-01, NF-09, NF-10 und NF-14 (b)** sind nachgewiesen.
- **Docs #170** ist mit diesem Protokoll erledigt.
