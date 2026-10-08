# Simulationsmodell – Rechenregeln der Monte-Carlo-Simulation (US-04)

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #72 PO-Festlegungen zum Simulationsmodell |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 1 |
| **Stand** | 08.10.2026 |
| **Grundlagen** | Festlegungen E15 bis E23 (`kurzformat_sprint-2.md`), Kurzformat Sprint 1 (US-04, Festlegungen 3 bis 6, E6, E8), Domänenmodell v3, Glossar, API-Design (Stand 02.10.2026, Abschnitt „Simulationen“), ADR-02, ADR-03, NF-01, NF-07, NF-08 |
| **Betrifft** | BE #39 Kursdaten laden und schätzen, BE #40 Monte-Carlo-Engine, BE #41 Unit-Tests Simulationslogik, BE #49 Perzentile, BE #50 SimulationService, FE #3 Fächerdiagramm |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Dieses Dokument beschreibt Schritt für Schritt, wie LookUpStock aus der Kurshistorie die möglichen Wertverläufe eines Portfolios berechnet. Die fachlichen Entscheidungen dahinter sind die PO-Festlegungen E15 bis E23 in `kurzformat_sprint-2.md`. Abschnitt 6 enthält zusätzlich Vorgaben für die Umsetzung, die mit dem Entwickler abgestimmt sind; sie sorgen dafür, dass Ergebnisse reproduzierbar (NF-08) und Tests eindeutig sind.

Schreibweise: `ln` ist der natürliche Logarithmus, `exp` die Exponentialfunktion, `√` die Quadratwurzel.

---

## 1 Eingaben

| Eingabe | Regel | Quelle |
|---|---|---|
| Portfolio | eigenes, gespeichertes Portfolio des angemeldeten Nutzers mit Gewichtungen `w_i` als Anteile (Summe 1), aus `Portfolio.getWeights()` | US-03, NF-04, ADR-03 |
| Startkapital `S` | in Euro, ≥ 0 | Festlegung 4 |
| Sparrate `C` | in Euro je Monat, ≥ 0; `S` und `C` nicht beide 0 | Festlegung 4 |
| Anlagehorizont `H` | ganze Zahl von 1 bis 40 Jahren; Anzahl Monate `M = 12 × H` | Festlegung 3 |
| Pfadanzahl `N` | optional, 1.000 bis 50.000, Standard 10.000 | Festlegung 5 |
| Seed | optional, ganze Zahl; ohne Angabe erzeugt das System eine zufällige ganze Zahl und gibt sie in `parameters.seed` zurück | Festlegung 6 |

## 2 Kurshistorie beschaffen (BE #39)

1. Je Anlage die **Monatswerte** abrufen: bei Twelve Data `time_series` mit `interval=1month`, die Anlage über Symbol und Börsenkennung (`mic_code`) bestimmt. Die Kurse sind um **Splits und Dividenden bereinigt** (Gesamtrendite, E20; bei Twelve Data `adjust=all`).
2. Den **laufenden, noch nicht abgeschlossenen Monat verwerfen** (E22).
3. Liegen für eine Anlage **weniger als 36 Monatswerte** vor, wird die Simulation nicht gerechnet: `422 INSUFFICIENT_DATA`, die Meldung nennt **alle** betroffenen Anlagen (E6, E8).
4. **Gemeinsamer Schätzzeitraum** (E17): die letzten Monate, für die alle Anlagen einen Monatswert haben, höchstens 120 Monatswerte. Fehlt einer Anlage ein Monat mittendrin, wird dieser Monat für alle Anlagen ausgelassen.
5. `historyFrom` = erster Tag des ersten verwendeten Monats, `historyTo` = letzter Tag des letzten verwendeten Monats (Beispiel API-Design: `2016-09-01` bis `2026-08-31`).
6. Herkunft der Daten: Cache, wenn jünger als 24 Stunden, sonst Twelve Data; ist Twelve Data nicht erreichbar und kein ausreichend junger Cache vorhanden: `503 UPSTREAM_UNAVAILABLE` (NF-07).

Aus `K` Monatswerten im Schätzzeitraum entstehen `K − 1` Monatsrenditen (bei 120 Monatswerten also 119).

## 3 Schätzung (ParameterEstimator)

### 3.1 Monatsrenditen der Anlagen

Für jede Anlage `i` und jeden Monat `t` im Schätzzeitraum:

```
r_i,t = Kurs_i,t / Kurs_i,t−1 − 1
```

### 3.2 Monatsrendite des Portfolios (E15, E18)

Weil die Gewichtungen jeden Monat zurückgesetzt werden (E15), ist die Monatsrendite des Portfolios die gewichtete Summe der Anlagerenditen desselben Monats. Dadurch ist der Zusammenhang zwischen den Anlagen (Korrelation) automatisch enthalten (E18):

```
R_t = w_1 · r_1,t + w_2 · r_2,t + … + w_n · r_n,t
g_t = ln(1 + R_t)                    (logarithmierte Monatsrendite)
```

### 3.3 Parameter der Simulation

Aus den `K − 1` Werten `g_t`:

```
μ = Mittelwert der g_t
σ = Standardabweichung der g_t (Stichprobe, Division durch K − 2)
```

`μ` und `σ` sind die einzigen Parameter, die die Engine braucht. Renditen werden in der Handelswährung der Anlage verwendet und nicht in Euro umgerechnet (E16).

### 3.4 Schätzung je Anlage (nur zur Anzeige)

Die API liefert in `estimates.assets` für jede Anlage Rendite und Volatilität als Jahreswerte. Sie dienen der Anzeige und werden aus denselben Monaten berechnet; die Simulation rechnet mit `μ` und `σ` aus 3.3:

```
g_i,t            = ln(1 + r_i,t)
annualReturn     = exp(12 · Mittelwert der g_i,t) − 1
annualVolatility = Standardabweichung der g_i,t · √12
```

## 4 Simulation (MonteCarloEngine)

Für jeden Pfad `p = 1 … N`:

```
V_p,0 = S
V_p,m = V_p,m−1 · exp(μ + σ · Z_p,m) + C      für m = 1 … M
```

- `Z_p,m` ist eine standardnormalverteilte Zufallszahl (Mittelwert 0, Standardabweichung 1). Daraus folgen lognormalverteilte Monatsrenditen; ein Wert kann nicht negativ werden (E23).
- Die Sparrate `C` wird **am Monatsende nach** der Wertentwicklung des Monats eingezahlt, erstmals in Monat 1 (E19). Sie wird nach den Gewichtungen aufgeteilt; durch E15 ist das in der Formel bereits enthalten.
- Keine Kosten, Steuern oder Inflation (E21).

Summe der Einzahlungen: `S + M · C` (Beispiel API-Design: 10.000 + 240 · 200 = 58.000).

## 5 Ergebnis (MetricsCalculator)

1. **Verlaufspunkte** für jeden Monat `m = 0 … M` (also `M + 1` Punkte): `p5`, `p50`, `p95` der Werte `V_1,m … V_N,m`.
2. **Perzentil** nach dem Nearest-Rank-Verfahren: Werte aufsteigend sortieren; das `q`-te Perzentil ist der Wert an Position `⌈q/100 · N⌉` (gezählt ab 1). Bei `N = 10.000`: `p5` = 500. Wert, `p50` = 5.000. Wert, `p95` = 9.500. Wert.
3. Monat 0: alle drei Perzentile sind gleich `S`.
4. Rundung **erst bei der Ausgabe**: Eurobeträge auf 2 Nachkommastellen, Anteile (`annualReturn`, `annualVolatility`) auf 4 Nachkommastellen. Intern wird ungerundet gerechnet.
5. Unter dem Ergebnis zeigt das Frontend die Hinweise „Wechselkursschwankungen werden nicht berücksichtigt.“ (E16) und „Die Werte sind nominal und berücksichtigen keine Kosten, Steuern und Inflation. Keine Anlageberatung.“ (E21, NF-14).

Die Kennzahlen aus US-05 (`metrics`, z. B. Value at Risk, Drawdown) werden mit US-05 in Sprint 3 festgelegt.

## 6 Vorgaben für die Umsetzung (mit dem Entwickler abgestimmt)

| Thema | Vorgabe | Grund |
|---|---|---|
| Zufallszahlen | ein Generator `java.util.Random(seed)`, Zufallszahlen über `nextGaussian()` | gleicher Seed = gleiches Ergebnis (NF-08) |
| Reihenfolge der Ziehung | Monat für Monat: erst alle Pfade für Monat 1, dann alle für Monat 2 usw. | feste Reihenfolge für NF-08; je Monat müssen nur `N` Werte im Speicher liegen, aus denen sofort die Perzentile berechnet werden (wichtig bei 50.000 Pfaden) |
| Vergleich (US-07) | jedes Portfolio des Vergleichs wird mit einem neuen `Random(seed)` desselben Seeds gerechnet | alle Varianten erhalten dieselben Zufallszahlen (Glossar „Vergleich“) |
| Rechengenauigkeit | Simulation mit `double`; Gewichtungen kommen aus `getWeights()` (aus `BigDecimal` umgerechnet) | Geschwindigkeit (NF-01) |
| Laufzeit | 10.000 Pfade, 5 Anlagen, 40 Jahre in höchstens 3 s, gemessen über `runtimeMs` | NF-01 |

**Auswirkung auf das Klassendiagramm:** Die `MonteCarloEngine` erhält `μ`, `σ` und die Simulationsparameter statt der Schätzungen je Anlage. Der `ParameterEstimator` nutzt `Portfolio.getWeights()` (ADR-03), um die Portfoliorendite zu bilden. Das wird im Klassendiagramm v2 (Docs #58) übernommen.

## 7 Prüfbare Beispiele für Unit-Tests (BE #41)

| Nr. | Fall | Eingabe | Erwartung |
|---|---|---|---|
| S1 | Ohne Schwankung | `μ = ln(1,01)`, `σ = 0`, `S = 10.000`, `C = 200`, `H = 1` | Alle Pfade und alle Perzentile in Monat 12 = **13.804,75 €** |
| S2 | Monat 0 | beliebige Eingaben, `S = 10.000` | `p5 = p50 = p95 = 10.000` in Monat 0 |
| S3 | Anzahl Punkte | `H = 20` | `timeline` hat 241 Einträge (Monat 0 bis 240) |
| S4 | Reproduzierbarkeit | zweimal gleiche Eingaben, `seed = 42` | identische `timeline` (NF-08) |
| S5 | Unterschiedlicher Seed | `seed = 42` und `seed = 43` | `timeline` unterscheidet sich |
| S6 | Einzahlungen | `S = 10.000`, `C = 200`, `H = 20` | Summe der Einzahlungen 58.000 € |
| S7 | Rebalancing und Korrelation | zwei Anlagen mit genau gegenläufigen Monatsrenditen (+2 % / −2 % im Wechsel), je 50 % | Portfoliorendite `R_t = 0` in jedem Monat, also `σ = 0` |
| S8 | Zu kurze Historie | eine Anlage mit 35 Monatswerten, eine mit 30 | `422 INSUFFICIENT_DATA`, beide Anlagen genannt (E6, E8) |
| S9 | Schätzzeitraum | Anlage A mit 200, Anlage B mit 60 Monatswerten | gemeinsamer Schätzzeitraum = 60 Monatswerte (59 Renditen) (E17) |
| S10 | Laufender Monat | Daten enthalten den laufenden Monat | dieser Monat wird nicht verwendet (E22) |

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 08.10.2026 | Neu: Rechenregeln der Simulation auf Grundlage von E15 bis E23 | Rückfrage aus BE #40 (Monte-Carlo-Engine), Docs #72 |
