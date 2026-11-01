# Abnahmetestfälle – Sprint 3

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 3 (12.10.2026 – 18.10.2026) |
| **Issue** | Docs #100 Abnahmetestfälle Sprint 3 und Testkonzept v3 (US-05 bis US-08, NF-09) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 18.10.2026; Status eingetragen am 18.10.2026 (Docs #101); AT-07-04 angepasst am 01.11.2026 (Docs #168) |
| **Durchführung** | 18.10.2026 durch den Product Owner; Ergebnis und Abnahmeentscheidung im [Abnahmeprotokoll Sprint 3](abnahmeprotokoll-sprint-3.md) |
| **Stories** | US-05 Kennzahlen anzeigen (Docs #6), US-07 Anlagen und Portfolios vergleichen (Docs #3, Kern-Story), US-06 Portfolio bearbeiten und löschen (Docs #5, Stretch), US-08 Simulation speichern und aufrufen (Docs #4, Stretch); Nachtest der Befunde aus Sprint 2 (F1 bis F7) |
| **Grundlagen** | Akzeptanzkriterien und Festlegungen E24 bis E34 in `anforderungen/kurzformat_sprint-3.md`, Sprint-Planning-Protokoll Sprint 3, Aktivitätsdiagramm US-07 v1, ADR-05, API-Design (Portfolios, Simulationen, Vergleiche), NFRs (NF-01, NF-04, NF-08, NF-09, NF-10), Abnahmeprotokoll Sprint 2, Backend-Umsetzung BE #67 bis #71 und Frontend FE #16, #17, #20 (Stand 18.10.2026), [Testkonzept](testkonzept.md) Version 3 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt [abnahmetestfaelle-sprint-2.md](abnahmetestfaelle-sprint-2.md) fort. Sie enthält die neuen Testfälle für US-05 bis US-08, den Bedienbarkeitstest (NF-09) und die Vorgaben für den Nachtest der in Sprint 2 nicht bestandenen Testfälle.

---

## Vorbemerkungen

Es gelten die Vorbemerkungen aus Sprint 1 und Sprint 2 (Bezug, Durchführung, Darstellung von UI und API, Status, Anmeldung, Meldungstexte). Zusätzlich gilt:

- **Abnahmeumgebung:** Frontend und Backend auf Heroku mit der echten Twelve-Data-API. Einen `MockMarketDataProvider` mit Ausfall-Schalter gibt es im Code nicht (Stand 18.10.2026). Der Ausfall der Finanzdaten-API und zu kurze Überschneidungen von Kurshistorien sind deshalb nur automatisiert prüfbar.
- **Basisportfolio (PO-Entscheidung vom 18.10.2026):** Twelve Data liefert im kostenlosen Tarif keine Kurse für Xetra (F3). Für alle manuellen Testfälle ab Sprint 3 werden deshalb Anlagen an US-Börsen verwendet:

  | Testdaten | Inhalt | Zweck |
  |---|---|---|
  | **„Mein Weltportfolio“** (Nutzer A) | URTH (iShares MSCI World ETF, NYSE Arca) 70 %, IEMG (iShares Core MSCI Emerging Markets ETF, NYSE Arca) 30 % | Basisportfolio für Simulation, Kennzahlen, Vergleich, Speichern |
  | **„Mein Aktienportfolio“** (Nutzer A) | AAPL (Apple, NASDAQ) 50 %, MSFT (Microsoft, NASDAQ) 50 % | zweites Portfolio für den Vergleich |
  | **„Krypto-Test“** (Nutzer A) | IBIT (iShares Bitcoin Trust, NASDAQ, Handel seit Januar 2024) 50 %, ETHA (iShares Ethereum Trust, NASDAQ, Handel seit Juli 2024) 50 % | zwei Anlagen mit weniger als 36 Monatswerten (E6, E8). *Bei der Durchführung festgestellt:* Twelve Data liefert für IBIT 43 Monatswerte (ab 09/2022), nur ETHA (28) ist zu kurz; ETHA wird über die Suche nur mit dem Namen „iShares Ethereum“ gefunden |
  | **„Messportfolio“** (Nutzer A) | URTH, IEMG, AAPL, MSFT, NVDA (NVIDIA, NASDAQ) je 20 % | Messfall mit 5 Anlagen (NF-01, E33) |
  | Einzelanlage URTH | über die Anlagensuche | Vergleich mit einer einzelnen Anlage (E30) |
  | Einzelanlage EUNL (Xetra) | über die Anlagensuche | Anlage ohne Kurse beim Anbieter (Docs #80) |

- **Basiseingaben:** Startkapital 10.000 €, Sparrate 200 €, Anlagehorizont 20 Jahre, 10.000 Pfade. Damit betragen die Einzahlungen 58.000 € (E19). Wo ein Seed genannt ist, wird `42` verwendet; wo ein Zielbetrag genannt ist, `100000`.
- **Testnutzer:** Nutzer A und Nutzer B wie in Sprint 2; die Portfolios oben legt Nutzer A vor der Abnahme an. Nutzer B hat kein Portfolio und keine gespeicherte Simulation.
- **Handyansicht (NF-10):** Jede neue oder geänderte Seite wird zusätzlich bei 360 px Breite geprüft (AT-NF10-01).

**Anzahl der neuen Testfälle**

| Story | Normalfall | Alternativszenario | Grenzwert | NFR | Summe | davon manuell | davon automatisiert |
|---|---|---|---|---|---|---|---|
| US-05 | 2 | 0 | 2 | 2 | 6 | 4 | 2 |
| US-06 | 2 | 3 | 0 | 1 | 6 | 5 | 1 |
| US-07 | 3 | 7 | 1 | 2 | 13 | 8 | 5 |
| US-08 | 3 | 2 | 1 | 1 | 7 | 5 | 2 |
| NF-09, NF-10 | 0 | 0 | 0 | 2 | 2 | 2 | 0 |
| **Gesamt** | 10 | 12 | 4 | 8 | 34 | 24 | 10 |

Bei Testfällen mit „manuell und automatisiert“ zählt die manuelle Durchführung; der automatisierte Teil ergänzt sie.

---

## US-05 Kennzahlen anzeigen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-05-01 | AK: Einzahlungen und Endwerte p5, p50, p95; AK: Value at Risk und maximaler Drawdown; E24, E27 (ohne Zielbetrag) | Normalfall | manuell | Als Nutzer A „Mein Weltportfolio“ mit den Basiseingaben und Seed 42 ohne Zielbetrag simulieren | **UI:** Unter dem Fächerdiagramm stehen „Eingezahlt“ 58.000 €, ungünstiger Endwert (p5), Median-Endwert, günstiger Endwert (p95), Value at Risk (95 %) in Euro und maximaler Drawdown (Median) in Prozent; es erscheint **keine** Zielwahrscheinlichkeit.<br>**API:** `POST /simulations` → `200 OK`; `metrics.totalContributions` = 58000; `p5EndValue`, `medianEndValue` und `p95EndValue` gleich `p5`, `p50` und `p95` des letzten Verlaufspunkts (Monat 240); `valueAtRisk95` = max(0, 58000 − `p5EndValue`); `targetProbability` = `null`. | bestanden (18.10.2026, manuell und API: Eingezahlt 58.000 €, Endwerte gleich Monat 240, `targetProbability` `null`) |
| AT-05-02 | AK: Wahrscheinlichkeit für den Zielbetrag; E27 | Normalfall | manuell | Wie AT-05-01, zusätzlich Zielbetrag 100.000 € | **UI:** Zusätzlich erscheint die Wahrscheinlichkeit, 100.000 € zu erreichen, als Prozentwert zwischen 0 % und 100 %; die übrigen Kennzahlen sind identisch mit AT-05-01.<br>**API:** `parameters.targetAmount` = 100000; `metrics.targetProbability` zwischen 0 und 1. | fehlgeschlagen ([B1](abnahmeprotokoll-sprint-3.md#4-gefundene-fehler-und-hinweise): kein Eingabefeld für den Zielbetrag); API-Teil bestanden (`targetProbability` 0,9791) |
| AT-05-03 | AK: Value at Risk und maximaler Drawdown werden im Glossar erklärt | NFR | manuell (Review) | `anforderungen/glossar.md` lesen | Die Einträge „Value at Risk (95 %)“, „Maximaler Drawdown“, „Endwert“, „Zielbetrag“ und „Zielwahrscheinlichkeit“ erklären die Kennzahlen verständlich und stimmen mit E25 bis E27 überein. | bestanden (Review Glossar Version 5) |
| AT-05-04 | E27: Zielbetrag optional, größer als 0 | Grenzwert | automatisiert (`SimulationParametersTest`) | `targetAmount` −1, 0, 0,01 und ohne Angabe | **API:** −1 und 0 → `400 VALIDATION_ERROR`, „Der Zielbetrag muss größer als 0 sein.“, `details` nennt `targetAmount`; 0,01 und ohne Angabe → gültig. | bestanden (`SimulationParametersTest`, CI auf `main` 18.10.2026) |
| AT-05-05 | E24 bis E26: Rechenregeln der Kennzahlen | Grenzwert | automatisiert (`MetricsCalculatorTest`) | Bekannte Pfade: Beispiel aus dem API-Design (58.000 € Einzahlungen); ungünstiger Endwert über den Einzahlungen; Pfad mit Höchststand 120 und Tiefstand 90; nur steigender Pfad; Startkapital 0 € | Einzahlungen 58.000 €; Value at Risk 0 €, wenn p5 über den Einzahlungen liegt; Drawdown 25 % bzw. 0 %; kein Fehler bei Startkapital 0 €; Endwerte gleich dem letzten Verlaufspunkt. | bestanden (`MetricsCalculatorTest`, CI) |
| AT-05-06 | AK: gleicher Seed → identische Kennzahlen; NF-08; E27 | NFR | manuell und automatisiert (`MetricsCalculatorTest`) | 1. AT-05-02 ein zweites Mal mit denselben Eingaben ausführen<br>2. Automatisiert: gleiche Pfade zweimal auswerten; Zielwahrscheinlichkeit mit und ohne Zielbetrag | 1. Alle Kennzahlen sind identisch mit dem ersten Lauf.<br>2. Identische Kennzahlen; mit Zielbetrag der Anteil der Pfade mit Endwert ≥ Zielbetrag, ohne Zielbetrag `null`. | bestanden (18.10.2026, zweiter Lauf mit Seed 42 identisch; `MetricsCalculatorTest`) |

---

## US-06 Portfolio bearbeiten und löschen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-06-01 | AK: Name, Anlagen und Gewichtungen ändern; AK: Liste zeigt geänderte Werte, `updatedAt` aktualisiert | Normalfall | manuell | Als Nutzer A „Mein Aktienportfolio“ bearbeiten: Name in „Mein Aktienportfolio 2“ ändern, MSFT entfernen, NVDA hinzufügen, Gewichtungen AAPL 60 % und NVDA 40 %; speichern; Seite neu laden | **UI:** Die Portfolioliste zeigt den neuen Namen und 2 Positionen; im Editor stehen AAPL 60 % und NVDA 40 %; in der Auswahl der Simulation erscheint der neue Name.<br>**API:** `PUT /portfolios/{id}` → `200 OK` mit den neuen Positionen; `updatedAt` liegt nach `createdAt`. | bestanden (18.10.2026: „Mein Aktienportfolio 2“ mit AAPL 60 % und NVDA 40 %, kein zweiter Eintrag) |
| AT-06-02 | AK: ungültige Änderungen werden abgelehnt, Portfolio bleibt unverändert; Prüfregeln wie US-03 | Alternativszenario | manuell | „Mein Weltportfolio“ bearbeiten: Gewichtungen URTH 70 % und IEMG 20 % (Summe 90 %); speichern; Seite neu laden | **UI:** Meldung „Die Gewichtungen müssen zusammen 100 % ergeben.“ (wie AT-03-02); nach dem Neuladen hat das Portfolio weiter URTH 70 % und IEMG 30 %.<br>**API:** `400 VALIDATION_ERROR`; `POST /portfolios/{id}/details` liefert die alten Werte. | bestanden (18.10.2026: Meldung „Die Gewichtssumme muss genau 100 % ergeben. Aktuell: 90,00 %.“ aus der Prüfung im Frontend wie beim Erstellen; Portfolio unverändert) |
| AT-06-03 | AK: dieselben Prüfregeln und Meldungen wie beim Erstellen | Alternativszenario | automatisiert (`PortfolioServiceTest`, `PortfolioControllerTest`) | Ändern mit ungültiger Gewichtung; Ändern über denselben Prüfweg wie das Anlegen | Ungültige Änderung → `400`, Portfolio unverändert (`update_ungueltigeGewichtung_laesstPortfolioUnveraendert`). Anlegen und Ändern nutzen dieselbe Prüfung (`buildValidated` und `Portfolio.validate()`); die Grenzwerte aus AT-03-02 bis AT-03-07 gelten damit auch beim Ändern (Code-Review). | bestanden (`PortfolioServiceTest`, `PortfolioControllerTest`, CI; gemeinsame Prüfung per Code-Review) |
| AT-06-04 | AK: Bestätigung vor dem Löschen; danach nicht mehr in der Liste und nicht mehr simulierbar | Normalfall | manuell | 1. „Mein Aktienportfolio 2“ löschen, Bestätigung **abbrechen**<br>2. Erneut löschen und bestätigen; Simulationsseite öffnen | 1. Das Portfolio bleibt in der Liste.<br>2. **UI:** Das Portfolio verschwindet aus der Liste und aus der Auswahl der Simulation.<br>**API:** `DELETE /portfolios/{id}` → `204 No Content`; danach `POST /simulations` mit dieser `portfolioId` → `404 NOT_FOUND`. | bestanden (18.10.2026, mit Portfolio „Löschtest“; Abbrechen und Bestätigen geprüft) |
| AT-06-05 | E28: gespeicherte Simulationen eines gelöschten Portfolios werden mitgelöscht | Alternativszenario | manuell | Als Nutzer A ein Portfolio „Löschtest“ (AAPL 100 %) anlegen, simulieren und als „Löschtest-Simulation“ speichern (US-08); danach das Portfolio löschen | **UI:** „Löschtest-Simulation“ erscheint nicht mehr in der Liste der gespeicherten Simulationen.<br>**API:** `POST /simulations/{id}/details` der gespeicherten Simulation → `404 NOT_FOUND`, „Simulation nicht gefunden.“ | bestanden (18.10.2026: „Löschtest-Simulation“ nach dem Löschen des Portfolios nicht mehr in der Liste) |
| AT-06-06 | AK: fremdes oder nicht vorhandenes Portfolio → 404, Daten unverändert; NF-04; Vorbedingung: angemeldet | NFR | manuell (API) und automatisiert (`PortfolioControllerTest`, `PortfolioServiceTest`) | 1. Mit dem Token von Nutzer B: `PUT` und `DELETE` auf „Mein Weltportfolio“ von A<br>2. Automatisiert zusätzlich: `DELETE` ohne Token | 1. Beide → `404 NOT_FOUND`, „Portfolio nicht gefunden.“; bei A ist „Mein Weltportfolio“ unverändert vorhanden.<br>2. `401 UNAUTHORIZED`. `PUT` ohne Token hat keinen eigenen Test; der Endpunkt prüft das Token mit derselben Zeile wie alle geschützten Endpunkte (Code-Review). | bestanden (18.10.2026, API als Nutzer B: `PUT` und `DELETE` → `404`; Daten von A unverändert; `PortfolioControllerTest`) |

---

## US-07 Anlagen und Portfolios vergleichen (Kern-Story)

Die Testfälle sind aus dem Aktivitätsdiagramm US-07 v1 abgeleitet; die Spalte „Bezug“ nennt die durchlaufenen Zweige. Die Zuordnung aller Zweige steht im Testkonzept, Abschnitt 5.

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-07-01 | AK: genau zwei Einträge (Portfolio und Anlage); AK: Bezeichnung ohne Eingabe = Name; AK: Seed angezeigt; AK: Diagramm und Kennzahlentabelle; E29 bis E31; Aktivitätsdiagramm: Hauptpfad, Zweige „Portfolio“, „Anlage vorhanden“, „keine Bezeichnung“, „ohne Seed“, „kein Zielbetrag“ | Normalfall | manuell | Als Nutzer A die Vergleichsseite öffnen; Eintrag 1 „Mein Weltportfolio“, Eintrag 2 Anlage URTH aus der Suche; keine Bezeichnungen; Basiseingaben ohne Seed; vergleichen | **UI:** Je Eintrag ein Fächerdiagramm (Median und Bandbreite p5 bis p95) nebeneinander, beschriftet „Mein Weltportfolio“ und mit dem Namen von URTH (E35); darunter die Kennzahlen aus US-05 nebeneinander; angezeigt werden der erzeugte Seed, der gemeinsame Schätzzeitraum und die Wahrscheinlichkeit, dass Eintrag 1 am Ende mehr wert ist als Eintrag 2.<br>**API:** `POST /comparisons` → `200 OK`; `results` mit 2 Einträgen (`label`, `portfolioId` bzw. `assetId`, `timeline`, `metrics`); `parameters.seed` gesetzt; `estimates.historyFrom`/`historyTo` vorhanden; `pairwise[0].probabilityAOutperformsB` zwischen 0 und 1. | fehlgeschlagen ([B2](abnahmeprotokoll-sprint-3.md#4-gefundene-fehler-und-hinweise): Seed und gemeinsamer Schätzzeitraum werden nicht angezeigt); übrige Punkte erfüllt, zwei Diagramme nebeneinander nach E35 |
| AT-07-02 | AK: eigene Bezeichnung; AK: Seed angezeigt; E27; Aktivitätsdiagramm: Zweige „Bezeichnung angegeben“, „mit Seed“, „Zielbetrag angegeben“ | Normalfall | manuell | Eintrag 1 „Mein Weltportfolio“ mit Bezeichnung „Welt“, Eintrag 2 „Mein Aktienportfolio“ mit Bezeichnung „Aktien“; Basiseingaben, Seed 42, Zielbetrag 100.000 € | **UI:** Die Einträge heißen „Welt“ und „Aktien“; Seed 42 wird angezeigt; in der Kennzahlentabelle steht je Eintrag die Zielwahrscheinlichkeit.<br>**API:** `results[0].label` = „Welt“, `results[1].label` = „Aktien“; `metrics.targetProbability` je Eintrag zwischen 0 und 1. | offen: nicht durchführbar ([B1](abnahmeprotokoll-sprint-3.md#4-gefundene-fehler-und-hinweise), [B3](abnahmeprotokoll-sprint-3.md#4-gefundene-fehler-und-hinweise): keine Eingabe für Bezeichnung und Zielbetrag) |
| AT-07-03 | AK: gleiche Parameter und gleicher Seed → identische Ergebnisse; NF-08; E29 (gleiche Zufallszahlen) | Normalfall | manuell und automatisiert (`ComparisonServiceTest`) | 1. AT-07-02 ein zweites Mal ausführen<br>2. Eintrag 1 und Eintrag 2 beide „Mein Weltportfolio“, Seed 42<br>3. Automatisiert: gleicher Seed, Seed für beide Einträge, Paarvergleich | 1. Alle Werte identisch mit dem ersten Lauf.<br>2. Beide Einträge haben identische Kennzahlen; die Wahrscheinlichkeit „Eintrag 1 höher“ beträgt 0 % (dieselben Zufallszahlen ergeben dieselben Pfade).<br>3. `gleicherSeedLiefertGleichesErgebnis`, `ohneSeedWirdEinSeedFuerBeideEintraegeErzeugt`, `gleicheEintraegeSindNieBesser`, `paarvergleichZaehltPfadeMitHoeheremEndwert` bestanden. | offen: manueller Teil nicht durchgeführt; automatisiert bestanden (`ComparisonServiceTest`) |
| AT-07-04 | AK: ungültige Parameter wie bei US-04 (400); Aktivitätsdiagramm: Rücksprung „Eingaben ungültig“ | Alternativszenario | manuell; API-Teil automatisiert (`ComparisonControllerTest`, `SimulationParametersTest`, siehe AT-07-12) | Wie AT-07-01, aber Anlagehorizont 0; vergleichen; danach auf 20 korrigieren und erneut vergleichen | **UI:** Zuerst Meldung „Der Anlagehorizont muss eine ganze Zahl von 1 bis 40 Jahren sein.“ am Feld und „Bitte die markierten Eingaben prüfen.“, kein Ergebnis; nach der Korrektur erscheint das Ergebnis.<br>**API:** Beim ungültigen Horizont sendet das Frontend **keine** Anfrage (Prüfung vor dem Absenden); nach der Korrektur `POST /comparisons` → `200 OK`. Die Ablehnung durch das Backend (`400 VALIDATION_ERROR`, `details` nennt `horizonYears`) ist automatisiert belegt (AT-07-12).<br>*Erwartung angepasst in Sprint 5 (Docs #168): Bisher wurde `400` von der API erwartet; die Oberfläche sendet bei ungültiger Eingabe aber keine Anfrage (Abnahmeprotokoll Sprint 4, Abschnitt 6).* | offen: nicht durchgeführt (bestanden im Nachtest am 25.10.2026, siehe [Abnahmetestfälle Sprint 4](abnahmetestfaelle-sprint-4.md)) |
| AT-07-05 | Prüfregel: genau zwei Einträge, jeder mit genau einem Portfolio oder einer Anlage; Aktivitätsdiagramm: „Eingaben ungültig“ | Alternativszenario | automatisiert (`ComparisonServiceTest`, `ComparisonControllerTest`) | `items` mit 0, 1 und 3 Einträgen; Eintrag ohne ID und mit beiden IDs; `seed` 4.2 | Falsche Anzahl → `400`, „Bitte genau zwei Einträge zum Vergleich auswählen.“, `details` nennt die Anzahl; Eintrag ohne bzw. mit beiden IDs → `400`, „Jeder Eintrag braucht entweder ein Portfolio oder eine Anlage.“; Kommazahl beim Seed → `400` wie bei US-04. | bestanden (`ComparisonServiceTest`, `ComparisonControllerTest`, CI) |
| AT-07-06 | AK: fremdes Portfolio wie nicht vorhanden (404); NF-04; Aktivitätsdiagramm: „nicht vorhanden oder fremd“ | NFR | manuell (API) und automatisiert (`ComparisonServiceTest`, `ComparisonControllerTest`) | Mit dem Token von Nutzer B: `POST /comparisons` mit der `portfolioId` von „Mein Weltportfolio“ (A) und URTH | `404 NOT_FOUND`, „Portfolio nicht gefunden.“; es wird nichts berechnet. | bestanden (18.10.2026, API als Nutzer B → `404` „Portfolio nicht gefunden.“; automatisiert) |
| AT-07-07 | Aktivitätsdiagramm: „Anlage nicht vorhanden“ | Alternativszenario | automatisiert (`ComparisonServiceTest`) | Eintrag mit unbekannter `assetId` | `404 NOT_FOUND`, „Anlage nicht gefunden.“ | bestanden (`ComparisonServiceTest`, CI) |
| AT-07-08 | AK: weniger als 36 Monatswerte → alle betroffenen Anlagen genannt (422); E6, E8; Aktivitätsdiagramm: „Anlage ohne Kurse oder < 36 Monatswerte“ | Alternativszenario | manuell | Eintrag 1 „Krypto-Test“ (IBIT, ETHA), Eintrag 2 URTH; Basiseingaben | **UI:** Meldung „Für folgende Anlagen liegen weniger als 3 Jahre Kursdaten vor: …“ mit **beiden** Anlagen IBIT und ETHA; kein Ergebnis.<br>**API:** `422 INSUFFICIENT_DATA`; `details` nennt beide Anlagen. | offen: nicht durchgeführt (Verhalten bei zu kurzer Historie über AT-04-08 geprüft) |
| AT-07-09 | Docs #80: Anlage ohne Kurse beim Anbieter (422, nicht „zu kurz“); Aktivitätsdiagramm: „Anlage ohne Kurse“ | Alternativszenario | manuell | Eintrag 1 Anlage EUNL (XETR) aus der Suche, Eintrag 2 URTH; Basiseingaben | **UI:** Meldung „Für folgende Anlagen liefert der Kursdatenanbieter keine Kurse: …“ mit EUNL und dem Hinweis, die Anlage an einer anderen Börse auszuwählen.<br>**API:** `422 INSUFFICIENT_DATA`; `details` nennt EUNL mit „Keine Kursdaten beim Anbieter verfügbar.“ | offen: nicht durchgeführt |
| AT-07-10 | E29: gemeinsamer Schätzzeitraum unter 36 Monatswerten; Aktivitätsdiagramm: „< 36 Monatswerte“ beim gemeinsamen Schätzzeitraum | Alternativszenario | automatisiert (`ParameterEstimatorTest`) | Zwei Anlagen mit je mindestens 36 Monatswerten, deren Zeiträume sich weniger als 36 Monate überschneiden | `422 INSUFFICIENT_DATA`; genannt wird die Anlage mit dem spätesten Beginn (`zuKurzeUeberschneidungNenntAnlageMitSpaetestemBeginn`). Mit echten Kursdaten nicht herbeiführbar, weil alle Reihen im selben Monat enden. | bestanden (`ParameterEstimatorTest`, CI) |
| AT-07-11 | AK: Kursdaten nicht abrufbar → „Kursdaten sind gerade nicht verfügbar.“ (503); NF-07; Aktivitätsdiagramm: Zweige „im Cache“, „nicht erreichbar“ | Alternativszenario | automatisiert (`PriceHistoryServiceTest`, `SimulationControllerTest`) | Kursdaten im Cache jünger als 24 h, genau 24 h alt und älter; Anbieter nicht erreichbar | Cache jünger als 24 h → ohne Abruf verwendet; genau 24 h und älter → Abruf; Anbieter nicht erreichbar ohne gültigen Cache → `503 UPSTREAM_UNAVAILABLE` mit dieser Meldung. Vergleich und Simulation laden die Kursdaten über denselben `ParameterEstimator`; für `POST /comparisons` gibt es keinen eigenen 503-Test. | bestanden (`PriceHistoryServiceTest`, `SimulationControllerTest`, CI; kein eigener 503-Test für `/comparisons`) |
| AT-07-12 | Vorbedingung: angemeldet; Parameter-Grenzwerte wie US-04 | Grenzwert | automatisiert (`ComparisonControllerTest`, `ComparisonServiceTest`, `SimulationParametersTest`) | `POST /comparisons` ohne Token; ungültige Parameter; Grenzwerte von Horizont, Pfaden und Beträgen | Ohne Token → `401 UNAUTHORIZED`, „Bitte melde dich an.“; ungültige Parameter → `400` mit denselben Meldungen wie bei US-04 (`ungueltigeParameterLiefern400WieBeiDerSimulation`); die Grenzwerte aus AT-04-10 bis AT-04-12 gelten, weil beide Endpunkte `SimulationParameters.validate()` nutzen. | bestanden (`ComparisonControllerTest`, `ComparisonServiceTest`, `SimulationParametersTest`, CI) |
| AT-07-13 | E33: Vergleich zweier Einträge mit je 10.000 Pfaden ≤ 6 s | NFR | manuell (Messung auf Heroku) | Eintrag 1 und Eintrag 2 „Messportfolio“ (5 Anlagen), Anlagehorizont 40 Jahre, 10.000 Pfade; ein Lauf zum Füllen des Cache, danach 20 Läufe; `runtimeMs` je Lauf notieren | Mindestens 19 von 20 Werten (95 %) ≤ 6.000 ms, gemessen wie NF-01. | offen: nicht gemessen |

---

## US-08 Simulation speichern und aufrufen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-08-01 | AK: unter einem Namen speichern; AK: Liste mit Name, Portfolio, Speicherzeitpunkt, neueste zuerst; E32 | Normalfall | manuell | Als Nutzer A „Mein Weltportfolio“ mit den Basiseingaben und Seed 42 simulieren; Kennzahlen notieren; als „Sparplan 20 Jahre“ speichern. Danach Anlagehorizont 10 Jahre simulieren und als „Kurz 10 Jahre“ speichern | **UI:** Nach dem Speichern erscheint „Simulation als „Sparplan 20 Jahre“ gespeichert.“; die Liste zeigt „Kurz 10 Jahre“ vor „Sparplan 20 Jahre“, jeweils mit „Mein Weltportfolio“ und Datum mit Uhrzeit.<br>**API:** `POST /simulations` mit `save: true` → `201 Created` mit `id`, `name` und `createdAt`; `parameters.seed` gleich dem Seed der angezeigten Simulation; `POST /simulations/list` liefert beide, neueste zuerst. | bestanden (18.10.2026: „Kurz 10 Jahre“ vor „Sparplan 20 Jahre“, je mit Portfolio und Uhrzeit) |
| AT-08-02 | AK: gespeicherte Simulation zeigt dieselben Werte ohne Neuberechnung, auch nach Änderung des Portfolios | Normalfall | manuell | Nach AT-08-01 „Mein Weltportfolio“ auf URTH 50 % und IEMG 50 % ändern (US-06); dann „Sparplan 20 Jahre“ öffnen | **UI:** Die Eingabefelder zeigen die gespeicherten Parameter (20 Jahre, Seed 42); Fächerdiagramm und Kennzahlen sind identisch mit den notierten Werten aus AT-08-01, obwohl das Portfolio inzwischen anders gewichtet ist; ein Hinweis nennt Name und Speicherzeitpunkt.<br>**API:** `POST /simulations/{id}/details` → `200 OK` mit denselben `estimates`, `timeline`, `metrics` und `runtimeMs` wie beim Speichern. Anschließend „Mein Weltportfolio“ wieder auf 70 % / 30 % setzen. | bestanden (18.10.2026: nach Änderung auf 50/50 identisches Ergebnis, API-Vergleich `true`, `runtimeMs` 359) |
| AT-08-03 | AK: gespeicherte Simulation löschen (204) | Normalfall | manuell | „Kurz 10 Jahre“ löschen, Bestätigung zuerst abbrechen, dann bestätigen | Nach dem Abbrechen bleibt der Eintrag; nach dem Bestätigen verschwindet er aus der Liste.<br>**API:** `DELETE /simulations/{id}` → `204 No Content`; danach `details` → `404`. | bestanden (18.10.2026) |
| AT-08-04 | AK: ohne Namen wird nicht gespeichert, Meldung nennt das Feld (400) | Alternativszenario | manuell | Simulation berechnen; Name leer bzw. nur Leerzeichen; speichern | **UI:** Meldung „Bitte einen Namen mit höchstens 100 Zeichen eingeben.“ am Namensfeld; die Liste bleibt unverändert.<br>**API:** Die Oberfläche sendet keine Anfrage; direkt per HTTP-Client `save: true` ohne `name` → `400 VALIDATION_ERROR`, `details` nennt `name`; es wird nichts gespeichert. | bestanden (18.10.2026; API ohne `name` → `400`, nichts gespeichert) |
| AT-08-05 | E32: Name höchstens 100 Zeichen | Grenzwert | automatisiert (`SimulationControllerTest`, Frontend-Test `saved-simulations.test.js`) | Name mit 100 und 101 Zeichen, nur Leerzeichen, mit Leerzeichen am Rand | 100 Zeichen → gespeichert (Name getrimmt); 101 Zeichen und nur Leerzeichen → `400` mit der Meldung aus AT-08-04, ohne zu rechnen. | bestanden (`SimulationControllerTest`, `saved-simulations.test.js`, CI) |
| AT-08-06 | AK: fremde oder nicht vorhandene Simulation → 404; nur eigene Einträge in der Liste; NF-04; Vorbedingung: angemeldet | NFR | manuell (API) und automatisiert (`SimulationControllerTest`, `SavedSimulationServiceTest`) | 1. Als Nutzer B die Liste der gespeicherten Simulationen öffnen<br>2. Mit dem Token von B: `details` und `DELETE` auf „Sparplan 20 Jahre“ von A<br>3. Automatisiert zusätzlich: `list` ohne Token | 1. Die Liste von B ist leer („Noch keine Simulation gespeichert.“).<br>2. Beide → `404 NOT_FOUND`, „Simulation nicht gefunden.“; bei A ist die Simulation unverändert vorhanden.<br>3. `401 UNAUTHORIZED`. | bestanden (18.10.2026, Nutzer B: Liste leer, `details` und `DELETE` → `404`; automatisiert) |
| AT-08-07 | AK: ohne Neuberechnung; nur mit `save: true` wird gespeichert | Alternativszenario | automatisiert (`SimulationControllerTest`, `SavedSimulationServiceTest`) | Simulation ohne `save`; Abruf einer gespeicherten Simulation | Ohne `save` wird nichts gespeichert (`simulateWithoutSave_storesNothing`); `details` gibt das gespeicherte Ergebnis zurück, ohne den `SimulationService` aufzurufen (`details_returnsStoredResultWithoutCalculating`, `readResult_returnsExactlyTheStoredResult`). | bestanden (`SimulationControllerTest`, `SavedSimulationServiceTest`, CI) |

Vergleiche speichern ist nach Sprint 4 verschoben (E34) und hat in Sprint 3 keinen Testfall.

---

## Nicht-funktionale Anforderungen

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-NF09-01 | NF-09: Nutzer ohne Vorkenntnisse legen ohne Anleitung ein Portfolio an und simulieren es | NFR | manuell (Bedienbarkeitstest, Ablauf im Testkonzept, Abschnitt 10) | 3 fachfremde Testpersonen erhalten je ein angemeldetes Testkonto und die Aufgabe: „Lege ein Portfolio aus zwei Anlagen an und starte eine Simulation.“ Keine Hilfe; Zeit ab dem ersten Klick | Mindestens 2 von 3 Testpersonen haben nach höchstens 5 Minuten ohne Hilfe ein gespeichertes Portfolio und ein Simulationsergebnis. Zeiten und Stellen, an denen gezögert wurde, werden notiert. | offen: Bedienbarkeitstest noch nicht durchgeführt |
| AT-NF10-01 | NF-10: neue und geänderte Seiten ab 360 px ohne horizontales Scrollen | NFR | manuell | Bei 360 px Breite (Entwicklertools) prüfen: Simulationsseite mit Kennzahlen, Speichern und Liste gespeicherter Simulationen; Portfolio-Editor mit Bearbeiten und Löschen; Vergleichsseite mit Ergebnis | Alle Bereiche sind ohne horizontales Scrollen bedienbar; Kennzahlen und Listeneinträge brechen sinnvoll um. | offen: nicht durchgeführt |

---

## Nachtest der Befunde aus Sprint 2

Nach der Behebung von F1 bis F7 werden alle in Sprint 2 nicht bestandenen Testfälle erneut durchgeführt (Testkonzept, Abschnitt 11, Punkt 5). Dabei gilt die rechte Spalte:

| Testfall | Befund | Bisher | Beim Nachtest | Grund | Ergebnis 18.10.2026 |
|---|---|---|---|---|---|
| AT-02-03, AT-03-08 | F1 | – | unverändert | Suchtreffer in den Editor übernehmen (Docs #78) | bestanden |
| AT-03-01, AT-03-07, AT-03-10 | F2 | „Mein Weltportfolio“ aus EUNL 70 %, IS3N 30 % | „Mein Weltportfolio“ aus URTH 70 %, IEMG 30 % | F2 (FE #21); Basisportfolio auf US-Börsen (PO-Entscheidung vom 18.10.2026) | bestanden |
| AT-04-01 bis AT-04-04, AT-04-15 | Fächerdiagramm fehlte (FE #3) | Basisportfolio mit EUNL | Basisportfolio mit URTH und IEMG | FE #3 umgesetzt; Basisportfolio | bestanden (AT-04-15 nach Ergänzung der Hinweise um 21:44) |
| AT-04-08 | F3 | Testanlagen mit kurzer Historie über den Mock | „Krypto-Test“ (IBIT, ETHA) als Portfolio | Kein `MockMarketDataProvider` vorhanden; echte Anlagen mit weniger als 36 Monatswerten | bestanden (nur ETHA zu kurz, siehe Testdaten; Nennung aller betroffenen Anlagen automatisiert in `ParameterEstimatorTest`) |
| AT-04-13 | F4 | – | Erster Lauf (frische Kurse) und zweiter Lauf (Cache) mit Seed 42 liefern identische Werte; automatisiert `PriceHistoryServiceTest.frischeUndGecachteKurseSindGleich` | F4 (BE #73) | bestanden |
| AT-04-14 | NF-01 nicht gemessen | Messfall mit Testanlagen des Mocks | „Messportfolio“ (5 Anlagen), 40 Jahre, 10.000 Pfade, 20 Läufe auf Heroku | Kein Mock; Messung wie AT-07-13 | bestanden (20 von 20 ≤ 3.000 ms, Maximum 1.602 ms) |
| AT-02-08 | F5 | – | unverändert (20 Suchbegriffe aus dem Cache, mindestens 19 Antworten ≤ 1 s); automatisiert `SearchCacheTest` | F5 (BE #77) | bestanden (20 von 20 ≤ 1.000 ms, Maximum 58 ms) |
| AT-02-07, AT-04-06 | kein Ausfall-Schalter | manuell geplant | nur automatisiert (`AssetControllerTest`, `SimulationControllerTest.anbieterAusfallLiefert503`) | Kein `MockMarketDataProvider` mit Ausfall-Schalter | bestanden (automatisiert) |
| – | F6 | – | Review: Klassen `UserManager` und `PropertyFileUserManagerImpl` entfernt, `V5__kursdaten_geladen.sql` unverändert | F6 (BE #67) | bestanden (Review `main`) |
| – | F7 | – | Frontend-CI führt `npm ci` und `npm test` aus; ein absichtlich fehlschlagender Test macht den Lauf rot | F7 (FE #16) | bestanden (Workflow führt `npm ci` und `npm test` aus, CI auf `main` grün) |

---

## Automatisierte Tests

| Testklasse | Repository | Teststufe | Abgedeckte Testfälle | Stand 18.10.2026 |
|---|---|---|---|---|
| `MetricsCalculatorTest` | BE | Unit-Test | AT-05-05, AT-05-06 | vorhanden (BE #71) |
| `SimulationParametersTest` | BE | Unit-Test | AT-05-04, AT-07-12 | vorhanden, um Zielbetrag ergänzt (BE #71) |
| `PortfolioServiceTest` | BE | Unit-Test mit Mockito | AT-06-03, AT-06-06 | vorhanden, um Ändern und Löschen ergänzt (BE #70) |
| `PortfolioControllerTest` | BE | Integrationstest (`MockMvc`) | AT-06-03, AT-06-06 | vorhanden, um `PUT` und `DELETE` ergänzt (BE #70) |
| `ComparisonServiceTest` | BE | Unit-Test mit Mockito | AT-07-03, AT-07-05 bis AT-07-07, AT-07-12 | vorhanden (BE #69) |
| `ComparisonControllerTest` | BE | Integrationstest (`MockMvc`) | AT-07-05, AT-07-06, AT-07-12 | vorhanden (BE #69) |
| `ParameterEstimatorTest` | BE | Unit-Test | AT-07-10 | vorhanden (BE #39, Docs #80) |
| `PriceHistoryServiceTest` | BE | Unit-Test mit vorgegebener Uhr | AT-07-11; Nachtest AT-04-13 | vorhanden (F4) |
| `SimulationControllerTest` | BE | Integrationstest (`MockMvc`) | AT-07-11, AT-08-05 bis AT-08-07 | vorhanden, um Speichern, Liste, Abruf und Löschen ergänzt (BE #68) |
| `SavedSimulationServiceTest` | BE | Unit-Test mit Mockito | AT-08-06, AT-08-07 | vorhanden (BE #68) |
| `saved-simulations.test.js` | FE | Unit-Test (`node:test`) | AT-08-05 | neu (FE #17) |

Für das Löschen eines Portfolios mit gespeicherten Simulationen (AT-06-05) gibt es keinen automatisierten Test, weil die Löschweitergabe in der Datenbank liegt (Migration V7, `ON DELETE CASCADE`); er wird manuell geprüft.
