# Testkonzept – LookUpStock

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026), Version 1 |
| **Issue** | Docs #23 Testkonzept v1 und Abnahmetestfälle |
| **Verantwortlich** | Paul Kaufmann (Paul02112004) |
| **Stand** | 02.10.2026 |
| **Grundlagen** | Kurzformat-Tabellen Sprint 1 mit Akzeptanzkriterien (Stand 30.09.2026), Entscheidungen des Product Owners E1 bis E7 (02.10.2026), API-Design (Stand 02.10.2026), ADR-01 bis ADR-03, Aktivitätsdiagramm US-04 v1, Domänenmodell v1, Sprint-Planning-Protokoll Sprint 1, CONTRIBUTING.md |

Die Abnahmetestfälle für Sprint 1 stehen in [abnahmetestfaelle-sprint-1.md](abnahmetestfaelle-sprint-1.md).

---

## 1. Zweck und Geltungsbereich

Dieses Testkonzept legt fest, wie LookUpStock getestet wird: welche Komponenten mit welchen Methoden, auf welchen Teststufen, mit welchen Daten und Werkzeugen geprüft werden, wann eine User Story als abgenommen gilt und wie gefundene Fehler behandelt werden.

Es gilt für das gesamte Projekt, also für das Backend (`Look-Up-Stock-BackEnd`), das Frontend (`Look-Up-Stock-FrontEnd`) und die Abnahme der User Stories durch den Product Owner. Das Dokument wird in jedem Sprint fortgeschrieben (siehe Abschnitt 13). Die konkreten Abnahmetestfälle werden je Sprint in einer eigenen Datei geführt.

## 2. Testziele

1. **Korrektheit der Kernlogik:** Die Portfolio-Prüfung (US-03) und die Monte-Carlo-Simulation (US-04) liefern für gültige Eingaben korrekte Ergebnisse und weisen ungültige Eingaben zuverlässig zurück.
2. **Erfüllung der Akzeptanzkriterien:** Jedes Akzeptanzkriterium der Stories eines Sprints ist durch mindestens einen Abnahmetestfall abgedeckt und nachweislich erfüllt.
3. **Nachweis der nicht-funktionalen Anforderungen:** Die operationalisierten NFRs (insbesondere NF-01, NF-07 und NF-08) werden durch Tests oder definierte Prüfungen belegt.
4. **Schutz vor Regressionen:** Automatisierte Tests laufen bei jedem Pull Request in der CI-Pipeline, sodass Änderungen bestehende Funktionen nicht unbemerkt beschädigen.

## 3. Testobjekte

Der Schwerpunkt liegt auf der Kernlogik der Kern-Story US-04 und der Portfolio-Prüfung aus US-03. Die Klassennamen stammen aus ADR-02 und ADR-03.

| Komponente | Schicht (ADR-01) | Story | Was geprüft wird |
|---|---|---|---|
| `Portfolio` mit `validate()` und `getWeights()` | Logik | US-03, US-04 | Prüfregeln: Name Pflicht mit höchstens 100 Zeichen, mindestens eine Anlage (keine Höchstzahl), keine doppelten Anlagen, jede Gewichtung > 0 % mit höchstens 2 Nachkommastellen, Summe 100 % ± 0,01 Prozentpunkte bei exakter Dezimalrechnung; Umrechnung der Gewichtungen in Anteile in fester Reihenfolge (ADR-03) |
| `ParameterEstimator` | Logik | US-04 | Schätzung von Jahresrendite und Jahresvolatilität aus Kursreihen; Erkennung von weniger als 3 Jahren Kurshistorie |
| `MonteCarloEngine` | Logik | US-04 | Pfadanzahl, Monatsschritte über den Anlagehorizont, Berücksichtigung von Startkapital und Sparrate, Reproduzierbarkeit bei gleichem Seed |
| `MetricsCalculator` | Logik | US-04 | Aufbereitung des Ergebnisses, insbesondere 5., 50. und 95. Perzentil je Monat [OFFEN: Ob die Perzentile im `MetricsCalculator` oder in der `MonteCarloEngine` berechnet werden, legt das Klassendiagramm v1 (Docs #19) fest, das noch nicht im Repository liegt.] |
| `SimulationService` (GRASP-Controller, ADR-02) | Logik | US-04 | Koordination der Schritte in der richtigen Reihenfolge, Seed-Erzeugung ohne Angabe, Fehlerfälle (Portfolio fehlt, Kursdaten nicht verfügbar, zu kurze Historie) |
| `PortfolioService` | Logik | US-03 | Aufruf von `validate()` vor dem Speichern; Ablehnung unbekannter `assetId` |
| `MarketDataProvider` mit Cache und Fallback | Daten | US-02, US-04 | Verhalten bei Ausfall der Finanzdaten-API, Weiterverwendung von Kursdaten jünger als 24 h (NF-07). Primärer Anbieter im Betrieb ist Twelve Data; ein Fallback-Anbieter ist noch nicht entschieden. In allen Tests wird der `MockMarketDataProvider` verwendet. |
| REST-Controller (`SimulationController`, Such- und Portfolio-Endpunkte) | Präsentation | US-02, US-03, US-04 | HTTP-Status, Fehlercodes und Fehlerformat laut API-Design |
| Frontend-Prüflogik und Darstellung | Präsentation | US-02, US-03, US-04 | Mindestlänge des Suchbegriffs, Gewichtungssumme, Formatierung, Fehlermeldungen, Fächerdiagramm |

## 4. Teststufen

| Teststufe | Ziel | Werkzeug | Verantwortlich | Zeitpunkt |
|---|---|---|---|---|
| **Unit-Tests** | Kernlogik isoliert prüfen, ohne Datenbank, ohne Web-Server und ohne Internet. Abhängigkeiten werden durch Mocks ersetzt. | Backend: JUnit 5, Mockito<br>Frontend: Jest | Entwickler der jeweiligen Komponente; Unit-Tests der Kernlogik laut Sprint Backlog: BE #32 (Portfolio), BE #38 (Suche), BE #41 (Simulationslogik) | Während der Umsetzung, vor dem Pull Request; automatisch in der CI bei jedem Pull Request |
| **Integrationstests** | REST-Endpunkte einschließlich Validierung, Fehlerformat und Zusammenspiel Controller → Service prüfen; Marktdaten kommen aus dem `MockMarketDataProvider`. | Spring Boot Test mit `@WebMvcTest` und `MockMvc` | Entwickler des jeweiligen Endpunkts (BE #27, BE #33, BE #39/#40) | Mit Fertigstellung eines Endpunkts; automatisch in der CI bei jedem Pull Request |
| **Abnahmetests** | Prüfung aus Nutzersicht gegen die Akzeptanzkriterien, die Alternativszenarien des Aktivitätsdiagramms und relevante Grenzwerte | Browser (Oberfläche) und ein HTTP-Client für direkte API-Aufrufe; automatisierte Tests für die als „automatisiert“ gekennzeichneten Testfälle; Testfälle in `abnahmetestfaelle-sprint-<n>.md` | Product Owner (KujoBaraka), unterstützt von den Entwicklern | Nach Fertigstellung einer Story, spätestens vor dem Sprint Review |
| **Nicht-funktionale Tests** | Nachweis der NFRs, in Sprint 1 vor allem Performance der Simulation (NF-01), Ausfallsicherheit (NF-07) und Reproduzierbarkeit (NF-08) | NF-01: wiederholte Messung über das Feld `runtimeMs` der Simulationsantwort; NF-08: JUnit-Test; übrige siehe Abschnitt 10 | Entwickler (Messung), Product Owner (Abnahme) | NF-08 und NF-12 automatisch in der CI; NF-01 und NF-07 vor dem Sprint Review |

**Durchführung der Abnahmetestfälle.** Grenzwerttestfälle, reine API-Fälle und Fälle, die einen gezielt manipulierten Mock erfordern, werden als automatisierte Unit- oder Integrationstests umgesetzt und bei jedem Pull Request in der CI-Pipeline ausgeführt. Manuelle Abnahmetests durch den Product Owner beschränken sich auf Normalfälle und Alternativszenarien aus Nutzersicht sowie auf Messungen, die nicht in der CI laufen können (NF-01, NF-02). Das senkt den manuellen Aufwand pro Sprint, macht die Ausführung reproduzierbar und erhöht die Testabdeckung (NF-11). Welche Durchführung für einen Testfall gilt, steht in der Spalte „Durchführung“ der Abnahmetestfälle. Grenzwerte desselben Eingabefelds werden in einem Testfall zusammengefasst und als parametrisierter Test umgesetzt.

## 5. Testmethoden

**Äquivalenzklassen.** Jedes Eingabefeld wird in Klassen gültiger und ungültiger Werte zerlegt, die sich gleich verhalten sollen. Aus jeder Klasse wird mindestens ein Vertreter getestet. Beispiel Anlagehorizont: gültig sind ganze Zahlen von 1 bis 40; ungültig sind Werte kleiner 1, Werte größer 40, Kommazahlen und ein leeres Feld.

**Grenzwertanalyse.** An jeder Grenze wird der Wert knapp darunter, genau auf der Grenze und knapp darüber getestet, weil Fehler wie `<` statt `<=` dort auftreten. Beispiele: Pfadanzahl 999, 1.000, 50.000 und 50.001; Gewichtungssumme 99,98 %, 99,99 %, 100,01 % und 100,02 %.

Exakte Dezimalrechnung: Die Grenzwerte der Gewichtungssumme liegen genau auf der Toleranz von ± 0,01 Prozentpunkten. Bei Rechnung mit `double` können Summen mit Nachkommastellen nicht exakt dargestellt werden; so ergibt die Prüfung `abs(70 + 30.01 - 100) <= 0.01` in `double` den Wert `false`, obwohl die Summe 100,01 % gültig ist. Deshalb ist festgelegt, dass das Backend Gewichtungen als exakte Dezimalzahlen (`BigDecimal`) verarbeitet und Gewichtungen höchstens 2 Nachkommastellen haben (PO-Entscheidung E2). Nachgewiesen wird das durch die Grenzwerte der Gewichtungssumme (AT-03-03, darunter 70 % + 30,01 %) und den Gleitkomma-Testfall in AT-03-05 (69,99 % + 30,01 % wird als 100,00 % akzeptiert). Beide werden als Unit-Tests von `Portfolio.validate()` umgesetzt.

**Ableitung aus dem Aktivitätsdiagramm.** Für die Kern-Story US-04 wird aus dem Aktivitätsdiagramm (`diagramme/aktivitaetsdiagramm_us-04_v1.puml`) für jede Verzweigung mindestens ein Testfall abgeleitet, sodass jeder Pfad vom Start bis zu einem Endknoten mindestens einmal durchlaufen wird (Pfadabdeckung der Alternativszenarien):

| Verzweigung im Aktivitätsdiagramm | Pfade | Abnahmetestfälle |
|---|---|---|
| Eingaben prüfen | gültig / ungültig (Rücksprung zur Eingabe, HTTP 400) | AT-04-01, AT-04-04, AT-04-10 bis AT-04-12 |
| Portfolio laden | vorhanden / nicht vorhanden (HTTP 404) | AT-04-01, AT-04-05 |
| Seed | ohne Seed (System erzeugt) / mit Seed | AT-04-01, AT-04-02 |
| Kursdaten | verfügbar / nicht verfügbar (HTTP 503) | AT-04-01, AT-04-06 |
| Kurshistorie je Anlage | ≥ 3 Jahre / < 3 Jahre (HTTP 422) | AT-04-08, AT-04-09 |

Der Cache-Fallback aus NF-07 ist im Aktivitätsdiagramm v1 nicht dargestellt; er wird zusätzlich durch AT-04-07 abgedeckt.

**Test-Orakel für die Simulation.** Weil Simulationsergebnisse zufallsabhängig sind, werden in Unit-Tests Fälle mit bekanntem Ergebnis verwendet: Bei einer Volatilität von 0 sind alle Pfade gleich und der Endwert lässt sich aus Startkapital, Sparrate und Rendite exakt berechnen. Zusätzlich werden Eigenschaften geprüft, die immer gelten müssen, z. B. 5. Perzentil ≤ 50. Perzentil ≤ 95. Perzentil in jedem Monat.

## 6. Testdaten

Alle automatisierten Tests verwenden den `MockMarketDataProvider`, der das Interface `MarketDataProvider` mit festen, synthetischen Kursreihen implementiert. Er wird auch für die Abnahmetests verwendet, damit Ergebnisse und Fehlerfälle gezielt herbeigeführt werden können.

| Testdaten | Inhalt | Zweck |
|---|---|---|
| Anlage EUNL (`assetId` 17) | iShares Core MSCI World UCITS ETF, ISIN IE00B4L5Y983, XETRA, EUR (wie im API-Design); synthetische Kursreihe über 10 Jahre | Normalfälle Suche, Portfolio, Simulation |
| Anlage IS3N (`assetId` 23) | iShares Core MSCI EM IMI (wie im API-Design); synthetische Kursreihe über 10 Jahre | Zweite Anlage für Portfolio 70 % / 30 % |
| Zwei Anlagen mit kurzer Historie | je eine Kursreihe knapp unter 3 Jahren | Alternativszenario < 3 Jahre mit zwei betroffenen Anlagen (AT-04-08) und Grenzwert (AT-04-09) |
| Anlage mit genau 3 Jahren Historie | Kursreihe von genau 3 Jahren | Grenzwert (AT-04-09) |
| Anlage mit konstantem Wachstum | Kursreihe ohne Schwankung | Test-Orakel: Volatilität 0, bekannte Rendite |
| 5 Anlagen mit 10 Jahren Historie | synthetische Kursreihen | Performancetest NF-01 mit 5 Anlagen (Messfall, AT-04-14) |
| Mehr als 25 Suchtreffer | mindestens 26 synthetische Anlagen, deren Name „MSCI“ enthält | Grenzwerte des Suchparameters `limit` (AT-02-05) |
| Beispielportfolios | „Mein Weltportfolio“ (EUNL 70 %, IS3N 30 %); Einzelanlage 100 %; Portfolio aus 5 Anlagen à 20 % | Simulation, Performance |
| Fester Seed | `42` (wie im API-Design) | Reproduzierbare Ergebnisse |

[OFFEN: Symbol, `assetId` und Zeitraum der Testanlagen mit kurzer und genau 3-jähriger Historie werden bei der Umsetzung des `MockMarketDataProvider` festgelegt und hier nachgetragen.]

**Begründung.** Feste Kursreihen und ein fester Seed sorgen dafür, dass jeder Testlauf dieselben Ergebnisse liefert (NF-08) und dass Testergebnisse nicht von aktuellen Marktdaten abhängen. Die Testdaten bleiben auch nach der Auswahl von Twelve Data als primärem Anbieter synthetisch. Weil der Mock keine Verbindung zur Finanzdaten-API benötigt, laufen die Tests ohne Internetzugang und ohne API-Key (NF-12, NF-06) und verbrauchen kein Kontingent der kostenlosen API.

## 7. Testumgebung

| Umgebung | Was läuft dort | Wann |
|---|---|---|
| **Lokal (Entwickler)** | Backend: Unit- und Integrationstests mit `mvnw test` bzw. `mvnw.cmd test`; Frontend: Jest | Vor jedem Commit bzw. vor dem Öffnen eines Pull Requests |
| **CI-Pipeline (GitHub Actions)** | Backend: Build, Unit- und Integrationstests, Abdeckungsbericht mit JaCoCo; Frontend: Build und Jest-Tests. Ohne Internetzugriff auf die Finanzdaten-API und ohne API-Key. | Automatisch bei jedem Pull Request auf `main`; ein Merge ist nur bei erfolgreicher Pipeline möglich (CONTRIBUTING.md) |
| **Abnahmeumgebung** | Vollständige Anwendung (Frontend, Backend, PostgreSQL), Backend mit `MockMarketDataProvider` | Vor dem Sprint Review |

Für die manuell durchgeführten Alternativszenarien muss der `MockMarketDataProvider` in der Abnahmeumgebung einen Ausfall der Finanzdaten-API darstellen können. Fälle mit vorgegebenem Cache-Alter oder ausbleibender Antwort werden automatisiert getestet. [OFFEN: Wie der Schalter für den API-Ausfall umgesetzt wird (z. B. Konfigurationswert oder Testprofil) und wo die Abnahmeumgebung läuft (lokal oder auf Heroku), ist mit BE #27, BE #39 und Docs #29 abzustimmen.]

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

**Zielwert:** mindestens 80 % für die Kernlogik (NF-11), d. h. für die Klassen der Simulation (`ParameterEstimator`, `MonteCarloEngine`, `MetricsCalculator`, `SimulationService`) und der Portfolio-Prüfung (`Portfolio`, `PortfolioService`).

**Messung:** JaCoCo erzeugt bei jedem Lauf der Backend-CI-Pipeline einen Abdeckungsbericht. Die Auswertung und Interpretation erfolgt in Docs #24 und wird im Sprint Review vorgestellt. Zusätzlich wird die Zweigabdeckung (Branch Coverage) mit ausgewiesen, weil die Prüfregeln viele Verzweigungen enthalten.

**Interpretation und Grenzen:**

- Eine hohe Abdeckung zeigt nur, dass Code ausgeführt wurde, nicht dass das Ergebnis geprüft wurde. Ein Test ohne aussagekräftige Assertions erhöht die Abdeckung, findet aber keine Fehler. Deshalb wird im Code-Review darauf geachtet, dass jeder Test erwartete Werte prüft.
- Fehlende Anforderungen erkennt die Metrik nicht: Code, der nie geschrieben wurde, kann nicht unabgedeckt sein. Diese Lücke schließen die Abnahmetestfälle, die über die Spalte „Bezug“ jedem Akzeptanzkriterium zugeordnet sind.
- Der Gesamtwert wird nach Klassen aufgeschlüsselt betrachtet. Ein hoher Durchschnitt kann eine schwach getestete Kernklasse verdecken.
- Konfigurations- und reine Datenklassen werden nicht in den Zielwert einbezogen, damit die Metrik die Kernlogik widerspiegelt.

## 10. Nachweis nicht-funktionaler Anforderungen

| NFR | Anforderung | Art des Nachweises | Testfall-ID oder Prüfmethode | Sprint |
|---|---|---|---|---|
| NF-01 | Performance der Simulation: 10.000 Pfade, mit 5 Anlagen (Messfall), Horizont 30 Jahre, serverseitig ≤ 3 s in 95 % von 20 Messläufen | Performancetest (Messung von `runtimeMs`) | AT-04-14 | 1 |
| NF-02 | Performance der Suche: ≤ 1 s mit Cache-Treffer, ≤ 3 s ohne Cache | Zeitmessung im Abnahmetest | AT-02-08 | 1 |
| NF-03 | Passwörter nur gehasht gespeichert | Unit-Test und Prüfung der Datenbankinhalte | wird mit US-01 ergänzt | 2 |
| NF-04 | Zugriff auf fremde Ressourcen liefert 404 | Integrations- und Abnahmetest | wird mit US-01 ergänzt | 2 |
| NF-05 | Nur HTTPS erreichbar | Manuelle Prüfung: Aufruf per `http://` wird nicht unverschlüsselt beantwortet | Prüfung nach dem Deployment (Docs #29) | ab Deployment |
| NF-06 | API-Key nur als Umgebungsvariable im Backend | Review: kein Key in Repositories und im ausgelieferten Frontend-Code; `.env` in `.gitignore`; CI-Tests laufen ohne Key | Checkliste im Code-Review; NF-12-Prüfung | 1 |
| NF-07 | Bei API-Ausfall Antwort ≤ 5 s mit verständlicher Meldung (HTTP 503); Kursdaten jünger als 24 h werden weiterverwendet | Abnahmetest mit simuliertem Ausfall; Unit-Test des Cache-Fallbacks | AT-02-07, AT-04-06, AT-04-07 | 1 |
| NF-08 | Gleicher Seed und gleiche Eingaben ergeben identische Perzentile | Automatisierter Unit-Test (`MonteCarloEngine`) | AT-04-13 | 1 |
| NF-09 | 3 von 3 Testpersonen erstellen ohne Hilfe in < 5 min ein Portfolio und simulieren es | Bedienbarkeitstest mit Testpersonen ohne Vorkenntnisse | [OFFEN: Termin und Ablauf des Bedienbarkeitstests] | [OFFEN] |
| NF-10 | Alle Seiten ab 360 px Breite ohne horizontales Scrollen bedienbar | Manuelle Prüfung im Browser bei 360 px Breite | Sichtprüfung aller Seiten im Abnahmetest | ab 1 (je neue Seite) |
| NF-11 | Zeilenabdeckung der Kernlogik ≥ 80 % in der CI | JaCoCo-Bericht | Abschnitt 9, Docs #24 | ab 1 |
| NF-12 | Tests laufen mit `MockMarketDataProvider` ohne Internetzugang | CI-Lauf ohne API-Key und ohne Zugriff auf die Finanzdaten-API | grüner CI-Lauf | ab 1 |
| NF-13 | Start aller Komponenten mit `docker compose up` | Manueller Start in sauberer Umgebung | Prüfung vor dem Sprint Review [OFFEN: Anforderung an das Deployment bei Umstieg auf Heroku neu formulieren] | ab Deployment |
| NF-14 | Hinweis „Keine Anlageberatung“ auf jeder Ergebnisseite | Abnahmetest | AT-04-15 | 1 |
| NF-15 | Schnittstelle vollständig in `api/` beschrieben | Abgleich der Integrationstests und Abnahmetests mit dem API-Design | Review bei jeder Schnittstellenänderung | ab 1 |

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
| Product Owner – Mert Eminel (KujoBaraka) | Sichtet die Abnahmetestfälle, führt die manuellen Abnahmetests durch, setzt den Status, priorisiert gefundene Fehler, nimmt Stories ab |
| Scrum Master – Carl Stibbe (carlstibbe) | Achtet auf die Einhaltung der Definition of Done; schreibt Tests zu eigenen Umsetzungs-Tasks (z. B. BE #39, FE #2, FE #3) |
| Entwickler – Peter Orban (Pixxels02) | Unit-Tests für Portfolio, Suche und Simulationslogik (BE #32, BE #38, BE #41); CI-Pipelines; Messung und Interpretation der Testabdeckung (Docs #24) |
| Entwickler – Paul Kaufmann (Paul02112004) | Testkonzept und Abnahmetestfälle (Docs #23); Tests zu eigenen Endpunkten (BE #27, BE #33); unterstützt den Product Owner bei der Abnahme |
| Alle Entwickler | Schreiben Unit- und Integrationstests zu ihrem Code, prüfen Tests im Code-Review, beheben Fehler aus Bug-Issues |

## 13. Fortschreibung

Das Testkonzept wird zu Beginn jedes Sprints überprüft und bei Bedarf ergänzt. Für jeden Sprint entsteht eine eigene Datei `abnahmetestfaelle-sprint-<n>.md`. Absehbar sind:

- **Sprint 2:** Abnahme- und Integrationstests für die Anmeldung (US-01), einschließlich NF-03 (gehashte Passwörter) und NF-04 (fremde Ressourcen liefern 404). Für US-02 bis US-04 kommt die Vorbedingung „Nutzer ist angemeldet“ hinzu; bestehende Testfälle werden um den Fall ohne gültigen Token (`401 UNAUTHORIZED`) ergänzt. Tests für die Kennzahlen (US-05, Feld `metrics`).
- **Sprint 3:** US-07 als zweite Kern-Story mit eigenen Unit-Tests, Abnahmetestfällen und Pfadabdeckung ihres Aktivitätsdiagramms; Speichern von Simulationen (US-08).
- **Laufend:** Nachweis der übrigen NFRs (NF-05, NF-09, NF-13) laut Tabelle in Abschnitt 10; Ergänzung von Tests, sobald ein Fallback-Anbieter zu Twelve Data entschieden ist.
