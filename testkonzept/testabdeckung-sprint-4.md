# Testabdeckung Sprint 4 (Docs #138)

Auswertung der Testabdeckung der Kernlogik (NF-11) zum Ende von Sprint 4. Erstmals stammt die Messung aus der CI-Pipeline des Backends und nicht aus einem lokalen Lauf.

## 1. Messgrundlage

| Punkt | Wert |
|---|---|
| Stand | `main`, Commit `dd88784` (25.10.2026, Merge von PR #104 zu BE #102 „JaCoCo in CI“) |
| CI-Lauf | Backend CI, Lauf [40224764870](https://github.com/LookUpStock/Look-Up-Stock-BackEnd/actions/runs/40224764870), Ergebnis grün |
| Werkzeug | JaCoCo 0.8.14 als Plugin in der `pom.xml`, Java 25 (Temurin) |
| Testlauf | 267 Tests, 0 fehlgeschlagen, 0 übersprungen |
| Bericht | CI-Artefakt `jacoco-report` (HTML, XML, CSV) und Zusammenfassung des CI-Laufs |

**Messung:** Die CI führt `./mvnw -B verify` aus. Dabei startet JaCoCo vor den Tests (`prepare-agent`), erzeugt danach den Bericht (`report`) und prüft die Kernlogik (`check-core-logic`). Die Prüfung umfasst die Klassen aus Testkonzept Abschnitt 9 und lässt den Build fehlschlagen, wenn die Zeilenabdeckung unter 80 % liegt.

**Behobene Abweichung aus Sprint 3:** In Sprint 3 wurde nur lokal gemessen, weil JaCoCo nicht in `pom.xml` und CI eingebunden war (Testabdeckung Sprint 3, Abschnitt 1). Mit BE #102 ist das umgesetzt.

## 2. Zeilen- und Zweigabdeckung der Kernlogik (NF-11)

| Klasse | Zeilen | Zeilen % | Zweige | Zweige % |
|---|---|---|---|---|
| `ParameterEstimator` | 59/60 | 98,3 % | 30/34 | 88,2 % |
| `MonteCarloEngine` | 31/31 | 100 % | 10/10 | 100 % |
| `MetricsCalculator` | 57/57 | 100 % | 28/28 | 100 % |
| `SimulationService` | 33/36 | 91,7 % | 8/8 | 100 % |
| `ComparisonService` | 65/65 | 100 % | 37/38 | 97,4 % |
| `Portfolio` | 57/64 | 89,1 % | 29/32 | 90,6 % |
| `PortfolioService` | 31/31 | 100 % | 14/14 | 100 % |
| **Kernlogik gesamt** | **333/344** | **96,8 %** | **156/164** | **95,1 %** |

Quelle: `jacoco.csv` aus dem CI-Artefakt `jacoco-report` des oben genannten Laufs; innere Klassen (z. B. von `Portfolio`) sind ihrer Klasse zugerechnet, wie in der Regel `check-core-logic`.

**Bewertung:** NF-11 ist erstmals per CI nachgewiesen. Die Kernlogik erreicht 96,8 % Zeilenabdeckung (333 von 344 Zeilen) und 95,1 % Zweigabdeckung und liegt damit deutlich über dem Zielwert von 80 %. Der Nachweis ist dauerhaft: Fällt der Wert unter 80 %, schlägt die Regel `check-core-logic` an und der CI-Lauf wird rot.

### Vergleich mit Sprint 3

| Messung | Kernlogik Zeilen % | Kernlogik Zweige % | Messort |
|---|---|---|---|
| Sprint 3 (`d9c02ab`) | 96,8 % | 95,1 % | lokal |
| Sprint 4 (`dd88784`) | 96,8 % | 95,1 % | CI |

## 3. Maßnahmen aus Sprint 3

| Maßnahme | Stand |
|---|---|
| JaCoCo in `pom.xml` und Backend-CI aufnehmen | erledigt (BE #102) |
| Abdeckung des Pakets `client` über Mock-Testdaten erhöhen | umgesetzt (BE #89, `MockMarketDataProviderTest`); Wirkung im Bericht des Pakets `client` prüfen |
| 503-Test für `POST /comparisons` ergänzen | erledigt (BE #90) |
| Nicht abgedeckte Zweige in `ParameterEstimator` und `Portfolio` prüfen | offen (optional) |
| Integrationstest für den Kursdaten-Cache, Unit-Tests für `AssetStatisticsService` | offen, für Sprint 5 vorschlagen |

## 4. Bericht im Sprint Review

NF-11 verlangt die Angabe der Abdeckung in jedem Sprint Review. Ab Sprint 5 wird der Wert aus der Zusammenfassung des jeweils letzten CI-Laufs auf `main` übernommen.