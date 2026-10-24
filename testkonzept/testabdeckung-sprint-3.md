# Testabdeckung Sprint 3 (Docs #102)

Auswertung der Testabdeckung der Kernlogik (NF-11) und der Pfadabdeckung des Aktivitätsdiagramms US-07 zum Ende von Sprint 3.

## 1. Messgrundlage

| Punkt | Wert |
|---|---|
| Stand | `main`, Commit `d9c02ab` (18.10.2026, Ende Sprint 3) |
| Werkzeug | JaCoCo 0.8.14, Java 25 |
| Testlauf | 224 Tests, 0 fehlgeschlagen, 0 übersprungen |
| Gemessen am | 22.10.2026 |

**Messung:** Die Messung wurde lokal durchgeführt, mit JaCoCo direkt über die Kommandozeile und ohne Änderung an der `pom.xml`:

```bash
./mvnw org.jacoco:jacoco-maven-plugin:0.8.14:prepare-agent test org.jacoco:jacoco-maven-plugin:0.8.14:report
```

**Abweichung vom Testkonzept:** Laut Testkonzept (Abschnitt 9) erzeugt die Backend-CI bei jedem Lauf einen JaCoCo-Bericht. Das ist derzeit nicht der Fall: Die `pom.xml` enthält kein JaCoCo-Plugin, und die CI lädt nur die Surefire-Testberichte hoch. Siehe Maßnahmen in Abschnitt 5.

## 2. Zeilen- und Zweigabdeckung der Kernlogik (NF-11)

| Klasse | Zeilen | Zeilen % | Zweige | Zweige % |
|---|---|---|---|---|
| `ParameterEstimator` | 59/60 | 98,3 % | 30/34 | 88,2 % |
| `MonteCarloEngine` | 31/31 | 100 % | 10/10 | 100 % |
| `MetricsCalculator` | 57/57 | 100 % | 28/28 | 100 % |
| `SimulationService` | 33/36 | 91,7 % | 8/8 | 100 % |
| `ComparisonService` | 64/64 | 100 % | 37/38 | 97,4 % |
| `Portfolio` | 57/64 | 89,1 % | 29/32 | 90,6 % |
| `PortfolioService` | 31/31 | 100 % | 14/14 | 100 % |
| **Kernlogik gesamt** | **332/343** | **96,8 %** | **156/164** | **95,1 %** |

**Bewertung:** NF-11 ist erfüllt. Die Kernlogik erreicht insgesamt 96,8 % Zeilenabdeckung, und jede einzelne Klasse liegt über dem Zielwert von 80 %.

### Vergleich mit dem Gesamtprojekt

| Paket | Zeilen | Zeilen % | Zweige % |
|---|---|---|---|
| `service` | 412/499 | 82,6 % | 72 % |
| `client` | 16/89 | 18,0 % | 21 % |
| `model` | 186/228 | 81,6 % | 92 % |
| `exception` | 62/77 | 80,5 % | 33 % |
| `controller` | 68/76 | 89,5 % | 62 % |
| `dto` | 112/114 | 98,2 % | 66 % |
| `config` | 9/13 | 69,2 % | – |
| `backend` (Startklasse) | 1/3 | 33,3 % | – |
| **Gesamt (97 Klassen)** | **866/1.099** | **78,8 %** | **70 %** (290/414) |

Das Gesamtprojekt liegt mit 78,8 % Zeilenabdeckung knapp unter 80 %. Das betrifft NF-11 nicht, weil der Zielwert nur für die Kernlogik gilt. Die Differenz zur Kernlogik (96,8 %) entsteht vor allem im Paket `client` mit nur 18 %. Dort liegt die Anbindung an die Finanzdaten-API, die in den Tests nicht aufgerufen wird (Dummy-Schlüssel in der CI, siehe Testkonzept). Das ist gewollt, heißt aber: Fehler in der Verarbeitung der API-Antworten fallen nur in manuellen Tests auf. Testdaten im `MockMarketDataProvider` (BE #89) und gezielte Tests mit aufgezeichneten Antworten würden diese Lücke verkleinern. Im Paket `exception` sind nur 33 % der Zweige abgedeckt; nicht alle Fehlerfälle werden in Tests ausgelöst.

### Klassen mit auffällig niedriger Abdeckung

| Klasse | Paket | Zeilen | Zeilen % | Zweige % | Ursache |
|---|---|---|---|---|---|
| `AssetStatisticsService` | `service` | 0/48 | 0 % | 0 % | Wird nur über `AssetStatsControllerTest` (`@WebMvcTest`) angesprochen; dort ist der Service gemockt. Keine eigenen Unit-Tests. |
| `PriceCache` | `service` | 0/21 | 0 % | 0 % | Kursdaten-Cache in der Datenbank (NF-07). In `PriceHistoryServiceTest` gemockt. |
| `PriceHistory` | `model` | 0/13 | 0 % | – | Entity des Kursdaten-Caches, wird nur über `PriceCache` genutzt. |
| `AssetService` | `service` | 22/33 | 66,7 % | 42 % | Suche (US-02); mehrere Zweige ohne Test. |
| `AuthToken` | `model` | 7/15 | 46,7 % | 100 % | Methoden ohne Aufruf in Tests. |
| `User` | `model` | 7/12 | 58,3 % | – | Methoden ohne Aufruf in Tests. |

Alle drei Klassen mit 0 % werden im Code verwendet; es handelt sich nicht um toten Code. `AssetStatisticsService` und `PriceCache` allein verursachen 69 der 87 nicht abgedeckten Zeilen im Paket `service`. Ohne sie läge das Paket bei 95,8 % (412/430).

**Bezug zur Pfadabdeckung:** Die Verzweigung „Kursdaten im Cache“ der Aktivitätsdiagramme US-04 und US-07 gilt über AT-07-11 als abgedeckt. Getestet ist dort aber nur die Entscheidung in `PriceHistoryService` mit einem gemockten Cache. Das tatsächliche Lesen und Schreiben des Caches in der Datenbank (`PriceCache`, `PriceHistory`) läuft in keinem automatisierten Test. Ein Integrationstest mit der Testdatenbank würde diese Lücke schließen.

## 3. Interpretation

- **Kein Durchschnittseffekt:** Auch die am schwächsten abgedeckten Klassen liegen deutlich über 80 %: `Portfolio` mit 89,1 % und `SimulationService` mit 91,7 %. Der hohe Gesamtwert verdeckt also keine schwach getestete Kernklasse (vgl. Testkonzept, Abschnitt 9).
- **Zweigabdeckung:** Die Prüfregeln haben viele Verzweigungen. Bei ihnen ist die Zweigabdeckung aussagekräftiger als die Zeilenabdeckung. Die meisten nicht durchlaufenen Zweige liegen in `ParameterEstimator` (4 von 34) und `Portfolio` (3 von 32). Welche Zweige das sind, zeigt der JaCoCo-Bericht (gelb oder rot markierte Zeilen). Das sollte geprüft werden.
- **Neue Klasse `ComparisonService`:** Die Klasse ist in Sprint 3 neu hinzugekommen (ADR-05). Sie ist bereits zu 100 % der Zeilen und 97,4 % der Zweige abgedeckt.
- **Grenzen der Metrik:** Die Abdeckung zeigt nur, dass Code ausgeführt wurde. Ob die Tests die Ergebnisse auch prüfen, stellt das Code-Review sicher. Fehlende Funktionen erkennt die Metrik nicht. So fehlt z. B. die Anzeige von Seed und Schätzzeitraum im Vergleich (B2). Solche Lücken decken die Abnahmetestfälle auf (Abschnitt 4).

## 4. Pfadabdeckung Aktivitätsdiagramm US-07

Grundlage sind `diagramme/aktivitaetsdiagramm_us-07_v1.puml`, die Verzweigungstabelle im Testkonzept (Abschnitt 5) und die Status in `testkonzept/abnahmetestfaelle-sprint-3.md`. Ein Pfad gilt als abgedeckt, wenn mindestens ein zugeordneter Testfall ihn erfolgreich durchlaufen hat.

| Verzweigung | Pfad | Testfälle | Abgedeckt |
|---|---|---|---|
| Eingaben prüfen | gültig | AT-07-03, AT-07-12 | ja |
| | ungültig (400) | AT-07-05, AT-07-12 | ja |
| Art des Eintrags | Portfolio | AT-07-01 ¹ | ja |
| | Anlage | AT-07-01 ¹ | ja |
| Portfolio laden | vorhanden | AT-07-01 ¹ | ja |
| | nicht vorhanden oder fremd (404) | AT-07-06 | ja |
| Anlage laden | vorhanden | AT-07-01 ¹ | ja |
| | nicht vorhanden (404) | AT-07-07 | ja |
| Bezeichnung | angegeben | AT-07-02 | **nein** (offen, B3) |
| | keine Bezeichnung | AT-07-01 ¹ | ja |
| Schleife über die Einträge | weiterer Eintrag | AT-07-01 ¹ | ja |
| | beide Einträge geladen | AT-07-01 ¹ | ja |
| Seed | ohne Seed | AT-07-01 ¹ | ja |
| | mit Seed | AT-07-03 (automatisiert) | ja |
| Kursdaten im Cache | im Cache, jünger als 24 h | AT-07-11 | ja |
| | nicht im Cache oder älter | AT-07-11 | ja |
| Finanzdaten-API | erreichbar | AT-07-01 ¹ | ja |
| | nicht erreichbar (503) | AT-07-11 | **teilweise** ² |
| Kurshistorie je Anlage | alle ≥ 36 Monatswerte | AT-07-01 ¹ | ja |
| | ohne Kurse oder < 36 Monatswerte (422) | AT-07-08, AT-07-09 | **nein** (nicht durchgeführt) |
| Gemeinsamer Schätzzeitraum | ≥ 36 Monatswerte | AT-07-01 ¹ | ja |
| | < 36 Monatswerte (422) | AT-07-10 | ja |
| Zielbetrag | angegeben | AT-07-02 | **nein** (offen, B1) |
| | kein Zielbetrag | AT-07-01 ¹ | ja |

¹ AT-07-01 ist fehlgeschlagen, aber nur wegen der fehlenden Anzeige von Seed und gemeinsamem Schätzzeitraum (B2). Die übrigen Punkte sind erfüllt, die Pfade selbst wurden also durchlaufen.
² Das Verhalten bei nicht erreichbarem Anbieter ist nur über `/simulations` getestet (`SimulationControllerTest`). Einen eigenen 503-Test für `/comparisons` gibt es nicht.

**Ergebnis:** Von 24 Pfaden sind 20 abgedeckt (83 %), 1 teilweise und 3 nicht.

**Interpretation:** Alle offenen Pfade gehören zu manuellen Testfällen. Die automatisierten Tests decken alle ihnen zugeordneten Pfade ab. Für die Lücken gibt es zwei Ursachen:
- Frontend-Fehler verhindern die Durchführung: Es fehlen die Eingaben für Bezeichnung (B3) und Zielbetrag (B1).
- Manuelle Alternativszenarien wurden nicht durchgeführt (AT-07-08, AT-07-09).

## 5. Maßnahmen

| Maßnahme | Issue |
|---|---|
| Eingabe Zielbetrag im Vergleich, danach AT-07-02 durchführen | FE #30 (B1) |
| Eigene Bezeichnung je Eintrag, danach AT-07-02 durchführen | FE #32 (B3) |
| Anzeige Seed und Schätzzeitraum, danach AT-07-01 wiederholen | FE #31 (B2) |
| 503-Test für `POST /comparisons` ergänzen | BE #90 |
| AT-07-04, AT-07-08 und AT-07-09 manuell durchführen | Docs #136 / #137 |
| JaCoCo in `pom.xml` und Backend-CI aufnehmen, damit die Messung wie im Testkonzept beschrieben in jedem CI-Lauf erfolgt | neues Issue vorschlagen |
| Nicht abgedeckte Zweige in `ParameterEstimator` und `Portfolio` prüfen | optional |
| Abdeckung des Pakets `client` über Mock-Testdaten erhöhen | BE #89 |
| Integrationstest für den Kursdaten-Cache (`PriceCache`, `PriceHistory`) mit der Testdatenbank | neues Issue vorschlagen |
| Unit-Tests für `AssetStatisticsService` | neues Issue vorschlagen |
| Fehlende Zweige in `AssetService` bei der Analyse der Tickersuche mit abdecken | BE #88 |
