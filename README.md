# LookUpStock – Dokumentation

LookUpStock ist ein Web-Service, mit dem Privatanleger Aktien und ETFs suchen, daraus gewichtete Portfolios zusammenstellen und deren mögliche Entwicklung mit einer Monte-Carlo-Simulation als Bandbreite (Fächerdiagramm) darstellen. Zwei Anlagen oder Portfolios lassen sich unter identischen Annahmen vergleichen. Für die Geschäftsführung gibt es ein Dashboard mit monatlichen Nutzungskennzahlen und einen CSV-Export (Change Request CR-01).

Das Projekt ist die Fallstudie Software Engineering und zugleich das Portfolio Webprogrammierung im Kurs WWI25B der DHBW Mosbach, Wintersemester 2026/27. Dieses Repository enthält die gesamte Projektdokumentation; der Code liegt in zwei weiteren Repositories.

## Links

| | Link |
|---|---|
| Web-Service (Frontend, MPA und SPA) | https://lookupstockfrontend-96a9998805cb.herokuapp.com |
| Frontend-Repository | https://github.com/LookUpStock/Look-Up-Stock-FrontEnd |
| Backend-Repository (core-service, marktdaten-service) | https://github.com/LookUpStock/Look-Up-Stock-BackEnd |
| Dokumentations-Repository (dieses) | https://github.com/LookUpStock/Look-Up-Stock-Docs |

## Team

| Name | Rolle | GitHub |
|---|---|---|
| Mert Eminel | Product Owner | KujoBaraka |
| Paul Kaufmann | Entwickler | Paul02112004 |
| Peter Orban | Entwickler, Gruppensprecher | Pixxels02 |
| Carl Stibbe | Scrum Master | carlstibbe |

Vorgehen: Scrum mit einwöchigen Sprints (Sprint 1 bis 5 vom 28.09. bis 01.11.2026, danach ein kurzer Sprint 6 für die Befunde der Abnahme Sprint 5). Wer was gemacht hat, zeigen die Issues, die Pull Requests und die Commit-Historie; die Datei `.mailmap` fasst die verschiedenen Git-Namen je Person zusammen (`git shortlog -sn`).

## Ordner

| Ordner / Datei | Inhalt |
|---|---|
| `anforderungen/` | Kurzformat-Tabellen der User Stories mit Akzeptanzkriterien und PO-Festlegungen (`kurzformat_sprint-1.md` bis `kurzformat_sprint-4.md`), Akteure, Glossar, nicht-funktionale Anforderungen nach FURPS+ (`nfr_furps.md`), Simulationsmodell, Change Request CR-01 (`change-request.md`) |
| `diagramme/` | aktueller Stand von Use-Case-Diagramm, Domänenmodell, Klassen- und ER-Diagramm sowie Aktivitäts- und Sequenzdiagrammen der Kern-Stories US-04 und US-07, jeweils als PlantUML-Quelle (`.puml`) und Bild |
| `diagramme/archiv/` | ältere Versionen aller Diagramme; zeigen das iterative Wachstum und den Stand vor und nach dem Change Request |
| `adr/` | Architecture Decision Records ADR-01 bis ADR-08 |
| `api/` | API-Design: REST und GraphQL zwischen Frontend und Backend, interne Schnittstellen zwischen den Diensten (`api-design.md`) |
| `testkonzept/` | Testkonzept, Abnahmetestfälle und Abnahmeprotokolle je Sprint, Testabdeckung, Bedienbarkeitstest NF-09 |
| `sprints/` | je Sprint ein Ordner `sprint-<n>/` mit Sprint-Planning-Protokoll und Sprint Review (mit Retrospektive) |
| `ki-log/` | KI-Log je Teammitglied (`KI-Log-<Vorname>-<Nachname>.docx`) nach der Vorlage aus der Aufgabenstellung |
| `portfolio/` | Portfolio Webprogrammierung, ein Kapitel je Datei (Aufbau in `portfolio/README.md`) |
| `Wireframe` | Links zu den Wireframes in Figma |
| `CONTRIBUTING.md` | Arbeitsweise im Team: Branches, Pull Requests, Definition of Done, KI-Nutzung |

## Einstieg für Leser

1. Anforderungen: `anforderungen/kurzformat_sprint-1.md` bis `kurzformat_sprint-4.md`, danach `nfr_furps.md` und `glossar.md`
2. Kern-Stories US-04 (Simulation) und US-07 (Vergleich): Aktivitäts- und Sequenzdiagramme v3 in `diagramme/`
3. Architektur: `adr/` und `diagramme/klassendiagramm_v7.png`, `diagramme/er-diagramm_v6.png`
4. Qualität: `testkonzept/testkonzept.md`, dann die Abnahmeprotokolle je Sprint
5. Change Request: `anforderungen/change-request.md`

## Werkzeuge

| Zweck | Werkzeug |
|---|---|
| Diagramme (UML, Domänenmodell, ER) | PlantUML; die Quellen liegen als `.puml` neben den Bildern. Die Aufgabenbeschreibung empfiehlt Visual Paradigm Online und erlaubt ein selbst gewähltes Werkzeug, wenn es genannt wird. Das Team hat PlantUML gewählt, weil die Diagramme so wie Code versioniert und im Pull Request geprüft werden können. |
| Wireframes | Figma |
| Projektmanagement | GitHub Issues und GitHub Projects; welche Issues zu welchem Sprint gehören, steht im Sprint-Planning-Protokoll (Milestones nur in Sprint 1 und 2) |
| Versionsverwaltung und Review | Git, GitHub mit Pull Requests; `main` ist in allen drei Repositories geschützt (Pull Request mit Freigabe) |
| CI | GitHub Actions: Tests bei jedem Pull Request, im Backend mit Abdeckungsbericht (JaCoCo) |
| Betrieb | Heroku: drei Apps (Frontend, core-service, marktdaten-service) mit Heroku Postgres |
| Finanzdaten | Twelve Data (kostenloser Tarif) |
| KI-Unterstützung | siehe KI-Log der Teammitglieder; Regeln in `CONTRIBUTING.md` |

## Hinweis zu den ADRs

Die Aufgabenbeschreibung nennt 3 bis maximal 5 ADRs. Das Repository enthält 8: ADR-04 ist durch ADR-06 teilweise abgelöst, ADR-01 durch ADR-07; ADR-07 (getrennte Dienste) und ADR-08 (SPA-Touchpoint) kamen für die Webprogrammierung hinzu. Der Dozent hat bestätigt, dass mehr als 5 ADRs in Ordnung sind.
