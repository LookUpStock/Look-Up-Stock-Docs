# Abnahmeprotokoll – Sprint 6

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 6 (03.11.2026 – 09.11.2026), Nachtest-Sprint vor der Abgabe |
| **Issue** | Docs #214 Abnahmetests Sprint 6, Abnahmeprotokoll Sprint 6 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Datum der Abnahme** | 05.11.2026, 09:22 bis 10:47 Uhr |
| **Stories** | US-11 Nutzungskennzahlen anzeigen (Docs #156), in Sprint 5 nach B8 und B9 nicht abgenommen. Nachtest der Hinweise H6 und H8 bis H12 aus Sprint 5. Nachweise für NF-05 (mit B11) und NF-13. In Sprint 6 gibt es keine neuen User Stories |
| **Grundlagen** | [Abnahmetestfälle Sprint 6](abnahmetestfaelle-sprint-6.md), [Abnahmeprotokoll Sprint 5](abnahmeprotokoll-sprint-5.md), [Testkonzept](testkonzept.md) Version 6, `anforderungen/nfr_furps.md` Version 5, `anforderungen/kurzformat_sprint-4.md` (US-11, E37 präzisiert, E40), Sprint-Planning-Protokoll Sprint 6, API-Design (Stand 04.11.2026) |
| **KI-Nutzung** | Durchführung mit KI-Unterstützung vorbereitet und ausgewertet, Tests vom Product Owner selbst durchgeführt, Heroku-Nachweise von Carl Stibbe, Entscheidungen vom Product Owner getroffen, siehe KI-Log |

Die Status der einzelnen Testfälle stehen in `abnahmetestfaelle-sprint-6.md`: beim Nachtest in der Spalte „Ergebnis“, bei AT-11-08, AT-NF05-01 und AT-NF13-01 in der Spalte „Status“. Dieses Protokoll fasst das Ergebnis zusammen, begründet die Abnahmeentscheidung zu US-11 und listet die Befunde.

---

## 1. Testumgebung und Durchführung

| | |
|---|---|
| **Frontend** | `https://lookupstockfrontend-96a9998805cb.herokuapp.com`. Blöcke 1 bis 5 auf dem Stand `main` 9c33830 (04.11.2026, 18:07, Merge PR #82 zu FE #81, B11); Nachtest H13 auf `main` a15f6ad (05.11.2026, 10:38, Merge PR #84 zu FE #83), laut Heroku-Activity unmittelbar danach deployt |
| **core-service** | `https://lookupstock-da2951e42089.herokuapp.com` (Heroku-App `lookupstock`), Stand `main` 3cecca4 (04.11.2026, 18:08, Merge PR #136 zu BE #135, B11), deployt am 04.11.2026 um 18:10 Uhr (Release v46). Der Stand enthält B8 (BE #128) und BE #129; Deploy laut BE #130 von Carl Stibbe bestätigt |
| **marktdaten-service** | `https://lookupstock-marktdaten-12ca5b75c510.herokuapp.com` (Heroku-App `lookupstock-marktdaten`), Stand `main` 3cecca4, deployt am 04.11.2026 um 18:10 Uhr (Release v23) |
| **Datenbank** | Heroku Postgres, Schemas `public` (core-service) und `marktdaten` (marktdaten-service) |
| **Finanzdaten** | echte Twelve-Data-API, nur über den marktdaten-service |
| **Testnutzer** | `abnahme3.a@example.com` (Nutzer A) aus Sprint 3; `gf@example.com` (Nutzer M, Rolle `MANAGER` seit Sprint 5); für AT-11-03 neu registriert: `zaehltest-0511@example.com` |
| **Testdaten** | Portfolios von Nutzer A aus Sprint 3 bis 5: „Mein Weltportfolio“, „Mein Aktienportfolio“, „Krypto-Test“, „Messportfolio“, „Vorladen“. Während der Abnahme angelegt: Portfolio „zähltest“ (URTH 100 %) mit der gespeicherten Simulation „zähltest Simulation“ (Nutzer `zaehltest-0511`). Die Prüfungen im Portfolio-Editor (H9, H13) wurden nicht gespeichert |
| **Manuelle Tests** | Product Owner im Browser Brave (Chromium) im normalen und im privaten Fenster; Netzwerk-Tab mit Filter `search`; API-Prüfungen mit `fetch` in der Konsole der Entwicklertools; Geräteansicht bei 360 px; `curl.exe -I` in PowerShell; Heroku-Dashboard durch Carl Stibbe, Nachweis per Screenshot |
| **Automatisierte Tests** | „Backend CI (core-service)“ Lauf #104 auf `main` 3cecca4 erfolgreich (04.11.2026, 18:08): 312 Tests, 0 Fehler, 0 übersprungen; die JaCoCo-Prüfung der Kernlogik ist bestanden („All coverage checks have been met.“). „Backend CI (marktdaten-service)“ Lauf #8 auf `main` 3cecca4 erfolgreich: 37 Tests, 0 Fehler. „Frontend CI“ Lauf #90 auf `main` a15f6ad erfolgreich (05.11.2026, 10:38): 180 Tests, 0 Fehler. Alle in den Testfällen genannten Testmethoden sind im Code vorhanden; kein Test ist mit `@Disabled` abgeschaltet |

**Bewertungsregel.** Wie in Sprint 3 bis 5 (Testkonzept, Abschnitt 8): Ein manueller Testfall ist `bestanden`, wenn Oberfläche, API bzw. Datenbank das erwartete Ergebnis zeigen; ein automatisierter, wenn die zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren. `fehlgeschlagen` heißt, mindestens ein erwartetes Ergebnis trat nicht ein. Priorität der Befunde nach Testkonzept, Abschnitt 11.

**Zählung im November 2026.** Bis zum Deploy von BE #128 am 04.11.2026 um 18:10 Uhr wurde jede über die Oberfläche gespeicherte Simulation zweimal als „Durchgeführte Simulation“ erfasst (B8). Die Ereignisse in `usage_event` tragen keinen Bezug zu Nutzer oder Simulation (AT-11-07); die doppelten Einträge lassen sich deshalb nicht sicher von echten unterscheiden. Der Product Owner hat entschieden, den Wert nicht nachträglich zu korrigieren. Die Kennzahl „Durchgeführte Simulationen“ für November 2026 ist um die Zahl der bis dahin über die Oberfläche gespeicherten Simulationen zu hoch. AT-11-03 vergleicht deshalb nur die Differenz vor und nach den Schritten.

---

## 2. Ergebnis

| Bereich | Testfälle | bestanden | fehlgeschlagen | offen | Entscheidung |
|---|---|---|---|---|---|
| US-11 Nutzungskennzahlen anzeigen (Nachtest AT-11-01, AT-11-03, AT-11-06; neu AT-11-08) | 4 | 4 | 0 | 0 | **abgenommen** |
| Nachtest Hinweise (H6/H8, H9, H11, H12) | 4 | 4 | 0 | 0 | H6 und H8 bis H12 behoben |
| NF-05 (AT-NF05-01) | 1 | 1 | 0 | 0 | erfüllt, B11 behoben |
| NF-13 (AT-NF13-01) | 1 | 1 | 0 | 0 | erfüllt |
| **Gesamt** | 10 | 10 | 0 | 0 | |

Alle Testfälle sind bestanden. Der während der Abnahme gefundene Hinweis H13 wurde in derselben Abnahme behoben und nachgetestet; H14 hat der Product Owner bewusst in das Product Backlog übernommen (Abschnitt 4).

---

## 3. Begründung

### US-11 Nutzungskennzahlen anzeigen – abgenommen

Alle Testfälle sind bestanden; die Fehler B8 und B9 aus Sprint 5 sind behoben.

- **Dashboard (AT-11-01, Nachtest B9 und H10, bestanden):** Als M steht „Auswertungen“ in der Navigation genau einmal, auf der Startseite wie auf dem Dashboard; „App (SPA)“ ist vorhanden. Das Dashboard zeigt „November 2026 (vorläufig)“, „Erfassung seit 01.11.2026“ und die neun Kennzahlen in der Reihenfolge aus E39. Über der Tabelle steht kein leeres Hinweisfeld mehr (H10). Für Oktober 2026 erscheint „Für diesen Monat liegen keine Auswertungen vor.“ ohne Werte.
- **Zählung (AT-11-03, Nachtest B8, bestanden):** Ausgangsstand 2 | 10 | 4 | 30 | 3 | 28 | 2 | 8 | 14, danach 3 | 11 | 5 | 31 | 4 | 29 | 3 | 10 | 15. Registrierungen, Anmeldungen, angelegte Portfolios, durchgeführte Simulationen, gespeicherte Simulationen, Vergleiche, Beispielsimulationen und Nutzerkonten stiegen jeweils um genau 1, die Anlagensuchen um 2 (zwei erfolgreiche Anfragen `search?q=URTH` im Netzwerk-Tab, je eine im Portfolio-Editor und im Vergleich). „Durchgeführte Simulationen“ stieg nach Simulieren und Speichern um **1** (in Sprint 5 um 2); der Vergleich hat sie nicht erhöht. Alle Werte lagen nach 10 Sekunden vor; keine Aktion war spürbar verzögert.
- **Zugriffsschutz (AT-11-06, Nachtest B9, bestanden):** Nutzer A sieht weder auf den MPA-Seiten noch auf `dashboard.html` einen Eintrag „Auswertungen“; „App (SPA)“ ist vorhanden. `dashboard.html` zeigt „Dafür fehlt die Berechtigung.“ ohne Kennzahlen. `GET /api/v1/reports/usage` liefert mit dem Token von A `403` `FORBIDDEN` „Dafür fehlt die Berechtigung.“, ohne Header `401` `UNAUTHORIZED` „Bitte melde dich an.“, jeweils als JSON. Nach dem Abmelden leitet `dashboard.html` zur Anmeldeseite weiter.
- **Zählung beim Speichern (AT-11-08, bestanden):** automatisiert; die vier Tests aus BE #128 laufen in CI #104 erfolgreich.

### Nachtest der Hinweise

- **H6 und H8 (AT-NF10-01, bestanden):** Vergleich „Mein Weltportfolio“ gegen „Mein Aktienportfolio“ mit 100.000 € Startkapital, 1.000 € Sparrate, 40 Jahren und Seed 42: Die längste Beschriftung der gemeinsamen Y-Achse („8.000.000.000 €“) ist am Desktop und bei 360 px vollständig lesbar, ohne seitliches Scrollen. Auf der Simulationsseite ist „20.000.000 €“ vollständig lesbar. Dazu Hinweis H14.
- **H9 (Sichtprüfung H4, bestanden nach Behebung von H13):** Der leere Editor zeigt bei der Gewichtssumme „—“ ohne Farbe; bei 40 % rot „ungültig“ mit „Die Gewichtssumme muss genau 100 % ergeben. Aktuell: 40,00 %.“; „Mein Weltportfolio“ zeigt beim Bearbeiten sofort „gültig“. Bei 100 % und leerem Namen stand zunächst „ungültig“ (H13). Nach FE #83 zeigt der Editor bei 100 % ohne Namen „gültig“; beim Speichern erscheint „Der Portfolio-Name ist erforderlich.“ am Namensfeld und „Bitte zuerst die markierten Fehler beheben.“, gespeichert wird nicht.
- **H11 (Adresse der SPA, bestanden):** Der Link „App (SPA)“ führt zu `/app#/portfolios`; ADR-08 und die Abnahmetestfälle Sprint 5 nennen diese Adresse und erklären die Umleitung (Docs #213).
- **H12 (AT-ADR8-01, bestanden):** Die Portfolioliste der SPA zeigt „Vorladen: 1 Position“, „Mein Weltportfolio: 2 Positionen“ und „Messportfolio: 5 Positionen“.

### NF-05 (AT-NF05-01) – erfüllt

Ein Vortest am 04.11.2026 ergab über `http://` die Antwort `200` (B11). Nach BE #135 und FE #81 liefern alle drei Apps über `http://` `308 Permanent Redirect` mit `Location: https://…` auf dieselbe Adresse (05.11.2026, 08:22 bis 08:24 UTC). Über `https://` zeigt der Browser bei allen drei Apps „Verbindung ist sicher“ ohne Zertifikatswarnung. `/actuator/health` liefert bei beiden Diensten `{"groups":["liveness","readiness"],"status":"UP"}`.

### NF-13 (AT-NF13-01) – erfüllt

Laut den Screenshots von Carl Stibbe deployen alle drei Apps automatisch von `main`, mit der Option „Wait for GitHub checks to pass before deploy“; ein Merge mit roter CI wird also nicht deployt. Die letzten Releases sind „Deployed 3cecca43“ bei `lookupstock` (v46) und `lookupstock-marktdaten` (v23) sowie „Deployed a15f6ad9“ beim Frontend (v43), jeweils ohne manuellen Eingriff. Die Konfiguration steht nur in Konfigurationsvariablen (nur Namen geprüft): core-service `APP_BASE`, `DATABASE_URL`, `INTERNAL_API_KEY`, `MARKTDATEN_URL`; marktdaten-service `APP_BASE`, `DATABASE_URL`, `INTERNAL_API_KEY`, `TWELVE_DATA_API_KEY`; Frontend keine. Der Twelve-Data-Key liegt damit nur beim marktdaten-service (NF-06). Beide Dienste melden `UP` (AT-NF05-01).

---

## 4. Befunde

Priorität nach Testkonzept, Abschnitt 11. Die Nummerierung setzt Sprint 3 bis 5 fort.

| ID | Repo | Beschreibung | Testfälle | Priorität | Art |
|---|---|---|---|---|---|
| B11 | FE, BE | Aufrufe über `http://` wurden nicht auf `https://` umgeleitet; der Vortest am 04.11.2026 lieferte `200` (NF-05). Behoben mit BE #135 (Filter in beiden Diensten) und FE #81 (eigener Startserver `server.js`), jeweils `308` anhand von `X-Forwarded-Proto`. | AT-NF05-01 | hoch | Fehler, **behoben am 04.11.2026**, Nachtest bestanden |
| H13 | FE | Unter „Gewichtssumme“ stand bei 100 % rot „ungültig“, solange der Portfolio-Name leer war, weil die Anzeige den Status des ganzen Portfolios zeigte. Nutzer konnten das als falsche Gewichtung lesen. Behoben mit FE #83 (PR #84, Peter Orban): Die Anzeige bewertet nur die Gewichte; der Name wird am Namensfeld gemeldet. | Sichtprüfung H4 | niedrig | Hinweis, **behoben am 05.11.2026**, Nachtest bestanden |
| H14 | FE | Im Vergleich teilen sich beide Fächerdiagramme eine Y-Achse (H1). Bei sehr unterschiedlichen Endwerten („Mein Aktienportfolio“ bis 6,7 Mrd. €) erscheint der kleinere Eintrag als flache Linie. Die gemeinsame Achse ist für den fairen Vergleich gewollt. Entscheidung des Product Owners: kein Umbau vor der Abgabe; ein Umschalter zwischen gemeinsamer und eigener Achse (oder eine logarithmische Achse) wird in das Product Backlog aufgenommen. | AT-NF10-01 | niedrig | Hinweis, **bewusst nicht umgesetzt** |

---

## 5. Nachweise der manuellen Tests

| Prüfung | Testfälle | Beobachtung |
|---|---|---|
| HTTPS | AT-NF05-01 | drei `curl.exe -I` über http: jeweils `308 Permanent Redirect`, `Location: https://…`; drei Seiten über https: „Verbindung ist sicher“; Health beider Dienste `UP` |
| Dashboard als M | AT-11-01 | „Auswertungen“ einmal, „App (SPA)“ vorhanden, kein leeres Hinweisfeld; November 2026 vorläufig, Erfassung seit 01.11.2026, neun Kennzahlen; Oktober → Meldung ohne Werte |
| Zähltest | AT-11-03 | 2 \| 10 \| 4 \| 30 \| 3 \| 28 \| 2 \| 8 \| 14 → 3 \| 11 \| 5 \| 31 \| 4 \| 29 \| 3 \| 10 \| 15; zwei Anfragen `search?q=URTH` mit 200; gespeicherte Simulation „zähltest Simulation“ (05.11.2026, 09:34); Beispielsimulation einmal (Seed 609252) |
| Ohne Rolle | AT-11-06 | Navigation von A ohne „Auswertungen“; `dashboard.html` „Dafür fehlt die Berechtigung.“; API `403` bzw. `401` als JSON; abgemeldet → Anmeldeseite |
| Fächerdiagramme | AT-NF10-01 | Vergleich: Y-Achse bis „8.000.000.000 €“ lesbar, Desktop und 360 px; Simulation: bis „20.000.000 €“ lesbar |
| Editor | Sichtprüfung H4 (H9, H13) | leer „—“; 40 % „ungültig“ mit Meldung; „Mein Weltportfolio“ „gültig“; nach FE #83: 100 % ohne Namen „gültig“, Speichern → Meldung am Namensfeld, nichts gespeichert |
| SPA | AT-ADR8-01 (H11, H12) | Adresse `/app#/portfolios`; „1 Position“, „2 Positionen“, „5 Positionen“ |
| Heroku (Carl Stibbe) | AT-NF13-01 | automatische Deploys von `main` mit Warten auf CI; Releases 3cecca43 (v46, v23) und a15f6ad9 (v43); Namen der Config Vars |

---

## 6. Abweichungen vom Testkonzept

- **AT-11-03:** Das Portfolio wurde als „zähltest“ statt „Zähltest“ angelegt, die Simulation als „zähltest Simulation“. Auf die Zählung hat das keinen Einfluss.
- **Sichtprüfung H4:** Die Schritte nannten keinen Portfolio-Namen. Mit leerem Namen zeigte sich H13; die Prüfung wurde nach der Behebung mit leerem Namen wiederholt und zusätzlich das Speichern geprüft.
- **H13 während der Abnahme behoben:** Befund um 10:20 Uhr, Merge von FE #83 um 10:38 Uhr, automatischer Deploy, Nachtest um 10:47 Uhr. Die übrigen Testfälle betrifft die Änderung nicht; der neutrale Anfangszustand aus H9 wurde nach dem Deploy erneut geprüft.
- **AT-NF13-01:** Die Activity-Einträge liegen als kopierter Text vor, die Deploy- und Config-Var-Ansichten als Screenshots. Bei den beiden Backend-Apps ist im Screenshot der Bereich „Automatic deploys“ zu sehen, nicht die Zeile mit dem verbundenen Repository; die Zuordnung ergibt sich aus dem deployten Commit 3cecca43 des Backend-Repositorys.
- **Heroku:** Der Product Owner hat keinen Zugang; die Prüfungen hat Carl Stibbe durchgeführt und per Screenshot belegt.
- **API-Prüfungen** wurden mit `fetch` in der Konsole der Entwicklertools durchgeführt.

---

## 7. Folgen

- **US-11 (Docs #156) ist abgenommen** und wird mit „Abgenommen am 05.11.2026, siehe Abnahmeprotokoll Sprint 6“ geschlossen. Damit sind alle umgesetzten User Stories US-01 bis US-09, US-11 und US-12 abgenommen; US-10, US-13 und „Vergleiche speichern“ sind bewusst nicht umgesetzt (Won't, Change Request CR-01).
- **B8, B9 und B11 sind behoben;** H6 und H8 bis H13 ebenfalls.
- **H14** geht als Hinweis mit Priorität niedrig in das Product Backlog.
- **NF-05 und NF-13** sind nachgewiesen.
- **Docs #214** ist mit diesem Protokoll erledigt.
