# Abnahmetestfälle – Sprint 4

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 4 (19.10.2026 – 25.10.2026) |
| **Issue** | Docs #135 Abnahmetestfälle Sprint 4 und Testkonzept v4 |
| **Verantwortlich** | Peter Orban (Pixxels02); Freigabe durch Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 25.10.2026 |
| **Durchführung** | Nachtest (Docs #136) und Abnahme (Docs #137) durch den Product Owner; Ergebnis im Abnahmeprotokoll Sprint 4 |
| **Stories** | Nachtest US-05 (Docs #6) und US-07 (Docs #3, Kern-Story) nach den Befunden B1 bis B7 und H1; B6 Anlagensuche „ETHA“ (BE #88, BE #97, FE #40); B5 laufender Monat (BE #87); ADR-06 Token im Header (Docs #115, BE #91, FE #37); US-09 Beispielsimulation für Gäste (Docs #2, Stretch); NF-12 (BE #89) |
| **Grundlagen** | Sprint-Planning-Protokoll Sprint 4 (Abschnitt 4 „Erledigt, wenn“), Abnahmeprotokoll Sprint 3, `anforderungen/kurzformat_sprint-3.md` (E24 bis E35), ADR-06, API-Design (Stand 25.10.2026, Docs #133), Aktivitäts- und Sequenzdiagramm US-07 v2 (Docs #130), [Testkonzept](testkonzept.md) Version 4 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und geprüft, siehe KI-Log |

Diese Datei setzt [abnahmetestfaelle-sprint-3.md](abnahmetestfaelle-sprint-3.md) fort. Sie enthält die Vorgaben für den Nachtest der in Sprint 3 nicht bestandenen oder offenen Testfälle sowie die neuen Testfälle für B5, B6, ADR-06, US-09 und NF-12. Testfälle zum Change Request (Ereigniskarte) werden nach der Auswirkungsanalyse (Docs #131) ergänzt.

---

## Vorbemerkungen

Es gelten die Vorbemerkungen aus Sprint 1 bis 3 (Bezug, Durchführung, Darstellung von UI und API, Status, Meldungstexte, Testdaten auf US-Börsen, Basiseingaben, Seed `42`, Zielbetrag `100000`). Zusätzlich gilt:

- **Anmeldung (ADR-06):** Geschützte Anfragen senden das Token im Header `Authorization: Bearer <token>`, nicht mehr im Body. Lesende Abfragen laufen per GET (`GET /portfolios`, `GET /portfolios/{id}`, `GET /simulations`, `GET /simulations/{id}`). Bei API-Schritten aus älteren Testfällen werden `POST /…/list` und `POST /…/{id}/details` durch diese Pfade ersetzt.
- **Abnahmeumgebung:** Frontend und Backend auf Heroku mit der echten Twelve-Data-API. Der `MockMarketDataProvider` (BE #89) liegt nur im Testcode und wird ausschließlich in automatisierten Tests verwendet.
- **ETHA:** Seit B6 (BE #97) wird ETHA auch über das Symbol gefunden. „Krypto-Test“ bleibt bis Ende 2026 als Portfolio mit einer zu kurzen Anlage gültig; Twelve Data liefert für IBIT 43 Monatswerte, nur ETHA ist zu kurz (Abnahmeprotokoll Sprint 3).
- **Handyansicht (NF-10):** Jede neue oder geänderte Seite wird zusätzlich bei 360 px Breite geprüft (AT-NF10-01).

**Anzahl der neuen Testfälle**

| Bereich | Normalfall | Alternativszenario | Grenzwert | NFR | Summe | davon manuell | davon automatisiert |
|---|---|---|---|---|---|---|---|
| US-02 (B6) | 1 | 1 | 0 | 0 | 2 | 2 | 0 |
| US-04 (B5) | 0 | 0 | 1 | 0 | 1 | 0 | 1 |
| ADR-06 | 1 | 3 | 0 | 1 | 5 | 5 | 0 |
| US-09 | 1 | 1 | 1 | 1 | 4 | 2 | 2 |
| NF-12 | 0 | 0 | 0 | 1 | 1 | 0 | 1 |
| **Gesamt** | 3 | 5 | 2 | 3 | 13 | 9 | 4 |

Bei Testfällen mit „manuell und automatisiert“ zählt die manuelle Durchführung; der automatisierte Teil ergänzt sie.

---

## Nachtest der Befunde aus Sprint 3

Nach dem Merge aller Frontend-Befunde werden die in Sprint 3 fehlgeschlagenen und offenen Testfälle erneut durchgeführt (Testkonzept, Abschnitt 11). Die Testfälle bleiben in [abnahmetestfaelle-sprint-3.md](abnahmetestfaelle-sprint-3.md) definiert; es gilt die Spalte „Beim Nachtest“.

| Testfall | Befund | Beim Nachtest | Grund | Ergebnis |
|---|---|---|---|---|
| AT-05-02 | B1 (FE #30) | unverändert | Eingabefeld Zielbetrag ergänzt | offen |
| AT-07-01 | B2 (FE #31) | unverändert | Seed und gemeinsamer Schätzzeitraum werden angezeigt | offen |
| AT-07-02 | B1, B3 (FE #30, FE #32) | unverändert | Eingabe von Bezeichnung und Zielbetrag ergänzt | offen |
| AT-07-03 | – | unverändert (manueller Teil) | in Sprint 3 nicht durchgeführt | offen |
| AT-07-04 | – | unverändert | in Sprint 3 nicht durchgeführt | offen |
| AT-07-08 | B7 (FE #34) | **Erwartung angepasst:** Die Meldung nennt **ETHA genau einmal**; IBIT wird nicht genannt (43 Monatswerte) | Testdaten-Annahme aus Sprint 3 korrigiert; B7 | offen |
| AT-07-09 | – | unverändert | in Sprint 3 nicht durchgeführt | offen |
| AT-07-13 | – | unverändert | in Sprint 3 nicht gemessen | offen |
| AT-04-08 | B7 (FE #34) | „Krypto-Test“ simulieren; ETHA wird **genau einmal** genannt, `message` und `details` stehen nicht doppelt hintereinander | B7 | offen |
| Sichtprüfung B4 | B4 (FE #33) | Die Vergleichsseite zeigt nach dem Ergebnis die Hinweise E16/E21 wie die Simulationsseite (NF-14 a, wie AT-04-15) | kein eigener Testfall laut Sprint-Planning | offen |
| Sichtprüfung H1 | H1 (FE #35) | In AT-07-02 haben beide Fächerdiagramme dieselbe Y-Achse (Maximum über beide Einträge) | Hinweis aus der Abnahme Sprint 3 | offen |
| AT-NF09-01 | – | unverändert | Bedienbarkeitstest aus Sprint 3 nachgeholt (Docs #134) | offen |
| AT-NF10-01 | – | zusätzlich die Startseite mit Beispielsimulation (US-09) | NF-10 aus Sprint 3 nachgeholt (FE #36) | offen |

---

## US-02 Anlagensuche (B6)

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-02-10 | B6 (BE #88, BE #97): Suche nach „ETHA“ ohne Fehler; E10 | Normalfall | manuell | Als Nutzer A im Portfolio-Editor „ETHA“ suchen | **UI:** Unter den Treffern steht ETHA (iShares Ethereum Trust, NASDAQ); es erscheint keine Fehlermeldung.<br>**API:** `GET /api/v1/assets/search?q=ETHA` → `200 OK`; `results` enthält einen Eintrag mit `symbol` „ETHA“ und Börse NASDAQ. | offen |
| AT-02-11 | FE #40: Fehlermeldung der Anlagensuche erscheint unter dem Suchfeld; E3 | Alternativszenario | manuell | Im Suchfeld nur „E“ eingeben und suchen | **UI:** „Bitte mindestens 2 Zeichen eingeben.“ steht direkt unter dem Suchfeld; das Feld bleibt bedienbar, nach Eingabe von „ETHA“ verschwindet die Meldung.<br>**API:** Die Oberfläche sendet keine Anfrage; direkt per HTTP-Client `q=E` → `400 VALIDATION_ERROR`. | offen |

---

## US-04 Simulation (B5)

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-04-17 | B5 (BE #87): Nur abgeschlossene Monate zählen; E22 | Grenzwert | automatisiert (`PriceHistoryServiceTest`) | Feste Uhr; Kursreihe mit Wert im laufenden Monat; Cache vor und nach dem Monatswechsel; Wert am letzten Tag des Monats | Der Wert des laufenden Monats wird gespeichert, aber weder zur Schätzung noch zu den 36 Monatswerten gezählt; nach dem Monatswechsel zählt der gerade abgeschlossene Monat; der letzte Tag gehört noch zum laufenden Monat (`laufenderMonatWirdBeimAbrufAusgeschlossenAberGespeichert`, `laufenderMonatAusDemCacheWirdAusgeschlossen`, `nachDemMonatswechselZaehltDerAbgeschlosseneMonat`, `letzterTagDesMonatsGehoertNochZumLaufendenMonat`). | offen |

---

## ADR-06 Token im Header, lesende Abfragen per GET

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-ADR6-01 | ADR-06: Header statt Body, lesende Abfragen per GET; FE #37 | Normalfall | manuell | Als Nutzer A anmelden; Portfolioliste, ein Portfolio und die gespeicherten Simulationen öffnen; „Mein Weltportfolio“ simulieren; Netzwerk-Tab der Entwicklertools beobachten | **UI:** Alle Seiten funktionieren wie in Sprint 3.<br>**API:** Die Liste kommt über `GET /api/v1/portfolios`, das Portfolio über `GET /api/v1/portfolios/{id}`, die Simulationen über `GET /api/v1/simulations`, jeweils `200 OK` mit Header `Authorization: Bearer …`; **kein** Request-Body enthält ein Feld `token`. | offen |
| AT-ADR6-02 | ADR-06: fehlender oder ungültiger Header → 401; NF-16 | Alternativszenario | manuell (API) und automatisiert (`CurrentUserArgumentResolverTest`) | `GET /api/v1/portfolios` 1. ohne Header, 2. mit `Authorization: Basic abc`, 3. mit `Authorization: Bearer ` (leer), 4. mit unbekanntem Token | Jeweils `401 UNAUTHORIZED`, „Bitte melde dich an.“. Automatisiert zusätzlich: `bearer` in Kleinschreibung ist gültig. | offen |
| AT-ADR6-03 | ADR-06 und NF-16: Abmelden sperrt das Token sofort | Alternativszenario | manuell (API) | Als Nutzer A anmelden; `POST /auth/logout` **ohne Body** mit Header; danach mit demselben Token `GET /api/v1/portfolios` | Logout → `204 No Content`; danach `401 UNAUTHORIZED`, „Bitte melde dich an.“. | offen |
| AT-ADR6-04 | NF-04 mit GET-Pfaden | NFR | manuell (API) und automatisiert (`PortfolioControllerTest`, `SimulationControllerTest`) | Mit dem Token von Nutzer B: `GET /api/v1/portfolios/{id}` von „Mein Weltportfolio“ (A) und `GET /api/v1/simulations/{id}` von „Sparplan 20 Jahre“ (A) | `404 NOT_FOUND`, „Portfolio nicht gefunden.“ bzw. „Simulation nicht gefunden.“; `GET /api/v1/portfolios` als B liefert nur eigene Einträge. | offen |
| AT-ADR6-05 | ADR-06: alte POST-Pfade entfallen, keine Übergangsphase | Alternativszenario | manuell (API) | Mit gültigem Token: `POST /api/v1/portfolios/list` | `405 METHOD_NOT_ALLOWED`, „Diese HTTP-Methode ist hier nicht erlaubt.“; es werden keine Daten geliefert. | offen |

---

## US-09 Beispielsimulation für Gäste (Stretch)

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-09-01 | AK US-09: ohne Anmeldung Beispielportfolio simulieren; Ergebnis mit Fächerdiagramm, Kennzahlen und Hinweisen E16/E21 (NF-14 a); Einladung zur Registrierung | Normalfall | manuell | Ohne Anmeldung (privates Fenster) die Startseite öffnen; ein Beispielportfolio wählen; Beispielsimulation starten | **UI:** Fächerdiagramm, Kennzahlen aus US-05 und die Hinweise E16/E21 erscheinen; ein Hinweis lädt zur Registrierung ein; es gibt keine Speichern-Funktion.<br>**API:** `GET /api/v1/demo/portfolios` → `200 OK` mit den Beispielportfolios; `POST /api/v1/demo/simulations` ohne `Authorization`-Header → `200 OK`, Antwort ohne `id`. | offen |
| AT-09-02 | NF-08: gleicher Seed → identisches Ergebnis | NFR | manuell (API) und automatisiert (`DemoServiceTest`) | `POST /api/v1/demo/simulations` zweimal mit identischen Parametern und `seed` 42; automatisiert zusätzlich ohne Seed | Beide Läufe liefern identische `timeline`, `estimates` und `metrics`; ohne Seed wird einer erzeugt und in `parameters.seed` zurückgegeben (`ohneSeedWirdEinerErzeugt`). | offen |
| AT-09-03 | Grenze für Gäste: höchstens 10.000 Pfade; Parameter wie US-04 | Grenzwert | automatisiert (`DemoServiceTest`, `DemoControllerTest`) | `paths` 10.000 und 10.001; `seed` 4.2 | 10.000 → gültig (`zehntausendPfadeSindErlaubt`); 10.001 → `400 VALIDATION_ERROR`, „In der Beispielsimulation sind höchstens 10.000 Simulationspfade möglich.“, `details` nennt `paths`; Kommazahl beim Seed → `400` (`kommazahlBeimSeedLiefert400`). | offen |
| AT-09-04 | Fehlerfälle der Beispielsimulation | Alternativszenario | automatisiert (`DemoServiceTest`, `DemoControllerTest`) | Unbekannte `portfolioId`; ohne `portfolioId`; ohne `parameters`; Kursdaten nicht abrufbar | Unbekannt → `404`, „Beispielportfolio nicht gefunden.“; ohne `portfolioId` → `400`, „Bitte ein Portfolio auswählen.“; ohne `parameters` → `400`; Anbieter nicht erreichbar → `503 UPSTREAM_UNAVAILABLE` (`kursdatenNichtVerfuegbarLiefert503`). | offen |

**Hinweis zu AT-09-01:** Die Beispielportfolios verwenden derzeit EUNL und IS3N an der Xetra. Twelve Data liefert dafür im kostenlosen Tarif keine Kurse (F3, Docs #80); auf Heroku ist deshalb `422` statt `200` zu erwarten. Die Umstellung auf US-Börsen wird im Backend nachgeholt (Issue siehe Abnahmeprotokoll).

---

## Change Request (Ereigniskarte)

Die Akzeptanzkriterien legt der Product Owner nach der Auswirkungsanalyse fest (Docs #131, `anforderungen/change-request.md`). Die Testfälle werden danach in diesem Abschnitt ergänzt (Umsetzung BE #92).

---

## Nicht-funktionale Anforderungen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-NF12-01 | NF-12: Tests laufen mit einer Attrappe der Datenquelle ohne Internet und ohne API-Key | NFR | automatisiert (`MockMarketDataProviderTest`, CI) | CI-Lauf auf `main` mit Dummy-Key (`ci-dummy-key`); `MockMarketDataProvider` mit den Testdaten aus Testkonzept Abschnitt 6, einschließlich Ausfall-Schalter | CI grün; `MockMarketDataProviderTest` bestanden (120, 35 und 36 Monatswerte, mehr als 25 Suchtreffer für „MSCI“, unbekanntes Symbol leer, Ausfall → `UpstreamUnavailableException`, Zusammenspiel mit `PriceHistoryService`). | offen |

AT-NF09-01 und AT-NF10-01 werden im Nachtest durchgeführt (siehe oben).

---

## Automatisierte Tests

| Testklasse | Repository | Teststufe | Abgedeckte Testfälle | Stand 25.10.2026 |
|---|---|---|---|---|
| `MockMarketDataProviderTest` | BE | Unit-Test | AT-NF12-01 | neu (BE #89) |
| `PriceHistoryServiceTest` | BE | Unit-Test mit vorgegebener Uhr | AT-04-17 | um den laufenden Monat ergänzt (BE #87) |
| `CurrentUserArgumentResolverTest` | BE | Integrationstest (`MockMvc`) | AT-ADR6-02 | neu (BE #91) |
| `PortfolioControllerTest`, `SimulationControllerTest` | BE | Integrationstest (`MockMvc`) | AT-ADR6-04; AT-06-06, AT-08-06 | auf Header und GET-Pfade umgestellt (BE #91); `PUT` ohne Token ergänzt (BE #90) |
| `ComparisonControllerTest` | BE | Integrationstest (`MockMvc`) | AT-07-11 | 503-Test für `POST /comparisons` ergänzt (BE #90) |
| `DemoServiceTest` | BE | Unit-Test mit Mockito | AT-09-02 bis AT-09-04 | neu (BE #93) |
| `DemoControllerTest` | BE | Integrationstest (`MockMvc`) | AT-09-03, AT-09-04 | neu (BE #93) |