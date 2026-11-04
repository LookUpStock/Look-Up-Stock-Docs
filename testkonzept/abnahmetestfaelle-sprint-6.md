# Abnahmetestfälle – Sprint 6

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 6 (03.11.2026 – 09.11.2026), Nachtest-Sprint vor der Abgabe |
| **Issue** | Docs #214 Abnahmetests Sprint 6, Abnahmeprotokoll Sprint 6 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 04.11.2026, vor der Abnahme; automatisierte Tests nach dem Merge von BE #128, FE #67 bis FE #71 eingetragen |
| **Durchführung** | Nachtest von AT-11-03 und AT-11-06 nach dem Deploy (BE #130), übrige Testfälle danach; Ergebnis im Abnahmeprotokoll Sprint 6 (Docs #214) |
| **Stories** | US-11 Nutzungskennzahlen anzeigen (Docs #156), nach den Fehlern B8 und B9 nicht abgenommen; Hinweise H6 und H8 bis H12 aus der Abnahme Sprint 5 |
| **Grundlagen** | `anforderungen/nfr_furps.md` Version 5 (NF-05, NF-13), [Abnahmeprotokoll Sprint 5](abnahmeprotokoll-sprint-5.md) (Abschnitte „Befunde“ und „Folgen“), [Abnahmetestfälle Sprint 5](abnahmetestfaelle-sprint-5.md), Sprint-Planning-Protokoll Sprint 6 (Docs #210; Entscheidung zu B8: Variante a), `anforderungen/kurzformat_sprint-4.md` (US-11, E37, E40), Umsetzung BE #128 (B8), BE #130 (Deploy), FE #67 (B9), FE #68 (H8), FE #69 (H9), FE #70 (H10), FE #71 (H12), Docs #213 (H11), BE #135 und FE #81 (B11, NF-05), [Testkonzept](testkonzept.md) Version 6 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt [abnahmetestfaelle-sprint-5.md](abnahmetestfaelle-sprint-5.md) fort. In Sprint 6 gibt es keine neuen User Stories. Sie enthält den Nachtest der Testfälle, die in Sprint 5 fehlgeschlagen sind oder einen Hinweis ergeben haben, einen neuen Testfall für die präzisierte Zählung gespeicherter Simulationen (E37, Variante a) und die Nachweise für NF-05 und NF-13, die bisher keinem Testfall zugeordnet waren (NFRs Version 5).

---

## Vorbemerkungen

Es gelten die Vorbemerkungen aus Sprint 1 bis 5 (Bezug, Durchführung, Darstellung von UI und API, Status, Meldungstexte, Testdaten auf US-Börsen, Basiseingaben, Seed `42`, Zielbetrag `100000`, Token im Header nach ADR-06, Abnahmeumgebung mit zwei Backend-Apps, Nutzer M mit der Rolle `MANAGER`, Wartezeit von 10 s vor dem Ablesen des Dashboards, zwei Browserfenster). Zusätzlich gilt:

- **Zeitpunkt (BE #130):** AT-11-03 und AT-11-06 werden erst durchgeführt, wenn Carl Stibbe den Deploy von core-service und Frontend nach dem Merge von BE #128 und FE #67 bestätigt hat. Die übrigen Testfälle setzen den Merge der jeweiligen Behebung voraus.
- **Zählung im November 2026:** Bis zum Deploy von BE #128 wurde jede über die Oberfläche gespeicherte Simulation zweimal als „Durchgeführte Simulation“ erfasst (B8). Dieser Wert wird nachträglich nicht korrigiert. AT-11-03 vergleicht deshalb nur die Differenz vor und nach den Schritten, nicht den absoluten Wert. Der Vermerk steht im Abnahmeprotokoll Sprint 6.
- **SPA-Adresse (H11):** Das Frontend wird auf Heroku mit `serve` ausgeliefert. Es leitet `app.html` mit `301` auf `/app` um; der Hash bleibt erhalten. Die SPA ist deshalb unter `/app#/…` erreichbar (ADR-08). Links in der Anwendung zeigen weiter auf `app.html#/…`.

**Anzahl der Testfälle**

| Bereich | Normalfall | Alternativszenario | Grenzwert | NFR | Summe | davon manuell | davon automatisiert |
|---|---|---|---|---|---|---|---|
| Nachtest Sprint 5 (B8, B9, H6, H8 bis H12) | 4 | 1 | 0 | 2 | 7 | 7 | 0 |
| US-11 neu (E37, Variante a) | 0 | 1 | 0 | 0 | 1 | 0 | 1 |
| NF-05, NF-13 | 0 | 0 | 0 | 2 | 2 | 2 | 0 |
| **Gesamt** | 4 | 2 | 0 | 4 | 10 | 9 | 1 |

Die Art eines Nachtests richtet sich nach dem Testfall aus Sprint 5. Bei Testfällen mit „manuell und automatisiert“ zählt die manuelle Durchführung.

---

## Nachtest der Befunde aus Sprint 5

Nach dem Merge der Behebungen werden die betroffenen Testfälle erneut durchgeführt (Testkonzept, Abschnitt 11). Die Testfälle bleiben in den Dateien der früheren Sprints definiert; es gilt die Spalte „Beim Nachtest“.

| Testfall | Befund | Beim Nachtest | Grund | Ergebnis |
|---|---|---|---|---|
| AT-11-03 | B8 (BE #128) | unverändert wie in Sprint 5 mit einem neuen Zählkonto `zaehltest-<TTMM>@example.com`. In Schritt 4 wird „Zähltest“ zuerst simuliert und danach als „Zähltest Simulation“ gespeichert. | Eine gespeicherte Simulation zählte zweimal als durchgeführte Simulation. Nach Variante (a) erfasst das Speichern (`save: true`) nur „Gespeicherte Simulationen“ (E37). | offen |
| AT-11-06 | B9 (FE #67) | unverändert; zusätzlich in Schritt 1 auf `dashboard.html` die Navigation prüfen: kein Eintrag „Auswertungen“, Eintrag „App (SPA)“ vorhanden, gleiche Einträge wie auf `index.html` | „Auswertungen“ stand fest im HTML von `dashboard.html`, „App (SPA)“ fehlte | offen |
| AT-11-01 | B9 (FE #67), H10 (FE #70) | unverändert; zusätzlich: Als M erscheint „Auswertungen“ in der Navigation von `dashboard.html` **genau einmal**, „App (SPA)“ ist vorhanden. Nach dem Laden der Kennzahlen steht über der Tabelle **kein** leeres blaues Hinweisfeld. Danach Oktober 2026 wählen: Die Meldung „Für diesen Monat liegen keine Auswertungen vor.“ erscheint weiterhin. | Für M stand „Auswertungen“ doppelt in der Navigation (B9); nach dem Laden blieb ein leeres Hinweisfeld stehen (H10) | offen |
| AT-NF10-01 (Fächerdiagramme) | H6, H8 (FE #68) | Als Nutzer A auf der Vergleichsseite „Mein Weltportfolio“ gegen „Mein Aktienportfolio“ vergleichen mit Startkapital 100000, Sparrate 1000, Anlagehorizont 40, Seed 42. Am Desktop und bei 360 px Breite die Y-Achse beider Fächerdiagramme ansehen; danach auf der Simulationsseite „Mein Weltportfolio“ mit denselben Eingaben simulieren. | Die Y-Achse war auf der Vergleichsseite bei Beschriftungen ab acht Ziffern abgeschnitten („.000.000 €“); deshalb war H6 nicht behoben | offen |
| Sichtprüfung H4 | H9 (FE #69) | Als Nutzer A den Portfolio-Editor für ein neues Portfolio öffnen: Die Gewichtssumme zeigt „—“ ohne rote oder grüne Farbe. URTH übernehmen, Gewicht 40 eintragen → rot „ungültig“; Gewicht auf 100 ändern → „gültig“. Danach „Mein Weltportfolio“ zum Bearbeiten öffnen → der Status erscheint sofort als „gültig“. | Der leere Editor zeigte vor jeder Eingabe rot „ungültig“ | offen |
| AT-ADR8-01 | H12 (FE #71) | unverändert; zusätzlich in der Portfolioliste der SPA ein Portfolio mit genau einer Anlage ansehen (z. B. „Zähltest“, URTH 100 %) | Die SPA zeigte „1 Positionen“ | offen |
| AT-ADR8-01 bis AT-ADR8-03 (Adresse) | H11 (Docs #213) | 1. `app.html#/portfolios` aufrufen und die Adresszeile ansehen.<br>2. ADR-08 und `abnahmetestfaelle-sprint-5.md` lesen. | Die SPA ist auf Heroku unter `/app#/…` erreichbar, die Dokumente nannten nur `app.html#/…` | offen |

**Erwartete Ergebnisse des Nachtests**

- **AT-11-03:** Gegenüber Schritt 1 steigen Neue Registrierungen, Anmeldungen, Angelegte Portfolios, **Durchgeführte Simulationen**, **Gespeicherte Simulationen**, Durchgeführte Vergleiche, Beispielsimulationen (Gäste) und Nutzerkonten insgesamt jeweils um genau 1; Anlagensuchen um die Zahl der erfolgreichen Suchanfragen aus Schritt 3 und 5. Die Beispielsimulation in Schritt 6 wird genau einmal gestartet.
- **AT-11-06 und AT-11-01:** Für Nutzer A gibt es auf keiner MPA-Seite einen Eintrag „Auswertungen“; für M steht er auf jeder MPA-Seite genau einmal. „App (SPA)“ steht auf allen MPA-Seiten einschließlich `dashboard.html`. Die übrigen erwarteten Ergebnisse gelten wie in Sprint 5.
- **AT-NF10-01 (Fächerdiagramme):** Die Y-Achsenbeschriftung ist auf beiden Seiten vollständig lesbar, auch die längste Beschriftung (mindestens achtstellig, z. B. „10.000.000 €“). Beide Diagramme im Vergleich haben dieselbe Y-Achse (H1). Bei 360 px gibt es kein horizontales Scrollen und keine Überlappung.
- **Sichtprüfung H4 (H9):** wie in den Schritten beschrieben; die Feldmeldungen verhalten sich weiter wie beim Nachtest von H4 in Sprint 5.
- **AT-ADR8-01 (H12):** „1 Position“ im Singular, „Mein Weltportfolio“ mit „2 Positionen“, wie in der MPA.
- **H11:** 1. Die Adresszeile zeigt `/app#/portfolios`, die Portfolioliste erscheint. 2. Beide Dokumente nennen die Adresse `/app#/…` und erklären die Umleitung.

---

## US-11 Nutzungskennzahlen anzeigen (Ergänzung)

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-11-08 | E37 nach Variante (a): `POST /simulations` mit `save: true` erfasst nur „Gespeicherte Simulationen“, ohne `save` nur „Durchgeführte Simulationen“; GraphQL `simulate` wie REST | Alternativszenario | automatisiert (`SimulationServiceTest`, `SimulationControllerTest`, `MutationControllerTest`; BE #128) | 1. Simulation ohne `save`<br>2. Simulation mit `save: true` und Namen<br>3. Mutation `simulate` mit `save: true` | 1. Genau ein Ereignis `SIMULATION_RUN`, kein `SIMULATION_SAVED`.<br>2. Genau ein Ereignis `SIMULATION_SAVED`, kein `SIMULATION_RUN`.<br>3. Wie 2. In jedem Fehlerfall kein Ereignis (wie AT-11-04).<br>Tests: `simulateForSaveRechnetGleichZaehltAberKeineDurchgefuehrteSimulation`, `simulateForSavePrueftEingabenWieSimulate` (`SimulationServiceTest`), `simulateWithSave_returns201WithIdNameAndCreatedAt` (`SimulationControllerTest`, ruft bei `save: true` nur `simulateForSave` auf), `simulateWithSaveCountsNoSecondSimulationRun` (`MutationControllerTest`). | offen |

Nach bestandenem Nachtest von AT-11-01, AT-11-03 und AT-11-06 und bestandenem AT-11-08 sind alle Testfälle von US-11 bestanden; US-11 (Docs #156) kann abgenommen werden.

---

## Nicht-funktionale Anforderungen

Die Adressen der drei Heroku-Apps stehen im Abnahmeprotokoll Sprint 5 (Abschnitt „Umgebung“). Für AT-NF13-01 ist Zugriff auf das Heroku-Dashboard nötig; ihn hat Carl Stibbe. Er zeigt die Angaben in einer Bildschirmfreigabe oder schickt Screenshots.

Ein Vortest zu AT-NF05-01 am 04.11.2026 ergab über `http://` die Antwort `200` statt einer Umleitung. Der Befund ist als B11 erfasst (BE #135, FE #81, Priorität hoch); AT-NF05-01 wird nach Merge und Deploy der Behebung durchgeführt.

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-NF05-01 | NF-05: nur über HTTPS erreichbar; Aufruf über `http://` wird auf `https://` umgeleitet; gültiges Zertifikat | NFR | manuell | In PowerShell nacheinander (`-I` zeigt nur die Kopfzeilen der Antwort):<br>1. `curl.exe -I http://lookupstockfrontend-96a9998805cb.herokuapp.com/`<br>2. `curl.exe -I http://lookupstock-da2951e42089.herokuapp.com/actuator/health`<br>3. `curl.exe -I http://lookupstock-marktdaten-12ca5b75c510.herokuapp.com/actuator/health`<br>4. Die drei Adressen mit `https://` im Browser öffnen und auf das Schloss-Symbol klicken | 1. bis 3. Antwort `301` oder `308` mit `Location: https://…`, **keine** Antwort `200` über `http://`.<br>4. Das Zertifikat ist gültig, der Browser zeigt keine Warnung. | offen |
| AT-NF13-01 | NF-13: Bereitstellung je Komponente ohne manuelle Zwischenschritte | NFR | manuell (Heroku-Dashboard, mit Carl Stibbe) | 1. Für die Apps des Frontends, `lookupstock` und `lookupstock-marktdaten` im Reiter „Deploy“ die Verbindung zum GitHub-Repository und den Branch ansehen<br>2. Im Reiter „Activity“ den letzten Build öffnen<br>3. Im Reiter „Settings“ unter „Config Vars“ nur die **Namen** der Variablen ansehen<br>4. Nach dem Deploy aus BE #130: `/actuator/health` beider Backend-Apps und die Startseite des Frontends aufrufen | 1. Jede App ist mit ihrem Repository verbunden und deployt von `main` (automatisch oder mit einem Klick auf „Deploy Branch“).<br>2. Der Build lief ohne manuelle Eingriffe durch; beim Start der Backend-Apps wendet Flyway die Migrationen selbst an.<br>3. Die Konfiguration steht nur in Konfigurationsvariablen.<br>4. Beide Dienste melden `UP`, die Startseite lädt, ohne weitere Schritte auf dem Server. | offen |

---

## Automatisierte Tests

| Testklasse | Repository | Teststufe | Abgedeckte Testfälle | Stand 04.11.2026 |
|---|---|---|---|---|
| `SimulationServiceTest`, `SimulationControllerTest`, `MutationControllerTest` | BE (core-service) | Unit- und Integrationstest | AT-11-08 | ergänzt (BE #128) |
| `simulation.test.js` | FE | Unit-Test (`node:test`) | Nachtest H8 (`leftMarginForLabels grows with the longest label`, `buildFanChartGeometry leaves room for eight-digit y labels in the comparison`) | ergänzt (FE #68) |
| `portfolio-errors.test.js` | FE | Unit-Test (`node:test`) | Nachtest H9 (vier Tests „… (FE #69, H9)“, u. a. `the empty editor shows a neutral weight sum status`) | ergänzt (FE #69) |
| `portfolio-save.test.js` | FE | Unit-Test (`node:test`) | Nachtest H12 (`formatPositionCount uses the singular for exactly one position`) | ergänzt (FE #71) |
| `dashboard-navigation.test.js` | FE | Unit-Test (`node:test`) | Nachtest B9 (`dashboard link is inserted and active for an authenticated manager`, `dashboard link is not inserted for regular or unauthenticated users`, `dashboard navigation keeps the SPA entry and has no static manager-only link`) | ergänzt (FE #67) |

Für H10 (FE #70) gibt es keinen eigenen automatisierten Test; der Nachtest erfolgt manuell in AT-11-01.
