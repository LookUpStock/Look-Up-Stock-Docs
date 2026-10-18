# Abnahmeprotokoll – Sprint 3

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 3 (12.10.2026 – 18.10.2026) |
| **Issue** | Docs #101 Abnahmetests Sprint 3 durchführen, Nachtest der Sprint-2-Befunde |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Datum der Abnahme** | 18.10.2026, 21:10 bis 22:26 Uhr |
| **Stories** | Neu: US-05 Kennzahlen anzeigen (Docs #6), US-06 Portfolio bearbeiten und löschen (Docs #5), US-07 Anlagen und Portfolios vergleichen (Docs #3, Kern-Story), US-08 Simulation speichern und aufrufen (Docs #4). Nachtest: US-02 (Docs #8), US-03 (Docs #1), US-04 (Docs #7, Kern-Story) aus Sprint 2 |
| **Grundlagen** | [Abnahmetestfälle Sprint 1](abnahmetestfaelle-sprint-1.md), [Sprint 2](abnahmetestfaelle-sprint-2.md) und [Sprint 3](abnahmetestfaelle-sprint-3.md), [Abnahmeprotokoll Sprint 2](abnahmeprotokoll-sprint-2.md), [Testkonzept](testkonzept.md) Version 3 (Abschnitte 8 und 11), `anforderungen/kurzformat_sprint-3.md` Version 2 |
| **KI-Nutzung** | Durchführung mit KI-Unterstützung vorbereitet und ausgewertet, Tests vom Product Owner selbst durchgeführt, Entscheidungen vom Product Owner getroffen, siehe KI-Log |

Die Status der einzelnen Testfälle stehen in der Spalte „Status“ von `abnahmetestfaelle-sprint-3.md`; die Ergebnisse des Nachtests in derselben Datei im Abschnitt „Nachtest der Befunde aus Sprint 2“. Dieses Protokoll fasst das Ergebnis je Story zusammen, begründet die Abnahmeentscheidung und listet die gefundenen Fehler.

---

## 1. Testumgebung und Durchführung

| | |
|---|---|
| **Frontend** | `https://lookupstockfrontend-96a9998805cb.herokuapp.com`, Stand `main` f72f28f (18.10.2026, 21:44, Hinweise E16/E21); Bearbeiten und Löschen seit PR #28 (21:13) |
| **Backend** | `https://lookupstock-da2951e42089.herokuapp.com`, Stand `main` d9c02ab (18.10.2026, 12:18, Merge PR #85 zu BE #68) |
| **Datenbank** | Heroku Postgres |
| **Finanzdaten** | echte Twelve-Data-API; ein `MockMarketDataProvider` ist im Code nicht vorhanden (Testkonzept Version 3, Abschnitt 6) |
| **Testnutzer** | `abnahme3.a@example.com` / `geheim123` (Nutzer A), `abnahme3.b@example.com` / `sicher456` (Nutzer B), am 18.10.2026 neu registriert |
| **Testdaten** | Portfolios von Nutzer A nach PO-Entscheidung vom 18.10.2026 mit Anlagen an US-Börsen: „Mein Weltportfolio“ (URTH 70 %, IEMG 30 %), „Mein Aktienportfolio“ (AAPL 50 %, MSFT 50 %), „Krypto-Test“ (IBIT 50 %, ETHA 50 %), „Messportfolio“ (URTH, IEMG, AAPL, MSFT, NVDA je 20 %), „Löschtest“ (AAPL 100 %) |
| **Manuelle Tests** | Product Owner im Browser (Chrome); API-Prüfungen und Messungen mit `fetch` in der Konsole der Entwicklertools |
| **Automatisierte Tests** | Backend: „Backend CI“ auf `main` d9c02ab erfolgreich (18.10.2026, 12:19). Frontend: „Frontend CI“ auf `main` f72f28f erfolgreich (18.10.2026, 21:44); seit F7 führt die CI `npm ci` und `npm test` aus |

**Bewertungsregel.** Wie in Sprint 2 (Testkonzept, Abschnitt 8): Ein manueller Testfall ist `bestanden`, wenn Oberfläche und API das erwartete Ergebnis zeigen; ein automatisierter, wenn die zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren. `offen` heißt, der Testfall wurde nicht oder nur teilweise durchgeführt.

**Änderungen während der Abnahme.** Carl Stibbe hat während der Abnahme Bearbeiten und Löschen von Portfolios (PR #28, 21:13) und die Hinweise E16/E21 auf der Simulationsseite (21:44) auf `main` gebracht. Die betroffenen Testfälle wurden danach durchgeführt bzw. wiederholt (AT-04-15).

---

## 2. Ergebnis je Story

| Story | Testfälle | bestanden | fehlgeschlagen | offen | Entscheidung |
|---|---|---|---|---|---|
| US-02 Anlagen suchen (Nachtest) | 9 | 9 | 0 | 0 | **abgenommen** |
| US-03 Portfolio mit Gewichtungen erstellen (Nachtest) | 11 | 11 | 0 | 0 | **abgenommen** |
| US-04 Monte-Carlo-Simulation durchführen (Nachtest, Kern-Story) | 16 | 16 | 0 | 0 | **abgenommen** |
| US-05 Kennzahlen anzeigen | 6 | 5 | 1 | 0 | **nicht abgenommen** |
| US-06 Portfolio bearbeiten und löschen | 6 | 6 | 0 | 0 | **abgenommen** |
| US-07 Anlagen und Portfolios vergleichen (Kern-Story) | 13 | 6 | 1 | 6 | **nicht abgenommen** |
| US-08 Simulation speichern und aufrufen | 7 | 7 | 0 | 0 | **abgenommen** |
| NF-09, NF-10 | 2 | 0 | 0 | 2 | – |
| **Gesamt** | 70 | 60 | 2 | 8 | |

US-02 bis US-04 umfassen die Testfälle aus Sprint 1 und die Ergänzungen aus Sprint 2; die Zahlen geben den Stand nach dem Nachtest wieder.

---

## 3. Begründung

### US-02 Anlagen suchen – abgenommen

F1 und F5 sind behoben: Ein angeklickter Suchtreffer wird als Position in den Portfolio-Editor übernommen (AT-02-03), und 20 Suchen aus dem Cache antworten in höchstens 58 ms (AT-02-08, NF-02). Der API-Ausfall (AT-02-07) ist laut Testkonzept Version 3 nur automatisiert prüfbar und dort bestanden.

**Festlegung des Product Owners:** Die Suche nach dem Ticker „ETHA“ lieferte keinen Treffer, die Suche nach dem Namen „iShares Ethereum“ schon (B6). Die Suche nach Ticker, ISIN und Name funktioniert für alle übrigen Testanlagen. Die Ursache liegt vermutlich in der Symbolsuche von Twelve Data; sie wird in Sprint 4 analysiert und verhindert die Abnahme nicht.

### US-03 Portfolio mit Gewichtungen erstellen – abgenommen

F2 ist behoben: Portfolios werden im Backend gespeichert, erscheinen mit Anzahl der Positionen und Datum in der Liste, sind nach erneuter Anmeldung vorhanden und in der Simulation auswählbar (AT-03-01, AT-03-07, AT-03-10). Fünf Portfolios mit bis zu fünf Positionen wurden angelegt.

### US-04 Monte-Carlo-Simulation durchführen – abgenommen

Alle Testfälle sind bestanden. Fächerdiagramm, Seed, Pfadanzahl und Schätzzeitraum werden angezeigt (AT-04-01 bis AT-04-03); eine ungültige Eingabe wird am Feld gemeldet und kann ohne neue Portfolioauswahl korrigiert werden (AT-04-04). F3 und F4 sind behoben: Anlagen an US-Börsen liefern Kurse, und der erste Lauf mit frischen Kursen ergibt dasselbe Ergebnis wie die folgenden Läufe aus dem Cache (AT-04-13). Die Hinweise E16/E21 stehen seit 21:44 unter dem Ergebnis (AT-04-15). NF-01 ist mit 20 von 20 Läufen unter 3 Sekunden erfüllt (Maximum 1.602 ms bei 5 Anlagen, 40 Jahren und 10.000 Pfaden; AT-04-14).

**Hinweis zu AT-04-08:** Im „Krypto-Test“ wurde nur ETHA als zu kurz gemeldet. Twelve Data liefert für IBIT 43 Monatswerte (ab 09/2022); die Annahme in den Testdaten war falsch, das System hat richtig reagiert. Dass alle betroffenen Anlagen genannt werden (E6), ist automatisiert belegt (`ParameterEstimatorTest.alleBetroffenenAnlagenWerdenGenannt`).

**Befund ohne Einfluss auf die Abnahme:** Die Monatsreihen enthalten den laufenden Monat (neuester Wert 01.10.2026 beim Abruf am 18.10.2026), obwohl laut E22 nur abgeschlossene Monate zählen (B5). E22 ist kein Akzeptanzkriterium von US-04; der Fehler wird in Sprint 4 behoben.

### US-05 Kennzahlen anzeigen – nicht abgenommen

Einzahlungen, die drei Endwerte, Value at Risk und maximaler Drawdown werden korrekt angezeigt und stimmen mit dem letzten Verlaufspunkt überein (AT-05-01); gleicher Seed liefert identische Kennzahlen (AT-05-06). Das Backend berechnet die Zielwahrscheinlichkeit korrekt (AT-05-02, API-Teil: 97,9 % für 100.000 €). In der Oberfläche gibt es aber **kein Eingabefeld für den Zielbetrag**, die Zielwahrscheinlichkeit wird deshalb nie angezeigt (B1). Damit ist ein Akzeptanzkriterium nicht erfüllt.

### US-06 Portfolio bearbeiten und löschen – abgenommen

Name, Anlagen und Gewichtungen lassen sich ändern, ohne dass ein zweiter Eintrag entsteht (AT-06-01); eine Summe von 90 % wird abgelehnt und das Portfolio bleibt unverändert (AT-06-02); Bearbeiten nutzt dieselben Prüfregeln wie das Erstellen (AT-06-03, automatisiert). Vor dem Löschen erscheint eine Bestätigung; Abbrechen lässt das Portfolio stehen (AT-06-04). Mit dem Portfolio werden seine gespeicherten Simulationen gelöscht (AT-06-05, E28). Nutzer B kann Portfolios von A weder ändern noch löschen (AT-06-06, NF-04).

### US-07 Anlagen und Portfolios vergleichen – nicht abgenommen

Der Vergleich eines Portfolios mit einer einzelnen Anlage funktioniert: Bezeichnungen, Kennzahlen nebeneinander und die Wahrscheinlichkeit, dass der erste Eintrag besser abschneidet (41,1 %), werden angezeigt; fremde Portfolios liefern `404` (AT-07-06). Es fehlen aber die **Anzeige des verwendeten Seeds und des gemeinsamen Schätzzeitraums** (B2) sowie die **Eingabe einer eigenen Bezeichnung** und des Zielbetrags (B3, B1). Sechs manuelle Testfälle wurden deshalb nicht durchgeführt; sie folgen nach der Behebung.

**Festlegung des Product Owners (E35):** Die beiden Verläufe werden absichtlich in zwei Diagrammen nebeneinander statt in einem gemeinsamen Diagramm dargestellt. Das Akzeptanzkriterium wurde in `kurzformat_sprint-3.md` Version 2 angepasst; AT-07-01 wurde gegen die angepasste Fassung bewertet. Empfehlung: dieselbe Y-Achse für beide Diagramme (H1).

### US-08 Simulation speichern und aufrufen – abgenommen

Speichern mit Namen, Ablehnung ohne Namen, Liste mit Portfolio und Uhrzeit (neueste zuerst), Öffnen ohne Neuberechnung und Löschen mit Bestätigung funktionieren (AT-08-01 bis AT-08-04). Nach einer Änderung des Portfolios zeigt die gespeicherte Simulation unverändert dieselben Werte; der Vergleich über die API ergab ein identisches Ergebnis (AT-08-02). Nutzer B sieht keine Simulationen von A und kann sie weder abrufen noch löschen (AT-08-06). Die Namenslänge (AT-08-05) und das Speichern nur mit `save: true` (AT-08-07) sind automatisiert belegt. Vergleiche speichern ist nach Sprint 4 verschoben (E34).

---

## 4. Gefundene Fehler und Hinweise

Priorität nach Testkonzept, Abschnitt 11.

| ID | Repo | Beschreibung | Testfälle | Priorität | Art |
|---|---|---|---|---|---|
| B1 | FE | Auf der Simulations- und der Vergleichsseite gibt es kein Eingabefeld für den Zielbetrag; die Zielwahrscheinlichkeit wird nie angezeigt. Das Backend liefert sie bereits (`targetAmount`, `metrics.targetProbability`). | AT-05-02, AT-07-02 | hoch | Bug |
| B2 | FE | Die Vergleichsseite zeigt weder den verwendeten Seed noch den gemeinsamen Schätzzeitraum an (`parameters.seed`, `estimates.historyFrom`/`historyTo` sind in der Antwort enthalten). Ohne Seed ist ein Ergebnis nicht reproduzierbar. | AT-07-01 | hoch | Bug |
| B3 | FE | Auf der Vergleichsseite kann je Eintrag keine eigene Bezeichnung eingegeben werden; es wird immer der Name verwendet (`label` wird nicht abgefragt). | AT-07-02 | hoch | Bug |
| B4 | FE | Die Hinweise E16/E21 fehlen auf der Vergleichsseite (`compare.html`); laut NF-14 (a) gehören sie auf jede Ergebnisseite. | – | mittel | Bug |
| B5 | BE | Die Monatsreihen enthalten den laufenden, noch nicht abgeschlossenen Monat (neuester Wert 01.10.2026 beim Abruf am 18.10.2026); E22 verlangt nur abgeschlossene Monate. Betrifft Schätzung und Zählung der 36 Monatswerte. | – | mittel | Bug |
| B6 | BE/FE | Die Suche nach dem Ticker „ETHA“ liefert keinen Treffer, die Suche nach dem Namen schon; dabei erscheint auch kein Hinweis „Keine passende Anlage gefunden.“. Ursache (Symbolsuche von Twelve Data oder Anzeige) ist zu analysieren. | – | niedrig | Analyse |
| B7 | BE/FE | Die Meldung bei zu kurzer Historie nennt die Anlage doppelt („…: iShares Ethereum Trust ETF. iShares Ethereum Trust ETF: Weniger als 3 Jahre Kursdaten.“), weil das Frontend `message` und `details` aneinanderhängt. | AT-04-08 | niedrig | Bug |
| H1 | FE | Empfehlung zu E35: Die beiden Diagramme im Vergleich haben unterschiedliche Y-Achsen (z. B. bis 500.000 € und bis 800.000 €), dadurch wirken die Verläufe ähnlicher, als sie sind. Eine gemeinsame Skala macht den Vergleich aussagekräftiger. | – | – | Hinweis |

**Hinweise zum Vorgehen (für Review und Retrospektive):**

- FE #19 wurde am 17.10. geschlossen, obwohl die Oberfläche erst am 18.10. um 21:13 gemergt wurde; das gleichnamige Backend-PR war bereits gemergt.
- Die Hinweise E16/E21 wurden direkt auf `main` gepusht (f72f28f), ohne Pull Request und Review (CONTRIBUTING.md).
- Das Klassendiagramm v3 ist gemergt, entspricht aber nicht dem Code (Vergleich noch „geplant“, `ComparisonItem` statt `ComparisonEntry`, Klassen von US-08 fehlen); Klassen- und ER-Diagramm v2 liegen noch nicht im Archiv.
- BE #53 ist geschlossen, ein `MockMarketDataProvider` existiert im Code aber nicht; Ausfälle der Finanzdaten-API sind deshalb nur automatisiert prüfbar.
- Für `POST /comparisons` mit nicht erreichbarem Anbieter (`503`) und für `PUT /portfolios/{id}` ohne Token gibt es keine eigenen Tests.
- Die Diagramme zu US-07 (Aktivitäts- und Sequenzdiagramm v1) beschreiben noch „beide Verläufe in einem Diagramm“ und werden an E35 angepasst.

---

## 5. Nachweise der manuellen Tests

| Prüfung | Testfälle | Beobachtung |
|---|---|---|
| Portfolios anlegen | AT-02-03, AT-03-01, AT-03-08 | URTH (NYSE), IEMG, AAPL, MSFT, IBIT, ETHA, NVDA per Klick übernommen; 5 Portfolios gespeichert, Liste mit Positionen und Datum |
| Kennzahlen | AT-05-01, AT-05-06, AT-04-13 | „Mein Weltportfolio“, 10.000 €, 200 €, 20 Jahre, 10.000 Pfade, Seed 42: Eingezahlt 58.000 €, p5 113.748 €, Median 218.380 €, p95 429.444 €, VaR 0 €, Drawdown 16,17 %; API-Endwerte gleich Verlaufspunkt Monat 240; zweiter Lauf identisch |
| Zielbetrag über die API | AT-05-02 | `targetAmount` 100000 → `targetProbability` 0,9791, übrige Kennzahlen unverändert |
| Speichern und Öffnen | AT-08-01, AT-08-02, AT-08-04 | ohne Namen Meldung am Feld bzw. `400`; „Sparplan 20 Jahre“ (21:37) und „Kurz 10 Jahre“ (21:38); nach Änderung von „Mein Weltportfolio“ auf 50/50 zeigt „Sparplan 20 Jahre“ dieselben Werte, `runtimeMs` 359, API-Vergleich `true` |
| Bearbeiten und Löschen | AT-06-01, AT-06-02, AT-06-04, AT-06-05, AT-08-03 | „Portfolio aktualisiert.“, kein Duplikat; 90 % abgelehnt; Bestätigungsdialoge bei Portfolio und gespeicherter Simulation, Abbrechen und OK geprüft; „Löschtest-Simulation“ mit dem Portfolio gelöscht |
| Zugriffsschutz | AT-06-06, AT-07-06, AT-08-06 | Nutzer B: Listen leer; `PUT`/`DELETE` Portfolio 34 → `404`; `details`/`DELETE` Simulation 3 → `404`; Vergleich mit Portfolio 34 → `404`; danach bei A alles unverändert |
| Vergleich | AT-07-01 | „Mein Weltportfolio“ gegen URTH (NYSE) ohne Seed: zwei Fächerdiagramme, Kennzahlen nebeneinander, „schneidet in 41,1 % der gemeinsamen Simulationspfade besser ab“; kein Seed und kein Schätzzeitraum sichtbar |
| Simulation, Nachtest | AT-04-01 bis AT-04-04, AT-04-08, AT-04-15 | Horizont 0 → Meldung am Feld, nach Korrektur Ergebnis; „Krypto-Test“ → `422` mit ETHA; Hinweise E16/E21 unter dem Ergebnis |
| Messungen | AT-04-14, AT-02-08 | „Messportfolio“, 40 Jahre, 10.000 Pfade: 20 Läufe 1.497 bis 1.602 ms; 20 Suchbegriffe aus dem Cache: 48 bis 58 ms |

---

## 6. Abweichungen vom Testkonzept

- **Testdaten:** „Löschtest“ wurde vor AT-06-05 versehentlich ohne gespeicherte Simulation gelöscht und neu angelegt; AT-06-04 wurde mit „Löschtest“ statt „Mein Aktienportfolio 2“ durchgeführt. IBIT ist entgegen der Annahme nicht zu kurz (siehe US-04).
- **Testnutzer:** neue Konten `abnahme3.a@example.com` und `abnahme3.b@example.com`.
- **Nicht durchgeführt:** sechs manuelle Testfälle zu US-07 (wegen B1 bis B3), der Bedienbarkeitstest (AT-NF09-01) und die Prüfung bei 360 px (AT-NF10-01).
- **API-Prüfungen und Messungen** wurden mit `fetch` in der Konsole der Entwicklertools durchgeführt.

---

## 7. Folgen für Sprint 4

- US-02, US-03, US-04, US-06 und US-08 sind abgenommen; ihre Story-Issues werden geschlossen bzw. bleiben geschlossen.
- US-05 und US-07 werden in das Sprint Backlog von Sprint 4 übernommen.
- Für B1 bis B7 werden Issues angelegt (Bugs mit Label `bug` in FE bzw. BE, B6 als Analyse-Task). Nach dem Merge werden AT-05-02, AT-07-01 bis AT-07-04, AT-07-08, AT-07-09 und AT-07-13 durchgeführt (Testkonzept, Abschnitt 11).
- Der Bedienbarkeitstest (NF-09) und die Prüfung bei 360 px (NF-10) werden zu Beginn von Sprint 4 nachgeholt.
- Aktivitäts- und Sequenzdiagramm US-07 werden an E35 angepasst.
