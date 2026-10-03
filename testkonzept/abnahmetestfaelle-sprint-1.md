# Abnahmetestfälle – Sprint 1

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026) |
| **Issue** | Docs #23 Testkonzept v1 und Abnahmetestfälle |
| **Verantwortlich** | Paul Kaufmann (Paul02112004) |
| **Stand** | 02.10.2026 |
| **Stories** | US-02 Anlagen suchen (Docs #8), US-03 Portfolio mit Gewichtungen erstellen (Docs #1), US-04 Monte-Carlo-Simulation durchführen (Docs #7, Kern-Story) |
| **Grundlagen** | Akzeptanzkriterien und Festlegungen des Product Owners in `anforderungen/kurzformat_sprint-1.md` (Stand 30.09.2026), Entscheidungen des Product Owners E1 bis E7 (02.10.2026), API-Design (Stand 02.10.2026), Aktivitätsdiagramm US-04 v1, [Testkonzept](testkonzept.md) |

---

## Vorbemerkungen

- **Umfang:** Jedes Akzeptanzkriterium ist durch mindestens einen Testfall abgedeckt; die Spalte „Bezug“ nennt das jeweilige Akzeptanzkriterium bzw. die NFR. Die Zuordnung der NFRs zu den Testfällen steht in Abschnitt 10 des Testkonzepts. Grenzwerte desselben Eingabefelds sind in einem Testfall zusammengefasst, der alle Werte nacheinander prüft (als parametrisierter Test umgesetzt).
- **Durchführung:** Die Spalte „Durchführung“ legt fest, wie ein Testfall abgenommen wird (Testkonzept, Abschnitt 4):
  - **manuell:** Der Product Owner (KujoBaraka) führt den Testfall mit Unterstützung der Entwickler in der Abnahmeumgebung durch. Das betrifft Normalfälle und Alternativszenarien aus Nutzersicht sowie Zeitmessungen.
  - **automatisiert:** Der Testfall wird durch die genannten Tests abgedeckt, die bei jedem Pull Request in der CI-Pipeline laufen. Das betrifft Grenzwerte, reine API-Fälle und Fälle, die einen gezielt manipulierten Mock erfordern. Die Namen der Testklassen sind Vorschläge nach der Projektstruktur und werden bei der Umsetzung angepasst.
- **Testdaten:** Das Backend läuft immer mit dem `MockMarketDataProvider` und den Testdaten aus Abschnitt 6 des Testkonzepts.
- **Erwartetes Ergebnis:** zuerst, was die Oberfläche zeigt (**UI**), danach die API-Antwort (**API**). Alle Pfade beziehen sich auf `/api/v1`; das Feld `token` wird in Sprint 1 ignoriert. Fehlerantworten haben das Format `{ "error": { "code", "message", "details" } }` laut API-Design.
- **Basiseingaben US-04:** „Mein Weltportfolio“ (EUNL 70 %, IS3N 30 %), `startCapital` 10000, `monthlyContribution` 200, `horizonYears` 20, ohne `paths` und ohne `seed`. In den Schritten wird nur der abweichende Wert genannt.
- **Status:** zunächst `offen`, nach der Durchführung `bestanden` oder `fehlgeschlagen`. Ein automatisierter Testfall gilt als `bestanden`, wenn alle genannten Tests im letzten CI-Lauf auf `main` erfolgreich waren. Bei `fehlgeschlagen` wird ein Bug-Issue angelegt und verlinkt (Testkonzept, Abschnitt 11).

**Anzahl der Testfälle**

| Story | Normalfall | Alternativszenario | Grenzwert | NFR | Summe | davon manuell | davon automatisiert |
|---|---|---|---|---|---|---|---|
| US-02 | 3 | 2 | 2 | 1 | 8 | 6 | 2 |
| US-03 | 1 | 3 | 5 | 0 | 9 | 3 | 6 |
| US-04 | 3 | 5 | 4 | 3 | 15 | 8 | 7 |
| **Gesamt** | 7 | 10 | 11 | 4 | 32 | 17 | 15 |

---

## US-02 Anlagen suchen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-02-01 | AK: Suche nach Ticker liefert Trefferliste mit Name, Ticker, Börse und Währung | Normalfall | manuell | Suchbegriff `EUNL` eingeben | **UI:** Trefferliste mit „iShares Core MSCI World UCITS ETF“, EUNL, XETRA, EUR.<br>**API:** `GET /assets/search?q=EUNL` → `200 OK`; Eintrag mit `symbol`, `name`, `exchange`, `currency`. | offen |
| AT-02-02 | AK: Suche nach ISIN und nach Name | Normalfall | manuell | 1. `IE00B4L5Y983` suchen<br>2. `msci` suchen | **UI:** 1. EUNL wird gefunden. 2. Liste mit mehreren passenden Anlagen.<br>**API:** jeweils `200 OK` mit passenden Einträgen. | offen |
| AT-02-03 | Nachbedingung: ausgewählte Anlage kann in ein Portfolio übernommen werden | Normalfall | manuell | In der Trefferliste EUNL auswählen | **UI:** EUNL erscheint im Portfolio-Editor als neue Position (`assetId` 17). | offen |
| AT-02-04 | AK: Suche startet erst ab zwei Zeichen | Grenzwert | automatisiert (AssetControllerIT, Frontend-Test Suchfeld) | Suchbegriff mit 0, 1 und 2 Zeichen | **UI:** Bei 0 und 1 Zeichen keine Suche, Hinweis auf mindestens 2 Zeichen; bei 2 Zeichen startet die Suche.<br>**API:** 0 und 1 Zeichen → `400 VALIDATION_ERROR`, „Bitte mindestens 2 Zeichen eingeben.“; 2 Zeichen → `200 OK`. | offen |
| AT-02-05 | E4: `limit` 1 bis 25, Standard 10 | Grenzwert | automatisiert (AssetControllerIT) | `GET /assets/search?q=msci` mit `limit` 0, 1, 25, 26 und ohne `limit` (Testdaten mit mehr als 25 Treffern) | **API:** 0 und 26 → `400 VALIDATION_ERROR`, `details` nennt `limit`; 1 → 1 Eintrag; 25 → 25 Einträge; ohne Angabe → 10 Einträge. | offen |
| AT-02-06 | AK: Bei keinem Treffer erscheint ein Hinweis | Alternativszenario | manuell | Suchbegriff `xyzabc` eingeben | **UI:** Hinweis, dass keine passende Anlage gefunden wurde.<br>**API:** `200 OK` mit `"results": []`. | offen |
| AT-02-07 | AK: Bei API-Ausfall verständliche Fehlermeldung statt Absturz; NF-07 | Alternativszenario | manuell | Finanzdaten-API im Mock auf „nicht erreichbar“, kein Cache; `EUNL` suchen | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“; die Seite bleibt bedienbar.<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |
| AT-02-08 | NF-02: Antwortzeit ≤ 1 s mit Cache, ≤ 3 s ohne Cache | NFR | manuell (Zeitmessung) | 1. `msci` zweimal suchen, Antwortzeit der zweiten Suche messen<br>2. Cache leeren, einen neuen Begriff suchen und messen | 1. ≤ 1 s. 2. ≤ 3 s. Messung im Netzwerk-Tab der Entwicklertools oder per HTTP-Client. | offen |

---

## US-03 Portfolio mit Gewichtungen erstellen

Vorbedingung aller Testfälle: EUNL (`assetId` 17) und IS3N (`assetId` 23) wurden über die Suche gefunden; der Portfolio-Editor ist geöffnet.

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-03-01 | AK: Name und mindestens eine Anlage; Summe 100 %; Nachbedingung: für Simulation auswählbar | Normalfall | manuell | Name `Mein Weltportfolio`; EUNL 70 %, IS3N 30 %; speichern; danach Simulationsseite öffnen | **UI:** Bestätigung; Portfolio erscheint in der Liste und ist auf der Simulationsseite auswählbar.<br>**API:** `POST /portfolios` → `201 Created` mit beiden Positionen; `POST /portfolios/list` enthält das Portfolio. | offen |
| AT-03-02 | E1: Name Pflicht, höchstens 100 Zeichen | Grenzwert | automatisiert (PortfolioTest, PortfolioControllerIT) | Name leer, mit 1, 100 und 101 Zeichen | **API:** 1 und 100 Zeichen → `201 Created`; leer und 101 Zeichen → `400 VALIDATION_ERROR`, `details` nennt `name`. | offen |
| AT-03-03 | AK: Summe 100 % ± 0,01 Prozentpunkte | Grenzwert | automatisiert (PortfolioTest, Frontend-Test Gewichtungen) | EUNL 70 % und IS3N 29,98 / 29,99 / 30,01 / 30,02 % (Summe 99,98 / 99,99 / 100,01 / 100,02 %) | **API:** 99,99 und 100,01 % → `201 Created`; 99,98 und 100,02 % → `400 VALIDATION_ERROR`, „Die Gewichtungen müssen zusammen 100 % ergeben.“ | offen |
| AT-03-04 | AK: jede Gewichtung > 0 % | Grenzwert | automatisiert (PortfolioTest) | Gewichtung von IS3N 0 %, 0,01 % und −10 % (EUNL jeweils auf 100 % ergänzt) | **API:** 0,01 % → `201 Created`; 0 % und −10 % → `400 VALIDATION_ERROR`. | offen |
| AT-03-05 | E2: höchstens 2 Nachkommastellen; exakte Dezimalrechnung | Grenzwert | automatisiert (PortfolioTest, Frontend-Test Gewichtungen) | 1. 33,33 / 33,33 / 33,34 %<br>2. 33,333 / 33,333 / 33,334 %<br>3. 69,99 / 30,01 % | **API:** 1. `201 Created`. 2. `400 VALIDATION_ERROR`. 3. `201 Created`; die Summe wird als genau 100,00 % erkannt (Gleitkomma-Testfall). | offen |
| AT-03-06 | AK: mindestens eine Anlage | Grenzwert | automatisiert (PortfolioTest, PortfolioControllerIT) | 0 Anlagen (`"positions": []`) und 1 Anlage (EUNL 100 %) | **API:** 0 Anlagen → `400 VALIDATION_ERROR`, `details` nennt `positions`; 1 Anlage → `201 Created`. | offen |
| AT-03-07 | AK: Summe ungleich 100 % → Speichern nicht möglich | Alternativszenario | manuell | EUNL 70 %, IS3N 25 %; speichern | **UI:** Speichern nicht möglich; Meldung zur Gewichtungssumme, aktuelle Summe (95 %) erkennbar.<br>**API:** `400 VALIDATION_ERROR`, `details`: „Summe der Gewichtungen ist 95 %.“ | offen |
| AT-03-08 | AK: doppelte Anlagen werden abgelehnt | Alternativszenario | manuell | EUNL über die Suche ein zweites Mal hinzufügen; speichern | **UI:** Zweites Hinzufügen wird verhindert oder Speichern ist nicht möglich; Hinweis auf die doppelte Anlage.<br>**API:** `400 VALIDATION_ERROR`. | offen |
| AT-03-09 | E5: unbekannte `assetId` wird abgelehnt | Alternativszenario | automatisiert (PortfolioServiceTest, PortfolioControllerIT) | `POST /portfolios` mit `assetId` 17 (50 %) und 99999 (50 %) | **API:** `400 VALIDATION_ERROR`; `details` mit `field` „positions“ nennt die ID 99999. | offen |

---

## US-04 Monte-Carlo-Simulation durchführen (Kern-Story)

Vorbedingung aller Testfälle, wenn nicht anders angegeben: „Mein Weltportfolio“ ist gespeichert und ausgewählt; die Finanzdaten-API (Mock) ist erreichbar.

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-04-01 | AK: Pflichtfelder; Standard 10.000 Pfade; Seed wird erzeugt und angezeigt; Aktivitätsdiagramm: Hauptpfad, Zweig „ohne Seed“ | Normalfall | manuell | Basiseingaben; Simulation starten | **UI:** Fächerdiagramm über 20 Jahre; erzeugter Seed wird angezeigt.<br>**API:** `200 OK`; `parameters.paths` = 10000; `parameters.seed` ist eine ganze Zahl; `timeline` hat 241 Punkte; `id` ist `null`. | offen |
| AT-04-02 | AK: Pfadanzahl und Seed optional; Aktivitätsdiagramm: Zweig „mit Seed“ | Normalfall | manuell | Basiseingaben mit Pfadanzahl 20000 und Seed 42 | **UI:** Fächerdiagramm; angezeigter Seed ist 42.<br>**API:** `200 OK`; `parameters.paths` = 20000, `parameters.seed` = 42. | offen |
| AT-04-03 | AK: Rendite und Volatilität aus Kursdaten geschätzt; Fächerdiagramm mit 5., 50. und 95. Perzentil | Normalfall | manuell | Basiseingaben; Diagramm und API-Antwort prüfen | **UI:** Drei beschriftete Verläufe bzw. Band für 5., 50. und 95. Perzentil; Start bei 10.000 €.<br>**API:** `estimates` enthält je Anlage `annualReturn` und `annualVolatility`; in jedem Punkt von `timeline` gilt `p5` ≤ `p50` ≤ `p95`. | offen |
| AT-04-04 | Aktivitätsdiagramm: Rücksprung „Eingaben ungültig“ | Alternativszenario | manuell | Anlagehorizont 0, starten; danach auf 20 korrigieren und erneut starten | **UI:** Zuerst Fehlermeldung am Feld, kein Diagramm; nach der Korrektur Fächerdiagramm ohne erneute Portfolioauswahl.<br>**API:** erst `400 VALIDATION_ERROR`, dann `200 OK`. | offen |
| AT-04-05 | Aktivitätsdiagramm: Portfolio nicht vorhanden | Alternativszenario | automatisiert (SimulationServiceTest, SimulationControllerIT) | `POST /simulations` mit `portfolioId` 9999 | **API:** `404 NOT_FOUND`, „Portfolio nicht gefunden.“ | offen |
| AT-04-06 | AK: Kursdaten nicht abrufbar → verständliche Fehlermeldung; Aktivitätsdiagramm: „nicht verfügbar“; NF-07 | Alternativszenario | manuell | Finanzdaten-API im Mock auf „nicht erreichbar“, kein Cache; Basiseingaben | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“; Eingaben bleiben erhalten.<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |
| AT-04-07 | NF-07: Kursdaten jünger als 24 h werden bei Ausfall weiterverwendet | Alternativszenario | automatisiert (CachingMarketDataProviderTest) | Finanzdaten-API nicht erreichbar; Kursdaten im Cache 23 h bzw. 25 h alt | **API:** 23 h → `200 OK`; 25 h → `503 UPSTREAM_UNAVAILABLE`. | offen |
| AT-04-08 | AK: weniger als 3 Jahre Kurshistorie → Hinweis nennt betroffene Anlagen; E6 | Alternativszenario | manuell | Portfolio aus EUNL 50 % und zwei Testanlagen mit kurzer Historie je 25 %; Basiseingaben | **UI:** Hinweis mit den Namen beider Testanlagen (nicht EUNL); kein Diagramm.<br>**API:** `422 INSUFFICIENT_DATA`; `details` enthält genau zwei Einträge mit `assetId` und `name`. | offen |
| AT-04-09 | AK: mindestens 3 Jahre Kurshistorie | Grenzwert | automatisiert (ParameterEstimatorTest) | Testanlage mit Historie knapp unter 3 Jahren und mit genau 3 Jahren | Knapp darunter → als zu kurz erkannt (`422`); genau 3 Jahre → gültig.<br>[OFFEN: Definition von „3 Jahre Kurshistorie“, z. B. Anzahl Monatswerte.] | offen |
| AT-04-10 | AK: Anlagehorizont 1 bis 40 ganze Jahre, Pflichtfeld | Grenzwert | automatisiert (SimulationControllerIT, MonteCarloEngineTest) | `horizonYears` 0, 1, 40, 41, 2.5 und ohne Angabe | **API:** 1 → `200 OK`, 13 Punkte; 40 → `200 OK`, 481 Punkte; 0, 41, 2.5 und ohne Angabe → `400 VALIDATION_ERROR`, `details` nennt `horizonYears`. | offen |
| AT-04-11 | AK: Startkapital und Sparrate Pflicht, nicht negativ, nicht beide 0 | Grenzwert | automatisiert (SimulationControllerIT) | 1. beide 0<br>2. nur Startkapital 0<br>3. nur Sparrate 0<br>4. Startkapital −1<br>5. Sparrate fehlt | **API:** 2 und 3 → `200 OK`; 1, 4 und 5 → `400 VALIDATION_ERROR` mit dem betroffenen Feld in `details`. | offen |
| AT-04-12 | AK: Pfadanzahl 1.000 bis 50.000; Seed ganze Zahl | Grenzwert | automatisiert (SimulationControllerIT) | `paths` 999, 1000, 50000, 50001; `seed` 4.2 | **API:** 1000 und 50000 → `200 OK`; 999, 50001 und Seed 4.2 → `400 VALIDATION_ERROR`. | offen |
| AT-04-13 | AK: gleicher Seed → identische Ergebnisse; NF-08 | NFR | automatisiert (MonteCarloEngineTest) | Zwei Läufe mit identischen Eingaben und Seed 42 | Alle Werte in `timeline` sind identisch. | offen |
| AT-04-14 | NF-01: 10.000 Pfade, 5 Anlagen (Messfall), Horizont 30 Jahre, ≤ 3 s in 95 % von 20 Läufen | NFR | manuell (Messung in der Referenzumgebung) | Portfolio aus 5 Testanlagen à 20 %, Kursdaten im Cache; `horizonYears` 30, `paths` 10000, Seed 42; 20-mal ausführen und `runtimeMs` notieren | Mindestens 19 von 20 Läufen mit `runtimeMs` ≤ 3000. Messwerte werden im Sprint Review dokumentiert.<br>[OFFEN: Referenzumgebung für die Messung.] | offen |
| AT-04-15 | NF-14: Hinweis „Keine Anlageberatung“ | NFR | manuell | Basiseingaben; Ergebnisseite prüfen | **UI:** Hinweis ist auf der Ergebnisseite ohne weiteres Klicken sichtbar. | offen |

---

## Automatisierte Tests

| Testklasse (Vorschlag) | Teststufe | Abgedeckte Testfälle |
|---|---|---|
| `PortfolioTest` (`validate()`, `getWeights()`) | Unit-Test, parametrisiert | AT-03-02 bis AT-03-06 |
| `PortfolioServiceTest` | Unit-Test mit Mockito | AT-03-09 |
| `PortfolioControllerIT` | Integrationstest (`MockMvc`) | AT-03-02, AT-03-06, AT-03-09 |
| `AssetControllerIT` | Integrationstest (`MockMvc`) | AT-02-04, AT-02-05 |
| `ParameterEstimatorTest` | Unit-Test | AT-04-09 |
| `MonteCarloEngineTest` | Unit-Test | AT-04-10, AT-04-13 |
| `CachingMarketDataProviderTest` | Unit-Test | AT-04-07 |
| `SimulationServiceTest` | Unit-Test mit Mockito | AT-04-05 |
| `SimulationControllerIT` | Integrationstest (`MockMvc`) | AT-04-05, AT-04-10 bis AT-04-12 |
| Frontend-Tests (Jest) | Unit-Test | AT-02-04, AT-03-03, AT-03-05 |

Alle automatisierten Tests laufen mit dem `MockMarketDataProvider` ohne Internetzugang (NF-12). Die Performancemessung AT-04-14 läuft nicht in der CI, weil die Laufzeit auf geteilten CI-Runnern schwankt.
