# Simulationsmodell – Rechenregeln der Monte-Carlo-Simulation (US-04)

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #72 PO-Festlegungen zum Simulationsmodell (Version 1); Docs #58 Anpassung an die umgesetzte Engine (Version 2) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 2 |
| **Stand** | 11.10.2026 |
| **Grundlagen** | Festlegungen E15 bis E23 (`kurzformat_sprint-2.md`, Version 3), Kurzformat Sprint 1 (US-04, Festlegungen 3 bis 6, E6, E8), Domänenmodell v3, Glossar, API-Design (Stand 02.10.2026, Abschnitt „Simulationen“), ADR-02, ADR-03, NF-01, NF-07, NF-08; Backend-Code `MonteCarloEngine` (BE #40) und `MetricsCalculator` (BE #49), Stand main 11.10.2026 |
| **Betrifft** | BE #39 Kursdaten laden und schätzen, BE #40 Monte-Carlo-Engine, BE #41 Unit-Tests Simulationslogik, BE #49 Perzentile, BE #50 SimulationService, FE #3 Fächerdiagramm |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Dieses Dokument beschreibt Schritt für Schritt, wie LookUpStock aus der Kurshistorie die möglichen Wertverläufe eines Portfolios berechnet. Die fachlichen Entscheidungen dahinter sind die PO-Festlegungen E15 bis E23 in `kurzformat_sprint-2.md`.

**Version 2:** Die Monte-Carlo-Engine (BE #40) und die Perzentilberechnung (BE #49) wurden anders umgesetzt als in Version 1 beschrieben. Der Product Owner hat am 11.10.2026 entschieden, das Modell an die Umsetzung anzupassen (E18 und E23 geändert). Abschnitt 3 bis 6 beschreiben deshalb jetzt die umgesetzte Rechnung; für den noch nicht umgesetzten `ParameterEstimator` (BE #39) legt Abschnitt 3 fest, welche Werte er liefern muss, damit sie zur Engine passen. Die Unterschiede zu Version 1 stehen in der Änderungshistorie am Ende.

Schreibweise: `√` ist die Quadratwurzel, `max(a, b)` der größere der beiden Werte.

---

## 1 Eingaben

| Eingabe | Regel | Quelle |
|---|---|---|
| Portfolio | eigenes, gespeichertes Portfolio des angemeldeten Nutzers mit Gewichtungen `w_i` als Anteile (Summe 1), aus `Portfolio.getWeights()` | US-03, NF-04, ADR-03, ADR-04 |
| Startkapital `S` | in Euro, ≥ 0 | Festlegung 4 |
| Sparrate `C` | in Euro je Monat, ≥ 0; `S` und `C` nicht beide 0 | Festlegung 4 |
| Anlagehorizont `H` | ganze Zahl von 1 bis 40 Jahren; Anzahl Monate `M = 12 × H` | Festlegung 3 |
| Pfadanzahl `N` | optional, 1.000 bis 50.000, Standard 10.000 | Festlegung 5 |
| Seed | optional, ganze Zahl; ohne Angabe erzeugt das System eine zufällige ganze Zahl und gibt sie in `parameters.seed` zurück | Festlegung 6 |

Die Eingaben werden von `SimulationParameters.validate()` geprüft (Information Expert, ADR-03); fehlt die Pfadanzahl, setzt `SimulationParameters` 10.000 ein. Den fehlenden Seed setzt der `SimulationService` über `withSeed(…)`.

## 2 Kurshistorie beschaffen (BE #39, noch nicht umgesetzt)

1. Je Anlage die **Monatswerte** abrufen: bei Twelve Data `time_series` mit `interval=1month`, die Anlage über Symbol und Börsenkennung (`mic_code`) bestimmt. Die Kurse sind um **Splits und Dividenden bereinigt** (Gesamtrendite, E20; bei Twelve Data `adjust=all`).
2. Den **laufenden, noch nicht abgeschlossenen Monat verwerfen** (E22).
3. Liegen für eine Anlage **weniger als 36 Monatswerte** vor, wird die Simulation nicht gerechnet: `422 INSUFFICIENT_DATA`, die Meldung nennt **alle** betroffenen Anlagen (E6, E8).
4. **Gemeinsamer Schätzzeitraum** (E17): die letzten Monate, für die alle Anlagen einen Monatswert haben, höchstens 120 Monatswerte. Fehlt einer Anlage ein Monat mittendrin, wird dieser Monat für alle Anlagen ausgelassen.
5. `historyFrom` = erster Tag des ersten verwendeten Monats, `historyTo` = letzter Tag des letzten verwendeten Monats (Beispiel API-Design: `2016-09-01` bis `2026-08-31`).
6. Herkunft der Daten: Cache, wenn jünger als 24 Stunden, sonst Twelve Data; ist Twelve Data nicht erreichbar und kein ausreichend junger Cache vorhanden: `503 UPSTREAM_UNAVAILABLE` (NF-07).

Aus `K` Monatswerten im Schätzzeitraum entstehen `K − 1` Monatsrenditen (bei 120 Monatswerten also 119).

## 3 Schätzung je Anlage (ParameterEstimator, BE #39)

Der `ParameterEstimator` liefert ein `EstimationResult` mit `historyFrom`, `historyTo` und je Anlage ein `Estimate(asset, annualReturn, annualVolatility)`. Diese Werte werden in der API angezeigt (`estimates.assets`) **und** von der Engine verwendet (E18).

Für jede Anlage `i` und jeden Monat `t` im Schätzzeitraum:

```
r_i,t            = Kurs_i,t / Kurs_i,t−1 − 1                      (Monatsrendite)
m_i              = Mittelwert der r_i,t
s_i              = Standardabweichung der r_i,t (Stichprobe, Division durch K − 2)

annualReturn_i     = 12 · m_i
annualVolatility_i = s_i · √12
```

Die Hochrechnung auf Jahreswerte ist genau die Umkehrung der Umrechnung in der Engine (Abschnitt 4). Dadurch rechnet die Engine wieder mit `m_i` und `s_i`, also mit den tatsächlich beobachteten Monatswerten. Renditen werden in der Handelswährung der Anlage verwendet und nicht in Euro umgerechnet (E16).

## 4 Simulation (MonteCarloEngine, BE #40, umgesetzt)

Die Engine erhält die Anteile `w_i` (aus `Portfolio.getWeights()`), das `EstimationResult` und die geprüften `SimulationParameters` mit gesetztem Seed. Je Anlage rechnet sie die Jahreswerte in Monatswerte um:

```
μ_i = annualReturn_i / 12
σ_i = annualVolatility_i / √12
```

Für jeden Pfad `p = 1 … N`:

```
V_p,0 = S

für m = 1 … M:
    R_p,m = w_1 · (μ_1 + σ_1 · Z_p,m,1) + … + w_n · (μ_n + σ_n · Z_p,m,n)
    V_p,m = max(0, V_p,m−1 · (1 + R_p,m)) + C
```

- `Z_p,m,i` ist eine standardnormalverteilte Zufallszahl (Mittelwert 0, Standardabweichung 1), **für jede Anlage einzeln** gezogen. Die Anlagen schwanken damit unabhängig voneinander; ein Zusammenhang zwischen den Anlagen (Korrelation) wird nicht abgebildet (E18).
- Die Monatsrenditen sind normalverteilt (E23). Fällt der Wert rechnerisch unter 0, wird er auf 0 gesetzt, bevor die Sparrate eingezahlt wird.
- Die Gewichtungen gelten in jedem Monat neu, das Portfolio wird also monatlich auf die festgelegten Anteile zurückgesetzt (Rebalancing, E15). Die Sparrate fließt in den Portfoliowert und wird damit nach denselben Anteilen angelegt.
- Die Sparrate `C` wird **am Monatsende nach** der Wertentwicklung des Monats eingezahlt, erstmals in Monat 1 (E19).
- Keine Kosten, Steuern oder Inflation (E21).
- Ergebnis der Engine: eine Tabelle `double[N][M + 1]` mit dem Portfoliowert je Pfad und Monat; Spalte 0 ist das Startkapital.

Summe der Einzahlungen: `S + M · C` (Beispiel API-Design: 10.000 + 240 · 200 = 58.000).

## 5 Ergebnis (MetricsCalculator, BE #49, umgesetzt)

1. **Verlaufspunkte** für jeden Monat `m = 0 … M` (also `M + 1` Punkte): `p5`, `p50`, `p95` der Werte `V_1,m … V_N,m`.
2. **Perzentil mit linearer Interpolation** (wie `QUANTIL.INKL` in Excel): Werte aufsteigend sortieren, Position `q · (N − 1)` bestimmen (gezählt ab 0); liegt sie zwischen zwei Werten, wird zwischen diesen linear interpoliert. Beispiel `N = 10.000`, `q = 0,05`: Position 499,95, also 95 % des Wertes an Position 500 plus 5 % des Wertes an Position 499. So ist jedes Ergebnis in Excel nachrechenbar.
3. Monat 0: alle drei Perzentile sind gleich `S`.
4. Rundung **erst bei der Ausgabe** (SimulationService bzw. API): Eurobeträge auf 2 Nachkommastellen, Anteile (`annualReturn`, `annualVolatility`) auf 4 Nachkommastellen. Intern wird ungerundet gerechnet.
5. Unter dem Ergebnis zeigt das Frontend die Hinweise „Wechselkursschwankungen werden nicht berücksichtigt.“ (E16) und „Die Werte sind nominal und berücksichtigen keine Kosten, Steuern und Inflation. Keine Anlageberatung.“ (E21, NF-14).

Die Kennzahlen aus US-05 (`metrics`, z. B. Value at Risk, Drawdown) werden mit US-05 in Sprint 3 festgelegt.

## 6 Umsetzung (Stand Code)

| Thema | Umsetzung | Grund |
|---|---|---|
| Zufallszahlen | ein Generator `java.util.Random(seed)` je Simulation, Zufallszahlen über `nextGaussian()` | gleicher Seed = gleiches Ergebnis (NF-08) |
| Reihenfolge der Ziehung | Pfad für Pfad; innerhalb eines Pfades Monat für Monat; innerhalb eines Monats Anlage für Anlage in der Reihenfolge von `getWeights()` (aufsteigende Asset-ID, ADR-03) | feste Reihenfolge für NF-08 |
| Vergleich (US-07) | jedes Portfolio des Vergleichs wird mit einem neuen `Random(seed)` desselben Seeds gerechnet | alle Varianten erhalten dieselben Zufallszahlen (Glossar „Vergleich“) |
| Rechengenauigkeit | Simulation mit `double`; Anteile aus `getWeights()` (aus `BigDecimal` umgerechnet) | Geschwindigkeit (NF-01) |
| Speicher | die Engine hält alle Werte `N × (M + 1)` im Speicher; im ungünstigsten Fall (50.000 Pfade, 40 Jahre) sind das rund 190 MB | siehe Abschnitt 7 |
| Laufzeit | 10.000 Pfade, 5 Anlagen, 40 Jahre in höchstens 3 s, gemessen über `runtimeMs` | NF-01 |

## 7 Grenzen des Modells

| Grenze | Auswirkung | Umgang |
|---|---|---|
| Keine Korrelation (E18) | Bewegen sich Anlagen meist in dieselbe Richtung (z. B. zwei Aktien-ETFs), ist das Fächerdiagramm schmaler, als es sein müsste; bei gegenläufigen Anlagen breiter. Einzelanlagen sind nicht betroffen. | bewusste Vereinfachung; in der Präsentation und im Sprint Review offen benennen. Kann bei Bedarf über das Product Backlog nachgerüstet werden. |
| Normalverteilte Renditen mit Untergrenze 0 (E23) | Bei sehr hoher Volatilität entstehen Pfade, die auf 0 fallen. Für realistische Volatilitäten (unter ca. 40 % pro Jahr) ist das praktisch ausgeschlossen. | Untergrenze verhindert negative Werte; Test `simulate_extremeVolatility_valuesNeverNegative` |
| Speicherbedarf | Bei 50.000 Pfaden und 40 Jahren rund 190 MB je Simulation; auf Heroku stehen je nach Tarif 512 MB für die ganze Anwendung zur Verfügung. | beim Deployment (Docs #64) und bei der Messung von NF-01 prüfen; bei Problemen die Perzentile Monat für Monat berechnen, statt alle Werte zu speichern |

## 8 Prüfbare Beispiele für Unit-Tests (BE #41)

Die Beispiele S1 bis S7 gelten für die umgesetzte Engine; S8 bis S10 für den `ParameterEstimator` (BE #39).

| Nr. | Fall | Eingabe | Erwartung |
|---|---|---|---|
| S1 | Ohne Schwankung | eine Anlage, `annualReturn = 0,12`, `annualVolatility = 0`, `S = 10.000`, `C = 200`, `H = 1` | Alle Pfade und alle Perzentile in Monat 12 = **13.804,75 €** (1 % je Monat) |
| S2 | Monat 0 | beliebige Eingaben, `S = 10.000` | `p5 = p50 = p95 = 10.000` in Monat 0 |
| S3 | Anzahl Punkte | `H = 20` | `timeline` hat 241 Einträge (Monat 0 bis 240) |
| S4 | Reproduzierbarkeit | zweimal gleiche Eingaben, `seed = 42` | identische `timeline` (NF-08) |
| S5 | Unterschiedlicher Seed | `seed = 42` und `seed = 43` | Pfade unterscheiden sich |
| S6 | Einzahlungen | `S = 10.000`, `C = 200`, `H = 20` | Summe der Einzahlungen 58.000 € |
| S7 | Rebalancing | zwei Anlagen ohne Schwankung, `annualReturn` 0,12 (70 %) und 0 (30 %), `S = 10.000`, `C = 0` | Wert nach Monat 1 = **10.070,00 €** (0,7 · 1 % + 0,3 · 0 %) |
| S8 | Zu kurze Historie | eine Anlage mit 35 Monatswerten, eine mit 30 | `422 INSUFFICIENT_DATA`, beide Anlagen genannt (E6, E8) |
| S9 | Schätzzeitraum | Anlage A mit 200, Anlage B mit 60 Monatswerten | gemeinsamer Schätzzeitraum = 60 Monatswerte (59 Renditen) (E17) |
| S10 | Laufender Monat | Daten enthalten den laufenden Monat | dieser Monat wird nicht verwendet (E22) |
| S11 | Hochrechnung | Monatsrenditen einer Anlage mit Mittelwert 0,5 % und Standardabweichung 2 % | `annualReturn = 0,06`, `annualVolatility = 0,0693` (gerundet) |

S1, S3 bis S5 und S7 entsprechen den vorhandenen Tests in `MonteCarloEngineTest` (BE #40, BE #41).

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 08.10.2026 | Neu: Rechenregeln der Simulation auf Grundlage von E15 bis E23 | Rückfrage aus BE #40 (Monte-Carlo-Engine), Docs #72 |
| 2 | 11.10.2026 | An die umgesetzte Engine angepasst: Schätzung und Simulation je Anlage statt über die Portfoliorendite (E18), normalverteilte Monatsrenditen mit Untergrenze 0 statt lognormal (E23), Perzentile mit linearer Interpolation statt Nearest-Rank, Ziehreihenfolge Pfad für Pfad. Neu: Abschnitt „Grenzen des Modells“, Hochrechnung der Jahreswerte für den ParameterEstimator, Beispiele S7 und S11. Entfallen: Beispiel S7 alt (Korrelation) | PO-Entscheidung vom 11.10.2026 nach Abgleich mit BE #40 und BE #49 |
