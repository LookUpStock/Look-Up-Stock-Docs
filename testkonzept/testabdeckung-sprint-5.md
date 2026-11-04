# Testabdeckung Sprint 5 (Docs #208)

Auswertung der Testabdeckung der Kernlogik (NF-11) zum Ende von Sprint 5. Seit ADR-07 gibt es im Backend zwei Dienste mit je eigener CI (`core-service`, `marktdaten-service`); die Kernlogik nach Testkonzept Abschnitt 9 liegt vollständig im `core-service`.

## 1. Messgrundlage

| Punkt | core-service | marktdaten-service | Frontend |
|---|---|---|---|
| Stand | `main`, Commit `5a32df2` (01.11.2026, Merge von PR #127 zu BE #113 „CORS bei Fehlerantworten“) | `main`, Commit `3d7953f` (01.11.2026, Merge von PR #125 zu BE #106 „Kursdaten vorladen“) | `main`, Commit `a839c19` (03.11.2026, Merge von PR #66 zu FE #51) |
| CI-Lauf | Backend CI (core-service), Lauf #94 ([41319027256](https://github.com/LookUpStock/Look-Up-Stock-BackEnd/actions/runs/41319027256)), grün | Backend CI (marktdaten-service), Lauf #6 ([41316974457](https://github.com/LookUpStock/Look-Up-Stock-BackEnd/actions/runs/41316974457)), grün | Frontend CI, Lauf #68 ([41600465794](https://github.com/LookUpStock/Look-Up-Stock-FrontEnd/actions/runs/41600465794)), grün |
| Werkzeug | JaCoCo 0.8.14, Java 25 (Temurin) | JaCoCo 0.8.14, Java 25 (Temurin) | Node.js Test Runner (`node --test`) |
| Testlauf | 307 Tests, 0 fehlgeschlagen, 0 übersprungen | 33 Tests, 0 fehlgeschlagen, 0 übersprungen | 160 Tests, 0 fehlgeschlagen |
| Bericht | CI-Artefakt `jacoco-report`, Zusammenfassung des Laufs | CI-Artefakt `marktdaten-jacoco-report` | keine Abdeckung in der CI, siehe Abschnitt 3 |

**Messung:** Unverändert seit Sprint 4 führt die CI `./mvnw -B verify` aus. JaCoCo erzeugt den Bericht und prüft mit der Regel `check-core-logic` die sieben Klassen der Kernlogik. Lauf #94 meldet „All coverage checks have been met.“, die Zeilenabdeckung der Kernlogik liegt also bei mindestens 80 %.

**Änderung gegenüber Sprint 4:** Mit der Aufteilung in zwei Dienste (ADR-07) hat jeder Dienst eine eigene CI mit Pfadfilter (`core-service/**`, `marktdaten-service/**`). Die Zahl der Tests im `core-service` stieg von 267 auf 307 (+40), u. a. durch Tests für Queue (`DatabaseMessagePublisherTest`, `PricePreloadRequesterTest`, `UsageEventRecorderTest`, `UsageEventConsumerTest`), GraphQL (`QueryControllerTest`, `MutationControllerTest`), `MarktdatenClientTest` und `CorsConfigTest`.

## 2. Zeilen- und Zweigabdeckung der Kernlogik (NF-11)

| Klasse | Zeilen | Zeilen % | Zweige | Zweige % |
|---|---|---|---|---|
| `ParameterEstimator` | 59/60 | 98,3 % | 30/34 | 88,2 % |
| `MonteCarloEngine` | 31/31 | 100 % | 10/10 | 100 % |
| `MetricsCalculator` | 57/57 | 100 % | 28/28 | 100 % |
| `SimulationService` | 35/38 | 92,1 % | 8/8 | 100 % |
| `ComparisonService` | 67/67 | 100 % | 37/38 | 97,4 % |
| `Portfolio` | 57/64 | 89,1 % | 29/32 | 90,6 % |
| `PortfolioService` | 38/38 | 100 % | 14/14 | 100 % |
| **Kernlogik gesamt** | **344/355** | **96,9 %** | **156/164** | **95,1 %** |

Quelle: `jacoco.csv` aus dem CI-Artefakt `jacoco-report` von Lauf #94 (JaCoCo-Sitzung vom 01.11.2026, 16:01 Uhr); innere Klassen (z. B. von `Portfolio`) sind ihrer Klasse zugerechnet, wie in der Regel `check-core-logic`. Der Gesamtwert entspricht der Zusammenfassung des CI-Laufs.

**Bewertung:** Die Kernlogik erreicht 96,9 % Zeilenabdeckung (344 von 355 Zeilen) und 95,1 % Zweigabdeckung und liegt damit deutlich über dem Zielwert von 80 %. Gegenüber Sprint 4 kamen 11 Zeilen hinzu (`SimulationService` +2, `ComparisonService` +2, `PortfolioService` +7), alle abgedeckt; die Zweigabdeckung ist unverändert. Nicht abgedeckt sind weiterhin 7 Zeilen in `Portfolio`, 3 in `SimulationService` und 1 in `ParameterEstimator`. NF-11 ist für Sprint 5 durch die bestandene Prüfung `check-core-logic` in Lauf #94 nachgewiesen (AT-ADR7-08, Abnahmeprotokoll Sprint 5).

### Vergleich mit den Vorsprints

| Messung | Kernlogik Zeilen % | Kernlogik Zweige % | Tests core-service | Messort |
|---|---|---|---|---|
| Sprint 3 (`d9c02ab`) | 96,8 % | 95,1 % | – | lokal |
| Sprint 4 (`dd88784`) | 96,8 % | 95,1 % | 267 | CI |
| Sprint 5 (`5a32df2`) | 96,9 % | 95,1 % | 307 | CI |

### Gesamter core-service

Über alle 121 Klassen des `core-service` liegt die Zeilenabdeckung bei 89,1 % (1240/1391), die Zweigabdeckung bei 76,2 % (355/466).

| Paket | Zeilen | Zeilen % |
|---|---|---|
| `queue` | 61/61 | 100 % |
| `dto` | 126/127 | 99,2 % |
| `exception` | 75/81 | 92,6 % |
| `controller` | 93/101 | 92,1 % |
| `service` | 494/561 | 88,1 % |
| `model` | 249/283 | 88,0 % |
| `client` | 44/52 | 84,6 % |
| `graphql` | 80/100 | 80,0 % |
| `config` | 17/22 | 77,3 % |
| Hauptklasse (`BackendApplication`) | 1/3 | 33,3 % |

## 3. Weitere Dienste und Frontend (nicht Teil von NF-11)

**marktdaten-service:** 33 Tests in sechs Testklassen (`PriceServiceTest`, `SearchServiceTest`, `SearchCacheTest`, `InternalControllerTest`, `PricePreloadConsumerTest`, `TwelveDataErrorTest`). JaCoCo erstellt einen Bericht über 30 Klassen; eine Prüfregel gibt es nicht, weil der Dienst keine Kernlogik nach Abschnitt 9 enthält. Der Bericht liegt im CI-Artefakt `marktdaten-jacoco-report`.

**Frontend:** Die Frontend-CI führt `npm test` ohne Abdeckungsmessung aus. Zur Einordnung wurde die Abdeckung lokal auf demselben Commit `a839c19` mit `node --test --experimental-test-coverage` gemessen (Node.js 22, CI nutzt Node.js 24; Testdateien ausgenommen): 160 Tests, **53,0 % Zeilen, 85,9 % Zweige, 82,8 % Funktionen** über `src/js`.

| Datei | Zeilen % | Zweige % |
|---|---|---|
| `api.js` | 97,1 % | 96,2 % |
| `portfolio.js` | 95,1 % | 87,7 % |
| `dashboard.js` | 93,0 % | 75,0 % |
| `config.js` | 86,7 % | 50,0 % |
| `portfolio-save.js` | 75,0 % | 73,3 % |
| `chart.js` | 63,2 % | 84,8 % |
| `dashboard-export.js` | 61,5 % | 83,3 % |
| `auth.js` | 60,9 % | 85,1 % |
| `spa/router.js` | 58,8 % | 90,9 % |
| `simulation.js` | 55,3 % | 96,8 % |
| `saved-simulations.js` | 47,2 % | 92,7 % |
| `search.js` | 42,5 % | 94,7 % |
| `demo.js` | 42,2 % | 83,3 % |
| `comparison.js` | 17,1 % | 27,8 % |
| `portfolio-ui.js` | 10,6 % | 100 % |

## 4. Interpretation der Metrik (Aufgaben III.3 Nr. 4)

- **Zielwert erreicht, aber nur ein Ausschnitt:** Die Prüfregel bezieht sich auf sieben Klassen. Die übrigen Klassen des `core-service` (Controller, Queue, GraphQL, Konfiguration) gehen nicht in den Zielwert ein. Das ist gewollt (Testkonzept Abschnitt 9), heißt aber, dass der Wert der Kernlogik (96,9 %) nicht für den ganzen Dienst gilt: Dort liegt die Zeilenabdeckung bei 89,1 % und die Zweigabdeckung bei nur 76,2 %; am schwächsten sind `graphql` und `config`.
- **Ausgeführt ist nicht geprüft:** Die Abdeckung zeigt nur, dass Code durchlaufen wurde. Ob die Tests die Ergebnisse prüfen, stellt das Code-Review sicher. Fehler wie B8 (gespeicherte Simulation zählt doppelt, BE #128) fallen bei hoher Abdeckung nicht auf, weil beide Erfassungen getestet sind; erst der Abnahmetest AT-11-03 hat den fachlichen Fehler gezeigt.
- **Fehlende Anforderungen erkennt die Metrik nicht:** Abweichungen wie die nicht dokumentierten Asset-Endpunkte `/stats` und `/prices` bzw. der dokumentierte, aber fehlende `GET /assets/{assetId}` (BE #129) sind über die Abdeckung nicht sichtbar. Sie fallen erst beim Abgleich mit dem API-Design auf.
- **Frontend:** Der niedrige Gesamtwert entsteht vor allem in den Dateien, die direkt das DOM bearbeiten (`portfolio-ui.js`, `comparison.js`, Anzeigeteile von `simulation.js`, `search.js`, `demo.js`). Ohne Browser-Umgebung laufen diese Teile in `node --test` nicht. Die Logik ohne DOM ist gut abgedeckt (`api.js` 97 %, `portfolio.js` 95 %). Das Projekt trennt deshalb seit Sprint 3 Logik und Anzeige („Logik: ohne Browser testbar“), damit die prüfbare Logik in Tests landet; die Anzeige deckt der manuelle Abnahmetest ab.

## 5. Maßnahmen aus Sprint 4

| Maßnahme | Stand |
|---|---|
| Nicht abgedeckte Zweige in `ParameterEstimator` und `Portfolio` prüfen | offen (optional), unverändert 4 bzw. 3 Zweige |
| Integrationstest für den Kursdaten-Cache | teilweise: Der 24-h-Cache (NF-07) ist seit BE #106 im `PriceServiceTest` des marktdaten-service als Unit-Test mit Mockito abgedeckt, dazu `SearchCacheTest`. Ein Integrationstest gegen die Testdatenbank fehlt weiterhin. |
| Unit-Tests für `AssetStatisticsService` | entfällt, die Klasse wird mit BE #129 entfernt (Sprint Planning 6, Variante b) |

## 6. Bericht im Sprint Review

Für Sprint Review 6 (Docs #211) wird der Wert aus der Zusammenfassung des letzten CI-Laufs auf `main` übernommen, nach dem Merge von BE #128 und BE #129. Mit BE #129 entfallen `AssetStatisticsService` und `AssetStatsControllerTest`; die Kernlogik ist davon nicht betroffen.
