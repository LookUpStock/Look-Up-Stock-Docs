# Abnahmeprotokoll – Sprint 4

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 4 (19.10.2026 – 25.10.2026) |
| **Issue** | Docs #137 Abnahmetests Sprint 4 durchführen; Docs #136 Nachtest US-05 und US-07 nach den Bugfixes |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Datum der Abnahme** | 25.10.2026, 21:11 bis 22:19 Uhr |
| **Stories** | Nachtest: US-05 Kennzahlen anzeigen (Docs #6), US-07 Anlagen und Portfolios vergleichen (Docs #3, Kern-Story) nach den Befunden B1 bis B4, B7 und H1. Neu: US-09 Beispielsimulation für Gäste (Docs #2, Stretch), ADR-06 Token im Header (Docs #115), B5 laufender Monat (BE #87), B6 Anlagensuche „ETHA“ (BE #88, BE #97, FE #40), NF-12 (BE #89). Nachgeholt: NF-10 (FE #36), NF-09 (Docs #134) |
| **Grundlagen** | [Abnahmetestfälle Sprint 3](abnahmetestfaelle-sprint-3.md) und [Sprint 4](abnahmetestfaelle-sprint-4.md), [Abnahmeprotokoll Sprint 3](abnahmeprotokoll-sprint-3.md), [Testkonzept](testkonzept.md) Version 4, `anforderungen/kurzformat_sprint-4.md` Version 2 (US-09, E44 bis E47), ADR-06, API-Design (Stand 25.10.2026) |
| **KI-Nutzung** | Durchführung mit KI-Unterstützung vorbereitet und ausgewertet, Tests vom Product Owner selbst durchgeführt, Entscheidungen vom Product Owner getroffen, siehe KI-Log |

Die Status der einzelnen Testfälle stehen in der Spalte „Status“ von `abnahmetestfaelle-sprint-4.md`, die Ergebnisse des Nachtests in derselben Datei in der Spalte „Ergebnis“ des Abschnitts „Nachtest der Befunde aus Sprint 3“. Dieses Protokoll fasst das Ergebnis je Story zusammen, begründet die Abnahmeentscheidung und listet die gefundenen Hinweise.

---

## 1. Testumgebung und Durchführung

| | |
|---|---|
| **Frontend** | `https://lookupstockfrontend-96a9998805cb.herokuapp.com`, Stand `main` 358071f (25.10.2026, 20:38, Beispielsimulation US-09); Vergleich mit B2, B3, B4 und H1 seit PR #50 (20:19) |
| **Backend** | `https://lookupstock-da2951e42089.herokuapp.com`, Stand `main` dd88784 (25.10.2026, 20:10, Merge PR #104 zu BE #102) |
| **Datenbank** | Heroku Postgres |
| **Finanzdaten** | echte Twelve-Data-API; der `MockMarketDataProvider` (BE #89) wird nur in automatisierten Tests verwendet |
| **Testnutzer** | `abnahme3.a@example.com` (Nutzer A) und `abnahme3.b@example.com` (Nutzer B) aus Sprint 3 |
| **Testdaten** | Portfolios von Nutzer A aus Sprint 3: „Mein Weltportfolio“ (id 34, URTH 70 %, IEMG 30 %), „Mein Aktienportfolio“ (AAPL 50 %, MSFT 50 %), „Krypto-Test“ (IBIT 50 %, ETHA 50 %), „Messportfolio“ (5 Anlagen je 20 %); gespeicherte Simulation „Sparplan 20 Jahre“ (id 3). Nutzer B hat keine Daten. Beispielportfolios 900 und 901 im Code (`DemoPortfolio`) |
| **Manuelle Tests** | Product Owner im Browser (Brave, Chromium 156); API-Prüfungen und Messungen mit `fetch` in der Konsole der Entwicklertools; eine Gegenprobe mit `curl` (AT-ADR6-05); Handyansicht mit der Geräteansicht der Entwicklertools bei 360 px |
| **Automatisierte Tests** | Backend: „Backend CI“ Lauf #67 auf `main` dd88784 erfolgreich (25.10.2026, 20:10); Zeilenabdeckung der Kernlogik laut CI 333 von 344 Zeilen = **96,8 %** (NF-11, Zielwert 80 %). Frontend: „Frontend CI“ auf `main` 358071f erfolgreich (25.10.2026, 20:38) |

**Bewertungsregel.** Wie in Sprint 3 (Testkonzept, Abschnitt 8): Ein manueller Testfall ist `bestanden`, wenn Oberfläche und API das erwartete Ergebnis zeigen; ein automatisierter, wenn die zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren. `offen` heißt, der Testfall wurde nicht oder nur teilweise durchgeführt.

**Vorbereitung der Testdaten.** „Mein Aktienportfolio“ stand nach den Tests aus Sprint 3 noch als „Mein Aktienportfolio 2“ (AAPL 60 %, NVDA 40 %) in der Liste. Vor AT-07-02 wurde es wieder auf „Mein Aktienportfolio“ mit AAPL 50 % und MSFT 50 % gesetzt.

---

## 2. Ergebnis je Story

| Story | Testfälle | bestanden | fehlgeschlagen | offen | Entscheidung |
|---|---|---|---|---|---|
| US-05 Kennzahlen anzeigen (Nachtest) | 6 | 6 | 0 | 0 | **abgenommen** |
| US-07 Anlagen und Portfolios vergleichen (Nachtest, Kern-Story) | 13 | 13 | 0 | 0 | **abgenommen** |
| US-09 Beispielsimulation für Gäste (Stretch) | 4 | 4 | 0 | 0 | **abgenommen** |
| ADR-06 Token im Header, lesende Abfragen per GET | 5 | 5 | 0 | 0 | umgesetzt |
| US-02 Anlagensuche (B6) | 2 | 2 | 0 | 0 | behoben |
| US-04 Simulation (B5, B7) | 2 | 2 | 0 | 0 | behoben |
| NF-09, NF-10, NF-12 | 3 | 2 | 0 | 1 | – |
| **Gesamt** | 35 | 34 | 0 | 1 | |

US-05 und US-07 umfassen alle Testfälle aus Sprint 3; die Zahlen geben den Stand nach dem Nachtest wieder. US-04 umfasst den Nachtest von AT-04-08 (B7) und den neuen Testfall AT-04-17 (B5). Die Sichtprüfungen zu B4 und H1 sind keine eigenen Testfälle (Sprint-Planning Sprint 4) und stehen in Abschnitt 3 bei US-07.

---

## 3. Begründung

### US-05 Kennzahlen anzeigen – abgenommen

B1 ist behoben: Die Simulationsseite hat das Feld „Zielbetrag in € (optional)“. „Mein Weltportfolio“ mit 10.000 €, 200 €, 20 Jahren, 10.000 Pfaden, Seed 42 und Zielbetrag 100.000 € zeigt die Zielwahrscheinlichkeit 98,37 %; die übrigen Kennzahlen sind identisch mit dem Lauf ohne Zielbetrag (AT-05-02). Die API liefert `parameters.targetAmount` 100000 und `metrics.targetProbability` 0,9837. Ein Zielbetrag von 0 wird am Feld mit „Der Zielbetrag muss größer als 0 sein.“ abgelehnt.

**Hinweis zu den Werten:** In Sprint 3 ergab derselbe Lauf 97,9 % und einen Median von 218.380 €, jetzt 98,37 % und 226.632 €. Das ist die erwartete Folge von B5: Der laufende Monat zählt nicht mehr, der Schätzzeitraum ist jetzt 01.11.2016 bis 01.09.2026.

### US-07 Anlagen und Portfolios vergleichen – abgenommen

Alle Testfälle sind bestanden.

- **B2 behoben:** Seed und gemeinsamer Schätzzeitraum werden angezeigt. „Mein Weltportfolio“ gegen URTH (NYSE) ohne Seed: „Gemeinsamer Schätzzeitraum: 01.11.2016 bis 01.09.2026 · Seed: 978871“, „„Mein Weltportfolio“ schneidet in 42,6 % der gemeinsamen Simulationspfade besser ab.“ (AT-07-01).
- **B1 und B3 behoben:** Mit den Bezeichnungen „Welt“ und „Aktien“, Seed 42 und Zielbetrag 100.000 € heißen die Einträge „Welt“ und „Aktien“, Seed 42 wird angezeigt, und je Eintrag steht die Zielwahrscheinlichkeit in der Kennzahlentabelle (98,37 % und 100 %; AT-07-02).
- **NF-08:** Ein zweiter Lauf von AT-07-02 liefert dieselben Werte; zweimal „Mein Weltportfolio“ mit Seed 42 ergibt identische Kennzahlen und 0 % (AT-07-03).
- **Fehlerfälle:** Anlagehorizont 0 wird am Feld gemeldet, nach der Korrektur erscheint das Ergebnis (AT-07-04). „Krypto-Test“ gegen URTH liefert `422` mit „Für folgende Anlagen liegen weniger als 3 Jahre Kursdaten vor: iShares Ethereum Trust ETF.“, ETHA genau einmal (AT-07-08, B7). EUNL (XETR) gegen URTH liefert `422` mit „Für folgende Anlagen liefert der Kursdatenanbieter keine Kurse: iShares Core MSCI World UCITS ETF USD (Acc). Bitte die Anlage an einer anderen Börse auswählen.“ (AT-07-09).
- **E33:** 20 Vergleiche „Messportfolio“ gegen „Messportfolio“ mit 40 Jahren und 10.000 Pfaden dauerten 2.957 bis 3.045 ms; 20 von 20 liegen unter 6.000 ms (AT-07-13).
- **Sichtprüfung B4:** Die Vergleichsseite zeigt nach dem Ergebnis die Hinweise E16 und E21.
- **Sichtprüfung H1:** Beide Fächerdiagramme haben dieselbe Y-Achse (in AT-07-02 bis 10.000.000 €).

**Hinweis zu AT-07-04:** Das Frontend prüft den Anlagehorizont schon vor dem Absenden und schickt keine Anfrage. Der API-Teil der Erwartung (`400 VALIDATION_ERROR`) ist deshalb über die Oberfläche nicht erreichbar; der Vergleich nutzt dieselben `SimulationParameters` wie US-04, deren Prüfung des Anlagehorizonts automatisiert belegt ist (`SimulationParametersTest`, AT-04-10).

**Beobachtung zu H1:** Mit der gemeinsamen Y-Achse wirkt das Diagramm des kleineren Eintrags fast flach, wenn die Endwerte weit auseinanderliegen (in AT-07-02 Median 226.632 € gegen 2.959.714 €). Das ist die gewünschte Folge von H1 und kein Fehler.

### US-09 Beispielsimulation für Gäste – abgenommen

Ohne Anmeldung zeigt die Startseite den Bereich „Beispielsimulation ausprobieren“ mit den Beispielportfolios „Beispiel: Weltportfolio“ und „Beispiel: Welt 70 / Schwellenländer 30“ und den festen Werten 5.000 €, 100 € pro Monat, 15 Jahre und 10.000 Pfade (E44 bis E47). Nach dem Start erscheinen Fächerdiagramm, Kennzahlen aus US-05, die Hinweise E16 und E21 und die Einladung „Jetzt kostenlos registrieren“; eine Speichern-Funktion gibt es nicht (AT-09-01). `GET /demo/portfolios` und `POST /demo/simulations` antworten mit `200 OK` ohne `Authorization`-Header; `id`, `name` und `createdAt` sind `null`, wie im API-Design festgelegt. Zwei Läufe mit Seed 42 liefern identische `timeline`, `estimates` und `metrics` (AT-09-02). Grenze und Fehlerfälle sind automatisiert belegt (AT-09-03, AT-09-04).

### ADR-06 Token im Header – umgesetzt

Das Frontend sendet das Token als `Authorization: Bearer …` und liest per GET (`GET /portfolios`, `GET /simulations`); kein Request-Body enthält `token` (AT-ADR6-01). Ohne Header, mit `Basic abc`, mit leerem `Bearer` und mit unbekanntem Token antwortet `GET /portfolios` jeweils mit `401 UNAUTHORIZED`, „Bitte melde dich an.“ (AT-ADR6-02). Nach `POST /auth/logout` ohne Body (`204`) liefert dasselbe Token `401` (AT-ADR6-03). Nutzer B erhält für Portfolio 34 und Simulation 3 von A jeweils `404` und sieht keine eigenen Einträge (AT-ADR6-04). `POST /portfolios/list` liefert `405 METHOD_NOT_ALLOWED`, „Diese HTTP-Methode ist hier nicht erlaubt.“, ohne Daten (AT-ADR6-05).

**Hinweis zu AT-ADR6-01:** Das Frontend lädt ein einzelnes Portfolio nicht über `GET /portfolios/{id}`, sondern übernimmt es aus der Liste in den Editor. Dieser Teil der Erwartung ist über die Oberfläche nicht beobachtbar. Dass der Pfad per GET erreichbar ist und fremde Portfolios ablehnt, zeigt AT-ADR6-04; einen Test für den Abruf eines eigenen Portfolios mit `200` gibt es weder manuell noch automatisiert.

### US-02 Anlagensuche (B6) – behoben

Die Suche nach „ETHA“ liefert `200 OK`; unter den Treffern steht „iShares Ethereum Trust ETF, ETHA · NASDAQ · USD“ (AT-02-10). Bei Eingabe von nur „E“ steht „Bitte mindestens 2 Zeichen eingeben.“ unter dem Suchfeld, und es wird keine Anfrage gesendet (AT-02-11). Die Antwort `400` für einen Suchbegriff mit einem Zeichen ist seit Sprint 1 automatisiert belegt (AT-02-04).

### US-04 Simulation (B5, B7) – behoben

„Krypto-Test“ liefert `422` mit „Für folgende Anlagen liegen weniger als 3 Jahre Kursdaten vor: iShares Ethereum Trust ETF.“; ETHA wird genau einmal genannt (AT-04-08, B7). Dass nur abgeschlossene Monate zählen, ist automatisiert belegt (AT-04-17, `PriceHistoryServiceTest`); in der Oberfläche endet der Schätzzeitraum am 01.09.2026.

### NF-09, NF-10 und NF-12

- **NF-10 (AT-NF10-01):** Bei 360 px Breite sind die Startseite mit Beispielsimulation (als Gast), die Simulationsseite mit Kennzahlen, Speichern und gespeicherten Simulationen, der Portfolio-Editor mit Liste und die Vergleichsseite mit Ergebnis ohne horizontales Scrollen bedienbar (`scrollWidth <= clientWidth` jeweils `true`). Bestanden, mit den Hinweisen H5 und H6.
- **NF-12 (AT-NF12-01):** CI grün mit `ci-dummy-key`; `MockMarketDataProviderTest` ist Teil des Laufs. Bestanden.
- **NF-09 (AT-NF09-01):** Der Bedienbarkeitstest mit drei fachfremden Personen ist für den 26.10.2026 geplant (Carl Stibbe, Docs #134). Offen; das Ergebnis wird in diesem Protokoll nachgetragen.

---

## 4. Gefundene Fehler und Hinweise

Es wurde kein Fehler gefunden, der eine Abnahme verhindert. Die folgenden Hinweise betreffen die Bedienung. Priorität nach Testkonzept, Abschnitt 11.

| ID | Repo | Beschreibung | Testfälle | Priorität | Art |
|---|---|---|---|---|---|
| H2 | FE | Nach einer Fehlermeldung bleibt das Ergebnis des vorherigen Laufs stehen, auf der Simulations- und der Vergleichsseite. Es wirkt dadurch, als gehöre es zur fehlerhaften Eingabe. | AT-04-08, AT-07-04, AT-07-08, AT-07-09 | – | Hinweis |
| H3 | FE | Auf der Vergleichsseite steht die Fehlermeldung in normaler schwarzer Schrift, auf der Simulationsseite rot. | AT-07-08, AT-07-09 | – | Hinweis |
| H4 | FE | Der Portfolio-Editor zeigt direkt nach dem Laden rote Pflichtfeld-Meldungen („Der Portfolio-Name ist erforderlich.“, „Das Symbol ist erforderlich.“ usw.), bevor etwas eingegeben wurde. Festlegung des Product Owners: in Sprint 5 klären. | AT-02-10, AT-02-11 | – | Hinweis |
| H5 | FE | Bei 360 px brechen die Knöpfe „Bearbeiten“ und „Löschen“ in der Portfolioliste mitten im Wort um („Bea-rbe-ite-n“). Ursache: Die Regel `.saved-portfolio div` (`flex-direction: column`, `overflow-wrap: anywhere`) trifft auch den Container der Knöpfe; bei den gespeicherten Simulationen ist das bereits korrigiert. Festlegung des Product Owners: Behebung in Sprint 5. | AT-NF10-01 | – | Hinweis |
| H6 | FE | Bei 360 px ist die Achsenbeschriftung der Fächerdiagramme (Startseite, Vergleich) sehr klein, weil das Diagramm nur verkleinert wird. | AT-NF10-01 | – | Hinweis |
| H7 | BE | Die Antwort `405` auf `POST /portfolios/list` enthält keinen CORS-Header; der Browser blockiert deshalb den Body, die Meldung war nur mit `curl` sichtbar. Für das Frontend ohne Folgen, weil es den Pfad nicht aufruft. | AT-ADR6-05 | – | Hinweis |

**Hinweise zum Vorgehen (für Review und Retrospektive):**

- Die Beispielsimulation im Frontend (FE #38, 358071f) wurde direkt auf `main` gepusht, ohne Pull Request und Review (CONTRIBUTING.md).
- Im Docs-Repository wurden Diagramm-Anpassungen direkt auf `main` gepusht, einer der Commits heißt nur „...“ (f07b569).
- Der Merge von FE #46 und #47 hat die Änderungen aus B2, B3, B4 und H1 überschrieben. Sie wurden mit FE #49 (PR #50) vor der Abnahme wiederhergestellt.
- Docs #134 (NF-09) und Docs #138 (Testabdeckung) wurden geschlossen, bevor ein Ergebnis vorlag; in `testabdeckung-sprint-4.md` fehlen noch die Werte je Klasse.

---

## 5. Nachweise der manuellen Tests

| Prüfung | Testfälle | Beobachtung |
|---|---|---|
| Zielbetrag | AT-05-02 | „Mein Weltportfolio“, 10.000 €, 200 €, 20 Jahre, 10.000 Pfade, Seed 42: Eingezahlt 58.000 €, p5 117.500 €, Median 226.632 €, p95 447.570 €, VaR 0 €, Drawdown 16,1 %; mit Zielbetrag 100.000 € zusätzlich 98,37 %, API `targetProbability` 0,9837; ohne Zielbetrag keine Zeile; Zielbetrag 0 → Meldung am Feld |
| Vergleich | AT-07-01, AT-07-02, AT-07-03 | Welt gegen URTH ohne Seed: Seed 978871, Schätzzeitraum 01.11.2016 bis 01.09.2026, 42,6 %; „Welt“ gegen „Aktien“ mit Seed 42: Median 226.632 € und 2.959.714 €, Zielwahrscheinlichkeit 98,37 % und 100 %, „Welt“ in 0 % besser, zweiter Lauf identisch; zweimal „Mein Weltportfolio“: identische Kennzahlen, 0 % |
| Fehlerfälle | AT-04-08, AT-07-04, AT-07-08, AT-07-09 | „Krypto-Test“ → `422`, ETHA genau einmal (Simulation und Vergleich); Horizont 0 → Meldung am Feld und „Bitte die markierten Eingaben prüfen.“, keine Anfrage; EUNL → `422` mit Hinweis auf eine andere Börse |
| Messung | AT-07-13 | 20 Vergleiche „Messportfolio“ gegen „Messportfolio“, 40 Jahre, 10.000 Pfade: 2.957 bis 3.045 ms, 20 von 20 ≤ 6.000 ms |
| Anlagensuche | AT-02-10, AT-02-11 | `GET /assets/search?q=ETHA` → `200`, ETHA · NASDAQ · USD unter den Treffern; „E“ → Hinweis unter dem Suchfeld, keine Anfrage im Netzwerk-Tab |
| Header und GET | AT-ADR6-01 | `GET /portfolios` und `GET /simulations` mit `Authorization: Bearer …`, je `200`; Payload von `POST /simulations` nur `portfolioId` und `parameters` |
| Anmeldung | AT-ADR6-02, AT-ADR6-03 | vier ungültige Varianten → je `401`, „Bitte melde dich an.“; Logout → `204`, danach `401` |
| Zugriffsschutz | AT-ADR6-04 | Nutzer B: Portfolio 34 → `404`, „Portfolio nicht gefunden.“; Simulation 3 → `404`, „Simulation nicht gefunden.“; eigene Portfolios 0 |
| Alte Pfade | AT-ADR6-05 | im Browser `405` ohne lesbaren Body (CORS, H7); mit `curl`: `HTTP/1.1 405`, `METHOD_NOT_ALLOWED`, „Diese HTTP-Methode ist hier nicht erlaubt.“, `Allow: GET, DELETE, PUT` |
| Beispielsimulation | AT-09-01, AT-09-02 | privates Fenster: `GET /demo/portfolios` → `200` mit 900 und 901; `POST /demo/simulations` ohne `Authorization` → `200`, `id` `null`, Seed 631377, Median 69.490 €; Seed 42 zweimal: `timeline`, `estimates`, `metrics` identisch |
| Handyansicht | AT-NF10-01 | 360 px: Startseite (Gast), Simulation, Portfolios, Vergleich jeweils `true`; H5 und H6 |

---

## 6. Abweichungen vom Testkonzept

- **Testdaten:** „Mein Aktienportfolio“ wurde vor AT-07-02 aus „Mein Aktienportfolio 2“ wiederhergestellt (siehe Abschnitt 1).
- **AT-07-08:** Erwartet ist nach der Anpassung in Sprint 4 nur ETHA; IBIT wird nicht genannt (43 Monatswerte, Abnahmeprotokoll Sprint 3).
- **AT-ADR6-05:** Der Meldungstext wurde mit `curl` in PowerShell geprüft, weil der Browser den Body ohne CORS-Header nicht anzeigt; der `curl`-Aufruf lief mit einem Platzhalter statt eines Tokens. Die `405` entsteht vor der Prüfung des Tokens; im Browser wurde derselbe Aufruf mit gültigem Token ausgeführt.
- **Formulierung von Testfällen:** AT-09-01 erwartet eine Antwort „ohne `id`“; laut API-Design ist `id` vorhanden und `null`. AT-ADR6-01 erwartet `GET /portfolios/{id}`, das das Frontend nicht verwendet. AT-07-04 erwartet `400` von der API, die Oberfläche sendet aber keine Anfrage. Die Testfälle werden in Sprint 5 angepasst.
- **Nicht durchgeführt:** der Bedienbarkeitstest (AT-NF09-01), geplant am 26.10.2026.
- **API-Prüfungen und Messungen** wurden mit `fetch` in der Konsole der Entwicklertools durchgeführt.

---

## 7. Folgen für Sprint 5

- US-05, US-07 und US-09 sind abgenommen; Docs #6, #3 und #2 werden geschlossen. Docs #136 und #137 sind erledigt.
- Für H2 bis H7 werden Issues angelegt (FE bzw. BE). H5 wird in Sprint 5 behoben; H4 wird im Sprint Planning Sprint 5 geklärt.
- Der Bedienbarkeitstest (NF-09) wird am 26.10.2026 durchgeführt und hier nachgetragen.
- `testabdeckung-sprint-4.md` wird um die Werte je Klasse ergänzt (Gesamtwert laut CI: 333 von 344 Zeilen, 96,8 %).
- AT-09-01, AT-ADR6-01 und AT-07-04 werden wie in Abschnitt 6 beschrieben angepasst; für den Abruf eines eigenen Portfolios per `GET /portfolios/{id}` fehlt ein Test.
- Die Testfälle zum Change Request (US-11, US-12) werden in den Abnahmetestfällen Sprint 5 erstellt.
