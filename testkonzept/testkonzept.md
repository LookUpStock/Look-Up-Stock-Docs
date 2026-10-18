# Testkonzept – LookUpStock

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 3 (12.10.2026 – 18.10.2026), Version 3 |
| **Issue** | Docs #100 Fortschreibung des Testkonzepts (Version 3); Docs #61 (Version 2); Docs #23 Testkonzept v1 (Version 1) |
| **Verantwortlich** | Version 1: Paul Kaufmann (Paul02112004); Version 2 und 3: Mert Eminel (KujoBaraka) |
| **Stand** | 18.10.2026 |
| **Grundlagen** | Kurzformat-Tabellen Sprint 1 mit Akzeptanzkriterien (Stand 30.09.2026), Entscheidungen des Product Owners E1 bis E7 (02.10.2026), API-Design (Stand 02.10.2026), ADR-01 bis ADR-03, Aktivitätsdiagramm US-04 v2, Domänenmodell v3, Sprint-Planning-Protokolle Sprint 1 und 2, CONTRIBUTING.md; ab Version 2 zusätzlich Kurzformat-Tabellen Sprint 2 (US-01, E11 bis E14), PO-Entscheidungen E8 bis E10, NFRs Version 3 und die Backend-Umsetzung BE #30; ab Version 3 zusätzlich Kurzformat-Tabellen Sprint 3 (US-05 bis US-08, E24 bis E34), Aktivitätsdiagramm US-07 v1, ADR-05, Abnahmeprotokoll Sprint 2 und die Backend-Umsetzung BE #67 bis #71 |

Die Abnahmetestfälle stehen je Sprint in einer eigenen Datei: [Sprint 1](abnahmetestfaelle-sprint-1.md), [Sprint 2](abnahmetestfaelle-sprint-2.md), [Sprint 3](abnahmetestfaelle-sprint-3.md). Die Änderungen gegenüber den Vorversionen stehen am Ende dieses Dokuments.

---

## 1. Zweck und Geltungsbereich

Dieses Testkonzept legt fest, wie LookUpStock getestet wird: welche Komponenten mit welchen Methoden, auf welchen Teststufen, mit welchen Daten und Werkzeugen geprüft werden, wann eine User Story als abgenommen gilt und wie gefundene Fehler behandelt werden.

Es gilt für das gesamte Projekt, also für das Backend (`Look-Up-Stock-BackEnd`), das Frontend (`Look-Up-Stock-FrontEnd`) und die Abnahme der User Stories durch den Product Owner. Das Dokument wird in jedem Sprint fortgeschrieben (siehe Abschnitt 13). Die konkreten Abnahmetestfälle werden je Sprint in einer eigenen Datei geführt.

## 2. Testziele

1. **Korrektheit der Kernlogik:** Die Portfolio-Prüfung (US-03, beim Ändern US-06), die Monte-Carlo-Simulation (US-04), die Kennzahlen (US-05) und der Vergleich (US-07) liefern für gültige Eingaben korrekte Ergebnisse und weisen ungültige Eingaben zuverlässig zurück. Beim Vergleich gehen Unterschiede nur auf die Zusammensetzung zurück (gleicher Seed, gemeinsamer Schätzzeitraum).
2. **Erfüllung der Akzeptanzkriterien:** Jedes Akzeptanzkriterium der Stories eines Sprints ist durch mindestens einen Abnahmetestfall abgedeckt und nachweislich erfüllt.
3. **Nachweis der nicht-funktionalen Anforderungen:** Die operationalisierten NFRs (insbesondere NF-01, NF-07 und NF-08, ab Sprint 3 auch NF-09) werden durch Tests oder definierte Prüfungen belegt.
4. **Zugriffsschutz (ab Sprint 2):** Passwörter werden nur als Hash gespeichert (NF-03), jeder Nutzer sieht, ändert und löscht nur seine eigenen Portfolios und gespeicherten Simulationen (NF-04), und eine Anmeldung ist zeitlich begrenzt und beendbar (NF-16).
5. **Schutz vor Regressionen:** Automatisierte Tests laufen bei jedem Pull Request in der CI-Pipeline, sodass Änderungen bestehende Funktionen nicht unbemerkt beschädigen.

## 3. Testobjekte

Der Schwerpunkt liegt auf der Kernlogik der Kern-Stories US-04 und US-07 und der Portfolio-Prüfung aus US-03. Die Klassennamen stammen aus ADR-02, ADR-03 und ADR-05 und entsprechen dem Code (Stand 18.10.2026).

| Komponente | Schicht (ADR-01) | Story | Was geprüft wird |
|---|---|---|---|
| `Portfolio` mit `validate()` und `getWeights()` | Logik | US-03, US-04 | Prüfregeln: Name Pflicht mit höchstens 100 Zeichen, mindestens eine Anlage (keine Höchstzahl), keine doppelten Anlagen, jede Gewichtung > 0 % mit höchstens 2 Nachkommastellen, Summe 100 % ± 0,01 Prozentpunkte bei exakter Dezimalrechnung; Umrechnung der Gewichtungen in Anteile in fester Reihenfolge (ADR-03) |
| `ParameterEstimator` | Logik | US-04 | Schätzung von Jahresrendite und Jahresvolatilität aus Kursreihen; Erkennung von weniger als 3 Jahren Kurshistorie |
| `MonteCarloEngine` | Logik | US-04 | Pfadanzahl, Monatsschritte über den Anlagehorizont, Berücksichtigung von Startkapital und Sparrate, Reproduzierbarkeit bei gleichem Seed |
| `MetricsCalculator` | Logik | US-04, US-05 | 5., 50. und 95. Perzentil je Monat (`percentiles()`, BE #49); Kennzahlen zum Ende des Anlagehorizonts (`metrics()`, BE #71): Einzahlungen, Endwerte, Value at Risk, maximaler Drawdown, Zielwahrscheinlichkeit (E24 bis E27) |
| `SimulationService` (GRASP-Controller, ADR-02) | Logik | US-04, US-07 | Koordination der Schritte in der richtigen Reihenfolge, Seed-Erzeugung ohne Angabe (`ensureSeed`), Fehlerfälle (Portfolio fehlt, Kursdaten nicht verfügbar, zu kurze Historie); `run()` simuliert einen Satz Gewichtungen und wird vom Vergleich wiederverwendet (ADR-05) |
| `ComparisonService` (GRASP-Controller, ADR-05) | Logik | US-07 | Genau zwei Einträge, Portfolio oder Anlage (Anlage = 100 %), Standardbezeichnung, ein Seed und ein gemeinsamer Schätzzeitraum für beide Einträge, Wahrscheinlichkeit „Eintrag 1 höher“, Fehlerfälle 400, 404, 422 |
| `SavedSimulationService` | Logik | US-08 | Speichern des berechneten Ergebnisses unter einem Namen, Liste neueste zuerst, Abruf ohne Neuberechnung, Löschen; nur eigene Simulationen (NF-04) |
| `PortfolioService` | Logik | US-03, US-06 | Aufruf von `validate()` vor dem Speichern und Ändern; Ablehnung unbekannter `assetId`; ungültige Änderung lässt das Portfolio unverändert; Ändern und Löschen fremder Portfolios liefert 404 (NF-04) |
| `PriceHistoryService` und `PriceCache` mit `MarketDataProvider` | Daten | US-02, US-04, US-07 | Kursdaten jünger als 24 h aus dem Cache ohne Abruf, sonst Abruf bei Twelve Data und Speichern im Cache; bei Ausfall ohne gültigen Cache 503 (NF-07); frische und gecachte Kurse auf 6 Nachkommastellen gerundet (F4, NF-08); Suchergebnisse 24 h im `SearchCache` (NF-02). Primärer Anbieter ist Twelve Data; ein Fallback-Anbieter ist noch nicht entschieden. In den Unit-Tests wird der `MarketDataProvider` durch Mockito-Mocks ersetzt (Stand Version 3, siehe Abschnitt 6). |
| REST-Controller (`SimulationController`, `ComparisonController`, Such- und Portfolio-Endpunkte) | Präsentation | US-02 bis US-08 | HTTP-Status, Fehlercodes und Fehlerformat laut API-Design, einschließlich `PUT`/`DELETE /portfolios/{id}`, `POST /comparisons` und Speichern, Liste, Abruf und Löschen von Simulationen |
| `AuthService` (`register`, `login`, `logout`, `authenticate`) | Logik | US-01 | Prüfung von E-Mail-Format und Passwortlänge, Vergleich der E-Mail ohne Groß-/Kleinschreibung (E11), Speichern nur des Passwort-Hashes (NF-03), gleiche Meldung bei falscher E-Mail und falschem Passwort, Ablauf der Anmeldung nach 24 h (NF-16), Sperren des Tokens beim Abmelden |
| `AuthController` | Präsentation | US-01 | HTTP-Status und Meldungen von `/auth/register`, `/auth/login` und `/auth/logout` laut API-Design |
| Token-Prüfung in den geschützten Endpunkten (BE #54, BE #50) | Präsentation / Logik | US-03, US-04 | `401` ohne gültigen Token; Zuordnung der Portfolios zum Nutzer; `404` bei fremden Portfolios (NF-04) |
| Frontend-Prüflogik und Darstellung | Präsentation | US-01 bis US-08 | Login- und Registrierungsmaske (FE #4) mit Prüfung von E-Mail und Passwort vor dem Absenden und Erkennung einer abgelaufenen Anmeldung über `expiresAt` (E14); Mindestlänge des Suchbegriffs, Gewichtungssumme, Formatierung, Fehlermeldungen, Fächerdiagramm; ab Sprint 3 Kennzahlen unter dem Fächerdiagramm, Bearbeiten und Löschen im Portfolio-Editor, Vergleichsseite, Speichern und Liste gespeicherter Simulationen mit Namensprüfung (FE #17) |

## 4. Teststufen

| Teststufe | Ziel | Werkzeug | Verantwortlich | Zeitpunkt |
|---|---|---|---|---|
| **Unit-Tests** | Kernlogik isoliert prüfen, ohne Datenbank, ohne Web-Server und ohne Internet. Abhängigkeiten werden durch Mocks ersetzt. | Backend: JUnit 5, Mockito<br>Frontend: Node-Test-Runner (`node:test`) | Entwickler der jeweiligen Komponente; Unit-Tests der Kernlogik laut Sprint Backlog: BE #32 (Portfolio), BE #38 (Suche), BE #41 (Simulationslogik), ab Sprint 2 BE #30 (Anmeldung) und FE #4 (Login-Maske), ab Sprint 3 BE #68 bis #71 und FE #17 | Während der Umsetzung, vor dem Pull Request; automatisch in der CI bei jedem Pull Request |
| **Integrationstests** | REST-Endpunkte einschließlich Validierung, Fehlerformat und Zusammenspiel Controller → Service prüfen; Marktdaten kommen aus dem `MockMarketDataProvider`. | Spring Boot Test mit `@WebMvcTest` und `MockMvc` | Entwickler des jeweiligen Endpunkts (BE #27, BE #33, BE #39/#40; ab Sprint 2 BE #30, BE #50, BE #54; ab Sprint 3 BE #68, BE #69, BE #70) | Mit Fertigstellung eines Endpunkts; automatisch in der CI bei jedem Pull Request |
| **Abnahmetests** | Prüfung aus Nutzersicht gegen die Akzeptanzkriterien, die Alternativszenarien des Aktivitätsdiagramms und relevante Grenzwerte | Browser (Oberfläche) und ein HTTP-Client für direkte API-Aufrufe; automatisierte Tests für die als „automatisiert“ gekennzeichneten Testfälle; Testfälle in `abnahmetestfaelle-sprint-<n>.md` | Product Owner (KujoBaraka), unterstützt von den Entwicklern | Nach Fertigstellung einer Story, spätestens vor dem Sprint Review |
| **Nicht-funktionale Tests** | Nachweis der NFRs, in Sprint 1 vor allem Performance der Simulation (NF-01), Ausfallsicherheit (NF-07) und Reproduzierbarkeit (NF-08); in Sprint 2 zusätzlich Passwortschutz (NF-03), Zugriffsschutz (NF-04) und begrenzte Sitzungsdauer (NF-16); in Sprint 3 Bedienbarkeit (NF-09) und die Rechenzeit des Vergleichs (E33) | NF-01: wiederholte Messung über das Feld `runtimeMs` der Simulationsantwort; NF-08: JUnit-Test; übrige siehe Abschnitt 10 | Entwickler (Messung), Product Owner (Abnahme) | NF-08 und NF-12 automatisch in der CI; NF-01 und NF-07 vor dem Sprint Review |

**Durchführung der Abnahmetestfälle.** Grenzwerttestfälle, reine API-Fälle und Fälle, die einen gezielt manipulierten Mock erfordern, werden als automatisierte Unit- oder Integrationstests umgesetzt und bei jedem Pull Request in der CI-Pipeline ausgeführt. Manuelle Abnahmetests durch den Product Owner beschränken sich auf Normalfälle und Alternativszenarien aus Nutzersicht sowie auf Messungen, die nicht in der CI laufen können (NF-01, NF-02). Das senkt den manuellen Aufwand pro Sprint, macht die Ausführung reproduzierbar und erhöht die Testabdeckung (NF-11). Welche Durchführung für einen Testfall gilt, steht in der Spalte „Durchführung“ der Abnahmetestfälle. Grenzwerte desselben Eingabefelds werden in einem Testfall zusammengefasst und als parametrisierter Test umgesetzt.

## 5. Testmethoden

**Äquivalenzklassen.** Jedes Eingabefeld wird in Klassen gültiger und ungültiger Werte zerlegt, die sich gleich verhalten sollen. Aus jeder Klasse wird mindestens ein Vertreter getestet. Beispiel Anlagehorizont: gültig sind ganze Zahlen von 1 bis 40; ungültig sind Werte kleiner 1, Werte größer 40, Kommazahlen und ein leeres Feld.

**Grenzwertanalyse.** An jeder Grenze wird der Wert knapp darunter, genau auf der Grenze und knapp darüber getestet, weil Fehler wie `<` statt `<=` dort auftreten. Beispiele: Pfadanzahl 999, 1.000, 50.000 und 50.001; Gewichtungssumme 99,98 %, 99,99 %, 100,01 % und 100,02 %.

Exakte Dezimalrechnung: Die Grenzwerte der Gewichtungssumme liegen genau auf der Toleranz von ± 0,01 Prozentpunkten. Bei Rechnung mit `double` können Summen mit Nachkommastellen nicht exakt dargestellt werden; so ergibt die Prüfung `abs(70 + 30.01 - 100) <= 0.01` in `double` den Wert `false`, obwohl die Summe 100,01 % gültig ist. Deshalb ist festgelegt, dass das Backend Gewichtungen als exakte Dezimalzahlen (`BigDecimal`) verarbeitet und Gewichtungen höchstens 2 Nachkommastellen haben (PO-Entscheidung E2). Nachgewiesen wird das durch die Grenzwerte der Gewichtungssumme (AT-03-03, darunter 70 % + 30,01 %) und den Gleitkomma-Testfall in AT-03-05 (69,99 % + 30,01 % wird als 100,00 % akzeptiert). Beide werden als Unit-Tests von `Portfolio.validate()` umgesetzt.

**Ableitung aus dem Aktivitätsdiagramm.** Für die Kern-Story US-04 wird aus dem Aktivitätsdiagramm (`diagramme/aktivitaetsdiagramm_us-04_v2.puml`) für jede Verzweigung mindestens ein Testfall abgeleitet, sodass jeder Pfad vom Start bis zu einem Endknoten mindestens einmal durchlaufen wird (Pfadabdeckung der Alternativszenarien):

| Verzweigung im Aktivitätsdiagramm | Pfade | Abnahmetestfälle |
|---|---|---|
| Eingaben prüfen | gültig / ungültig (Rücksprung zur Eingabe, HTTP 400) | AT-04-01, AT-04-04, AT-04-10 bis AT-04-12 |
| Portfolio laden | vorhanden / nicht vorhanden (HTTP 404) | AT-04-01, AT-04-05 |
| Seed | ohne Seed (System erzeugt) / mit Seed | AT-04-01, AT-04-02 |
| Kursdaten im Cache | im Cache und jünger als 24 h / nicht im Cache oder älter | AT-04-07 |
| Finanzdaten-API | erreichbar / nicht erreichbar (HTTP 503) | AT-04-01, AT-04-06 |
| Kurshistorie je Anlage | ≥ 36 Monatswerte / < 36 Monatswerte (HTTP 422, E8) | AT-04-08, AT-04-09 |

Seit Version 2 des Aktivitätsdiagramms ist der Cache-Fallback aus NF-07 als eigene Verzweigung enthalten. Die Prüfung des Tokens (`401`) ist eine Vorbedingung von US-04 und nicht Teil des Aktivitätsdiagramms; sie wird durch AT-04-16 abgedeckt.

Für die zweite Kern-Story US-07 gilt dasselbe Vorgehen mit dem Aktivitätsdiagramm `diagramme/aktivitaetsdiagramm_us-07_v1.puml` (ab Version 3). Jede Verzweigung ist durch mindestens einen Testfall aus `abnahmetestfaelle-sprint-3.md` abgedeckt; die Auswertung der Pfadabdeckung erfolgt in Docs #102:

| Verzweigung im Aktivitätsdiagramm US-07 | Pfade | Abnahmetestfälle |
|---|---|---|
| Eingaben prüfen | gültig / ungültig (Rücksprung zur Eingabe, HTTP 400) | AT-07-01, AT-07-04, AT-07-05, AT-07-12 |
| Art des Eintrags | Portfolio / Anlage | AT-07-01 (je ein Eintrag), AT-07-02 (zwei Portfolios) |
| Portfolio laden | eigenes Portfolio vorhanden / nicht vorhanden oder fremd (HTTP 404) | AT-07-01, AT-07-06 |
| Anlage laden | vorhanden / nicht vorhanden (HTTP 404) | AT-07-01, AT-07-07 |
| Bezeichnung | angegeben / keine Bezeichnung (Name wird verwendet) | AT-07-02, AT-07-01 |
| Schleife über die Einträge | weiterer Eintrag / beide Einträge geladen | AT-07-01 |
| Seed | ohne Seed (System erzeugt einen für beide Einträge) / mit Seed | AT-07-01, AT-07-02 |
| Kursdaten im Cache | im Cache und jünger als 24 h / nicht im Cache oder älter | AT-07-11 |
| Finanzdaten-API | erreichbar / nicht erreichbar (HTTP 503) | AT-07-01, AT-07-11 |
| Kurshistorie je Anlage | alle ≥ 36 Monatswerte / Anlage ohne Kurse oder < 36 Monatswerte (HTTP 422) | AT-07-01, AT-07-08, AT-07-09 |
| Gemeinsamer Schätzzeitraum | ≥ 36 Monatswerte / < 36 Monatswerte (HTTP 422, E29) | AT-07-01, AT-07-10 |
| Zielbetrag | angegeben / kein Zielbetrag | AT-07-02, AT-07-01 |

Die Token-Prüfung (`401`) ist wie bei US-04 Vorbedingung und nicht Teil des Diagramms; sie wird durch AT-07-12 abgedeckt.

**Zeitabhängige Tests.** Der Ablauf einer Anmeldung nach 24 Stunden (NF-16) und das Alter von Cache-Daten (NF-07) werden nicht abgewartet. Das Backend bezieht die aktuelle Zeit über eine austauschbare Uhr (`Clock`, `ClockConfig`). Im Unit-Test wird eine feste Uhrzeit vorgegeben, sodass Grenzwerte wie „genau 24 h nach dem Login“ sekundengenau und reproduzierbar geprüft werden (AT-01-10, AT-04-07).

**Test-Orakel für die Simulation.** Weil Simulationsergebnisse zufallsabhängig sind, werden in Unit-Tests Fälle mit bekanntem Ergebnis verwendet: Bei einer Volatilität von 0 sind alle Pfade gleich und der Endwert lässt sich aus Startkapital, Sparrate und Rendite exakt berechnen. Zusätzlich werden Eigenschaften geprüft, die immer gelten müssen, z. B. 5. Perzentil ≤ 50. Perzentil ≤ 95. Perzentil in jedem Monat.

## 6. Testdaten

Alle automatisierten Tests verwenden den `MockMarketDataProvider`, der das Interface `MarketDataProvider` mit festen, synthetischen Kursreihen implementiert. Er wird auch für die Abnahmetests verwendet, damit Ergebnisse und Fehlerfälle gezielt herbeigeführt werden können.

| Testdaten | Inhalt | Zweck |
|---|---|---|
| Anlage EUNL (`assetId` 17) | iShares Core MSCI World UCITS ETF, ISIN IE00B4L5Y983, XETRA, EUR (wie im API-Design); synthetische Kursreihe über 10 Jahre | Normalfälle Suche, Portfolio, Simulation |
| Anlage IS3N (`assetId` 23) | iShares Core MSCI EM IMI (wie im API-Design); synthetische Kursreihe über 10 Jahre | Zweite Anlage für Portfolio 70 % / 30 % |
| Zwei Anlagen mit kurzer Historie | je eine Kursreihe knapp unter 3 Jahren | Alternativszenario < 3 Jahre mit zwei betroffenen Anlagen (AT-04-08) und Grenzwert (AT-04-09) |
| Anlage mit genau 3 Jahren Historie | Kursreihe mit genau 36 Monatswerten (E8); die Anlagen mit kurzer Historie haben 35 Monatswerte | Grenzwert (AT-04-09) |
| Anlage mit konstantem Wachstum | Kursreihe ohne Schwankung | Test-Orakel: Volatilität 0, bekannte Rendite |
| 5 Anlagen mit 10 Jahren Historie | synthetische Kursreihen | Performancetest NF-01 mit 5 Anlagen (Messfall, AT-04-14) |
| Mehr als 25 Suchtreffer | mindestens 26 synthetische Anlagen, deren Name „MSCI“ enthält | Grenzwerte des Suchparameters `limit` (AT-02-05) |
| Beispielportfolios | „Mein Weltportfolio“ (EUNL 70 %, IS3N 30 %); Einzelanlage 100 %; Portfolio aus 5 Anlagen à 20 % | Simulation, Performance |
| Fester Seed | `42` (wie im API-Design) | Reproduzierbare Ergebnisse |

**Testnutzer (ab Sprint 2):**

| Testdaten | Inhalt | Zweck |
|---|---|---|
| Nutzer A | `anna@example.com`, Passwort `geheim123` (wie im API-Design); besitzt „Mein Weltportfolio“ | Normalfälle US-01 bis US-04 |
| Nutzer B | `ben@example.com`, Passwort `sicher456`; ohne Portfolio | Zugriffsschutz: B darf die Daten von A nicht sehen (NF-04) |
| Nutzer C | `carla@example.com`, gleiches Passwort wie Nutzer A | Nachweis des Salzes: gleiches Passwort ergibt unterschiedliche Hashes (NF-03) |
| Abgelaufener Token | Token mit `expiresAt` in der Vergangenheit (im Unit-Test über die vorgegebene Uhr) | Ablauf der Anmeldung (NF-16, E14) |

Die Testnutzer werden über `POST /auth/register` angelegt, nicht direkt in die Datenbank geschrieben, damit der Passwort-Hash auf dem regulären Weg entsteht.

*(Geklärt in Version 3:)* Die Anlagen mit 35 und 36 Monatswerten sind in `ParameterEstimatorTest` als feste Kursreihen der Mockito-Mocks angelegt (`genau36MonatswerteReichen`, `fuenfunddreissigMonatswerteLiefern422`); ein `MockMarketDataProvider` wird dafür nicht benötigt.

**Stand Version 3 (Sprint 3).** Einen `MockMarketDataProvider` mit festen Kursreihen und Ausfall-Schalter gibt es im Code nicht (BE #53 wurde ohne entsprechende Klasse geschlossen). Die Unit-Tests ersetzen den `MarketDataProvider` stattdessen durch Mockito-Mocks mit festen Kursreihen; damit bleiben die automatisierten Tests reproduzierbar und laufen ohne Internet (NF-12). Die manuellen Abnahmetests laufen auf Heroku mit der echten Twelve-Data-API. Weil der kostenlose Tarif keine Kurse für Xetra liefert (F3), gelten für die manuellen Testfälle ab Sprint 3 Anlagen an US-Börsen (PO-Entscheidung vom 18.10.2026):

| Testdaten | Inhalt | Zweck |
|---|---|---|
| „Mein Weltportfolio“ (Nutzer A) | URTH 70 %, IEMG 30 % (NYSE Arca) | Basisportfolio der manuellen Tests; ersetzt EUNL 70 % / IS3N 30 % |
| „Mein Aktienportfolio“ (Nutzer A) | AAPL 50 %, MSFT 50 % (NASDAQ) | zweites Portfolio für den Vergleich (US-07) und das Bearbeiten (US-06) |
| „Krypto-Test“ (Nutzer A) | IBIT 50 %, ETHA 50 % (NASDAQ, Handel seit 2024) | zwei Anlagen mit weniger als 36 Monatswerten (E6, E8); bis Ende 2026 gültig |
| „Messportfolio“ (Nutzer A) | URTH, IEMG, AAPL, MSFT, NVDA je 20 % | Messfall mit 5 Anlagen (NF-01, E33) |
| EUNL (XETR) | über die Anlagensuche | Anlage ohne Kurse beim Anbieter (Docs #80) |
| Zielbetrag | 100.000 € (wie im API-Design) | Zielwahrscheinlichkeit (US-05, E27) |
| Gespeicherte Simulationen | „Sparplan 20 Jahre“ und „Kurz 10 Jahre“ von Nutzer A | US-08, Zugriffsschutz mit Nutzer B (NF-04) |

Die Tabellen oben mit EUNL, IS3N und den Testanlagen des Mocks bleiben für die automatisierten Tests und als Stand von Sprint 1 und 2 gültig.

**Begründung.** Feste Kursreihen und ein fester Seed sorgen dafür, dass jeder Testlauf dieselben Ergebnisse liefert (NF-08) und dass Testergebnisse nicht von aktuellen Marktdaten abhängen. Die Testdaten bleiben auch nach der Auswahl von Twelve Data als primärem Anbieter synthetisch. Weil der Mock keine Verbindung zur Finanzdaten-API benötigt, laufen die Tests ohne Internetzugang und ohne API-Key (NF-12, NF-06) und verbrauchen kein Kontingent der kostenlosen API.

## 7. Testumgebung

| Umgebung | Was läuft dort | Wann |
|---|---|---|
| **Lokal (Entwickler)** | Backend: Unit- und Integrationstests mit `mvnw test` bzw. `mvnw.cmd test`; Frontend: `npm test` (Node-Test-Runner) | Vor jedem Commit bzw. vor dem Öffnen eines Pull Requests |
| **CI-Pipeline (GitHub Actions)** | Backend: Build, Unit- und Integrationstests, Abdeckungsbericht mit JaCoCo; Frontend: Tests mit dem Node-Test-Runner. Ohne Internetzugriff auf die Finanzdaten-API und ohne API-Key. | Automatisch bei jedem Pull Request auf `main`; ein Merge ist nur bei erfolgreicher Pipeline möglich (CONTRIBUTING.md) |
| **Abnahmeumgebung** | Vollständige Anwendung (Frontend, Backend, PostgreSQL) auf Heroku; ab Sprint 2 mit der echten Twelve-Data-API, da kein `MockMarketDataProvider` vorhanden ist (siehe Abschnitt 6). Für NF-03 und NF-14 (b) ist lesender Zugriff auf die Datenbank nötig (Heroku Dataclip bzw. Postgres-Konsole). | Vor dem Sprint Review |

Für manuell durchgeführte Alternativszenarien müsste die Abnahmeumgebung einen Ausfall der Finanzdaten-API darstellen können. Solange es dafür keinen Schalter gibt, werden der API-Ausfall (AT-02-07, AT-04-06, AT-07-11), das Cache-Alter und zu kurze Überschneidungen von Kurshistorien (AT-07-10) nur automatisiert getestet. *(Stand Version 3:)* Ein solcher Schalter ist nicht umgesetzt; die Abnahmeumgebung läuft auf Heroku (Docs #64).

## 8. Ein- und Ausgangskriterien

**Eingangskriterien für den Abnahmetest einer Story**

- Die Umsetzungs-Tasks der Story sind gemergt; die CI-Pipeline ist grün.
- Die Abnahmetestfälle der Story sind dokumentiert und vom Product Owner gesichtet.
- Die Abnahmeumgebung läuft mit den Testdaten aus Abschnitt 6.

**Ausgangskriterien (Story gilt als abgenommen)**

Eine Story gilt als abgenommen, wenn alle ihre Abnahmetestfälle den Status `bestanden` haben und kein offener Fehler mit Priorität „hoch“ besteht. Damit ist der Punkt „die Abnahmetestfälle dokumentiert und bestanden sind“ der Definition of Done erfüllt. Die übrigen Punkte der Definition of Done (Code-Review, Unit-Tests in der CI, aktualisierte Diagramme, KI-Log) gelten unverändert.

Ein automatisiert durchgeführter Abnahmetestfall gilt als `bestanden`, wenn alle ihm zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren.

Für die Kernlogik gilt zusätzlich der Zielwert der Testabdeckung aus Abschnitt 9 (NF-11).

## 9. Qualitätsmetrik

**Metrik:** Zeilenabdeckung (Line Coverage) der Kernlogik, gemessen mit JaCoCo.

**Zielwert:** mindestens 80 % für die Kernlogik (NF-11), d. h. für die Klassen der Simulation (`ParameterEstimator`, `MonteCarloEngine`, `MetricsCalculator`, `SimulationService`), des Vergleichs (`ComparisonService`, ab Version 3) und der Portfolio-Prüfung (`Portfolio`, `PortfolioService`).

**Messung:** JaCoCo erzeugt bei jedem Lauf der Backend-CI-Pipeline einen Abdeckungsbericht. Die Auswertung und Interpretation erfolgt in Docs #24 und wird im Sprint Review vorgestellt. Zusätzlich wird die Zweigabdeckung (Branch Coverage) mit ausgewiesen, weil die Prüfregeln viele Verzweigungen enthalten.

**Interpretation und Grenzen:**

- Eine hohe Abdeckung zeigt nur, dass Code ausgeführt wurde, nicht dass das Ergebnis geprüft wurde. Ein Test ohne aussagekräftige Assertions erhöht die Abdeckung, findet aber keine Fehler. Deshalb wird im Code-Review darauf geachtet, dass jeder Test erwartete Werte prüft.
- Fehlende Anforderungen erkennt die Metrik nicht: Code, der nie geschrieben wurde, kann nicht unabgedeckt sein. Diese Lücke schließen die Abnahmetestfälle, die über die Spalte „Bezug“ jedem Akzeptanzkriterium zugeordnet sind.
- Der Gesamtwert wird nach Klassen aufgeschlüsselt betrachtet. Ein hoher Durchschnitt kann eine schwach getestete Kernklasse verdecken.
- Konfigurations- und reine Datenklassen werden nicht in den Zielwert einbezogen, damit die Metrik die Kernlogik widerspiegelt.

## 10. Nachweis nicht-funktionaler Anforderungen

| NFR | Anforderung | Art des Nachweises | Testfall-ID oder Prüfmethode | Sprint |
|---|---|---|---|---|
| NF-01 | Performance der Simulation: 10.000 Pfade, mit 5 Anlagen (Messfall), Horizont 40 Jahre, serverseitig ≤ 3 s in 95 % von 20 Messläufen; Vergleich zweier Einträge ≤ 6 s (E33) | Performancetest (Messung von `runtimeMs`) | AT-04-14; Vergleich AT-07-13 | 1 / 3 |
| NF-02 | Performance der Suche: ≤ 1 s mit Cache-Treffer (NFRs Version 2) | Zeitmessung im Abnahmetest | AT-02-08 | 1 |
| NF-03 | Passwörter nur als gesalzener Hash (BCrypt), mindestens 8 Zeichen | Unit-Test und Prüfung der Datenbankinhalte | AT-01-08, AT-01-11 | 2 |
| NF-04 | Zugriff auf fremde Ressourcen liefert 404; Listen zeigen nur eigene Einträge | Integrations- und Abnahmetest | AT-01-13, AT-01-14; ab Sprint 3 AT-06-06 (`PUT`/`DELETE`), AT-07-06 (Vergleich), AT-08-06 (gespeicherte Simulationen) | 2 / 3 |
| NF-05 | Nur HTTPS erreichbar | Manuelle Prüfung: Aufruf per `http://` wird nicht unverschlüsselt beantwortet | Prüfung nach dem Deployment (Docs #64) | ab Deployment |
| NF-06 | API-Key nur als Umgebungsvariable im Backend | Review: kein Key in Repositories und im ausgelieferten Frontend-Code; `.env` in `.gitignore`; CI-Tests laufen ohne Key | Checkliste im Code-Review; NF-12-Prüfung | 1 |
| NF-07 | Bei API-Ausfall Antwort ≤ 5 s mit verständlicher Meldung (HTTP 503); Kursdaten jünger als 24 h werden weiterverwendet | Abnahmetest mit simuliertem Ausfall; Unit-Test des Cache-Fallbacks | AT-02-07, AT-04-06, AT-04-07, AT-07-11 | 1 / 3 |
| NF-08 | Gleicher Seed und gleiche Eingaben ergeben identische Perzentile | Automatisierter Unit-Test (`MonteCarloEngine`, `MetricsCalculator`, `ComparisonService`, `PriceHistoryService`) | AT-04-13, AT-05-06, AT-07-03 | 1 / 3 |
| NF-09 | Mindestens 2 von 3 Testpersonen erstellen ohne Hilfe in < 5 min ein Portfolio und simulieren es | Bedienbarkeitstest mit Testpersonen ohne Vorkenntnisse | AT-NF09-01; Ablauf unter der Tabelle | 3 |
| NF-10 | Alle Seiten ab 360 px Breite ohne horizontales Scrollen bedienbar | Manuelle Prüfung im Browser bei 360 px Breite | Sichtprüfung aller Seiten im Abnahmetest; Sprint 3: AT-NF10-01 | ab 1 (je neue Seite) |
| NF-11 | Zeilenabdeckung der Kernlogik ≥ 80 % in der CI | JaCoCo-Bericht | Abschnitt 9, Docs #24 | ab 1 |
| NF-12 | Tests laufen mit `MockMarketDataProvider` ohne Internetzugang | CI-Lauf ohne API-Key und ohne Zugriff auf die Finanzdaten-API | grüner CI-Lauf | ab 1 |
| NF-13 | Start aller Komponenten mit `docker compose up` | Manueller Start in sauberer Umgebung | Prüfung vor dem Sprint Review [OFFEN: Anforderung an das Deployment bei Umstieg auf Heroku neu formulieren; Prüfung mit Docs #64] | ab Deployment |
| NF-14 | (a) Hinweis „Keine Anlageberatung“ auf jeder Ergebnisseite; (b) Nutzerkonto speichert nur E-Mail-Adresse und Passwort-Hash | Abnahmetest; Prüfung der Datenbank | (a) AT-04-15; (b) AT-01-12 | 1 / 2 |
| NF-15 | Schnittstelle vollständig in `api/` beschrieben | Abgleich der Integrationstests und Abnahmetests mit dem API-Design | Review bei jeder Schnittstellenänderung | ab 1 |
| NF-16 | Anmeldung 24 h gültig; abgelaufener oder nach dem Abmelden gesperrter Token liefert 401 | Unit-Test mit vorgegebener Uhr; Abnahmetest | AT-01-02, AT-01-03, AT-01-06, AT-01-10 | 2 |

**Ablauf des Bedienbarkeitstests (NF-09, ab Version 3).**

1. **Testpersonen:** 3 Personen ohne Vorkenntnisse zu LookUpStock und ohne Finanz- oder Informatikstudium; der Product Owner wählt sie aus.
2. **Vorbereitung:** Je Testperson ein eigenes, bereits angemeldetes Testkonto ohne Portfolio; Start auf der Portfolioseite im Browser auf einem Laptop.
3. **Aufgabe (schriftlich vorgelegt):** „Lege ein Portfolio aus zwei Anlagen an und starte eine Simulation.“ Welche Anlagen gewählt werden, ist der Testperson überlassen; Anlagen an US-Börsen funktionieren mit dem aktuellen Tarif.
4. **Durchführung:** Der Product Owner beobachtet, gibt keine Hilfe und misst die Zeit vom ersten Klick bis zum angezeigten Fächerdiagramm. Notiert werden die Zeit, Stellen, an denen die Testperson zögert oder nachfragt, und Fehlermeldungen.
5. **Bewertung:** Bestanden, wenn mindestens 2 von 3 Testpersonen die Aufgabe ohne Hilfe in höchstens 5 Minuten lösen. Beobachtete Probleme werden auch bei bestandenem Test als Verbesserungsvorschläge ins Product Backlog aufgenommen.
6. **Termin:** im Rahmen der Abnahme Sprint 3 (Docs #101); das Ergebnis steht im Abnahmeprotokoll Sprint 3.

## 11. Fehlermanagement

1. Ein Fehler, der bei einem Test gefunden wird, wird als GitHub-Issue mit dem Label `bug` im betroffenen Repository (BE, FE oder Docs) angelegt.
2. Das Issue enthält die Testfall-ID (z. B. AT-04-10), die Schritte zur Reproduktion, das erwartete und das tatsächliche Ergebnis und, falls hilfreich, einen Screenshot.
3. Die Priorität wird vom Product Owner festgelegt:
   - **hoch:** Ein Akzeptanzkriterium ist nicht erfüllt oder die Anwendung stürzt ab. Die Story kann nicht abgenommen werden.
   - **mittel:** Ein Akzeptanzkriterium ist erfüllt, aber z. B. eine Meldung ist missverständlich.
   - **niedrig:** Kosmetische Mängel.
4. Die Behebung erfolgt auf einem Branch `fix/<issue-nr>-<kurzbeschreibung>`. Wo möglich, wird zuerst ein automatisierter Test ergänzt, der den Fehler zeigt.
5. Nach dem Merge wird der betroffene Abnahmetestfall erneut durchgeführt und sein Status aktualisiert. Erst dann wird das Issue geschlossen.

## 12. Rollen und Verantwortlichkeiten

| Rolle / Person | Aufgaben im Test |
|---|---|
| Product Owner – Mert Eminel (KujoBaraka) | Sichtet die Abnahmetestfälle, führt die manuellen Abnahmetests durch, setzt den Status, priorisiert gefundene Fehler, nimmt Stories ab; ab Sprint 2: Abnahmetestfälle und Fortschreibung des Testkonzepts (Docs #61), Abnahme (Docs #62), Frontend-Tests zur Login-Maske (FE #4); ab Sprint 3: Abnahmetestfälle und Testkonzept v3 (Docs #100), Abnahme mit Nachtest und Bedienbarkeitstest (Docs #101), Frontend-Tests zu gespeicherten Simulationen (FE #17) |
| Scrum Master – Carl Stibbe (carlstibbe) | Achtet auf die Einhaltung der Definition of Done; schreibt Tests zu eigenen Umsetzungs-Tasks (z. B. BE #39, FE #2, FE #3); ab Sprint 2 `MockMarketDataProvider` mit Testdaten (BE #53) und Abnahmeumgebung auf Heroku (Docs #64) |
| Entwickler – Peter Orban (Pixxels02) | Unit-Tests für Portfolio, Suche und Simulationslogik (BE #32, BE #38, BE #41); CI-Pipelines; Messung und Interpretation der Testabdeckung (Docs #24); ab Sprint 3 Tests zu Kennzahlen, Suchcache und Speichern (BE #71, BE #77, BE #68), Testabdeckung und Pfadabdeckung US-07 (Docs #102) |
| Entwickler – Paul Kaufmann (Paul02112004) | Testkonzept und Abnahmetestfälle (Docs #23); Tests zu eigenen Endpunkten (BE #27, BE #33; ab Sprint 2 BE #30, BE #50, BE #54; ab Sprint 3 BE #69, BE #70, F3, F4); unterstützt den Product Owner bei der Abnahme |
| Alle Entwickler | Schreiben Unit- und Integrationstests zu ihrem Code, prüfen Tests im Code-Review, beheben Fehler aus Bug-Issues |

## 13. Fortschreibung

Das Testkonzept wird zu Beginn jedes Sprints überprüft und bei Bedarf ergänzt. Für jeden Sprint entsteht eine eigene Datei `abnahmetestfaelle-sprint-<n>.md`. Absehbar sind:

- **Sprint 2 (umgesetzt in Version 2):** Abnahme- und Integrationstests für die Anmeldung (US-01) einschließlich NF-03, NF-04, NF-14 (b) und NF-16; Vorbedingung „Nutzer ist angemeldet“ für US-03 und US-04 mit Testfällen ohne gültigen Token (`401 UNAUTHORIZED`); Korrekturen an den übernommenen Testfällen aus Sprint 1. Die Tests für die Kennzahlen (US-05) entfallen in Sprint 2, weil US-05 nach Sprint 3 verschoben wurde.
- **Sprint 3 (umgesetzt in Version 3):** US-07 als zweite Kern-Story mit Unit-Tests, Abnahmetestfällen und Ableitung aus ihrem Aktivitätsdiagramm (Auswertung der Pfadabdeckung in Docs #102); Speichern von Simulationen (US-08); Kennzahlen (US-05, Feld `metrics`); Bearbeiten und Löschen von Portfolios (US-06) einschließlich `PUT` und `DELETE` mit fremden IDs (NF-04); Ablauf des Bedienbarkeitstests (NF-09); Basisportfolio der manuellen Tests auf US-Börsen; Nachtest der Befunde F1 bis F7.
- **Sprint 4:** Beispielsimulation für Gäste (US-09), Vergleich mit bis zu vier Einträgen (US-10), Speichern von Vergleichen (E34); mögliche Änderungen durch den Change Request.
- **Laufend:** Nachweis der übrigen NFRs (NF-05, NF-13) laut Tabelle in Abschnitt 10; Ergänzung von Tests, sobald ein Fallback-Anbieter zu Twelve Data entschieden ist.

## Änderungshistorie

| Version | Datum | Abschnitt | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 02.10.2026 | alle | Erstfassung | Docs #23 |
| 2 | 08.10.2026 | Kopf | Sprint 2, Verweis auf `abnahmetestfaelle-sprint-2.md`, Grundlagen aktualisiert | Docs #61 |
| 2 | 08.10.2026 | 2, 3 | Testziel Zugriffsschutz; Testobjekte `AuthService`, `AuthController`, Token-Prüfung und Login-Maske; offene Frage zum `MetricsCalculator` durch Klassendiagramm v1 geklärt | US-01, BE #30 |
| 2 | 08.10.2026 | 4, 7 | Frontend-Tests mit dem Node-Test-Runner statt Jest | Abgleich mit dem Frontend-Repository (FE #6) |
| 2 | 08.10.2026 | 5 | Aktivitätsdiagramm v2 mit Cache-Zweig; Kurshistorie als 36 Monatswerte; neuer Absatz „Zeitabhängige Tests“ | Aktivitätsdiagramm v2, E8, NF-16 |
| 2 | 08.10.2026 | 6 | Testnutzer A, B und C; Anlagen mit 35 und 36 Monatswerten | US-01, NF-03, NF-04, E8 |
| 2 | 08.10.2026 | 10 | NF-01 Horizont 40 Jahre, NF-02 nur Cache-Fall, NF-09 2 von 3 Testpersonen (Abgleich mit NFRs Version 2); Testfälle für NF-03, NF-04, NF-14 (b) und NF-16 eingetragen; Docs #29 durch Docs #64 ersetzt | NFRs Version 2, US-01, Sprint-Planning-Protokoll Sprint 2 |
| 2 | 08.10.2026 | 12, 13 | Rollen um Sprint-2-Aufgaben ergänzt; Fortschreibung: US-05 und US-06 nach Sprint 3 | Sprint-Planning-Protokoll Sprint 2 |
| 3 | 18.10.2026 | Kopf | Sprint 3, Verweis auf `abnahmetestfaelle-sprint-3.md`, Grundlagen aktualisiert | Docs #100 |
| 3 | 18.10.2026 | 2, 3, 4 | Testziele um Kennzahlen, Vergleich und Ändern/Löschen ergänzt; Testobjekte `ComparisonService`, `SavedSimulationService`, `MetricsCalculator.metrics()`, `PriceHistoryService`/`PriceCache` statt Cache im `MarketDataProvider`, Frontend ab Sprint 3; Teststufen mit Sprint-3-Tasks | US-05 bis US-08, ADR-05, Code-Stand 18.10.2026 |
| 3 | 18.10.2026 | 5 | Ableitung der Testfälle aus dem Aktivitätsdiagramm US-07 v1 mit Zuordnung aller Verzweigungen | Kern-Story US-07, Docs #90 |
| 3 | 18.10.2026 | 6, 7 | Kein `MockMarketDataProvider` im Code: Unit-Tests mit Mockito-Mocks, Abnahme auf Heroku mit Twelve Data; Testdaten der manuellen Tests auf US-Börsen umgestellt | Abnahmeprotokoll Sprint 2 (F3), PO-Entscheidung vom 18.10.2026 |
| 3 | 18.10.2026 | 9 | `ComparisonService` als Klasse der Kern-Story US-07 in die Kernlogik für NF-11 aufgenommen | ADR-05 |
| 3 | 18.10.2026 | 10 | Testfälle für NF-01 (Vergleich, E33), NF-04, NF-07, NF-08 und NF-10 ergänzt; Ablauf des Bedienbarkeitstests (NF-09) festgelegt | Docs #100, NF-09 |
| 3 | 18.10.2026 | 12, 13 | Rollen um Sprint-3-Aufgaben ergänzt; Fortschreibung: Sprint 3 umgesetzt, Ausblick Sprint 4 | Sprint-Planning-Protokoll Sprint 3 |
