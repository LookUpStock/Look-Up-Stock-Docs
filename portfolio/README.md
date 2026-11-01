# Portfolio Webprogrammierung – Aufbau und Zuständigkeiten

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #161 Portfolio Webprogrammierung (Sammel-Issue); dieses Dokument mit Docs #174 |
| **Abgabe** | 08.11.2026, 20:00 Uhr, per E-Mail mit Link zum Projekt-Code (Portfolio-Guidelines); Präsentation am 09.11.2026 |
| **Stand** | 01.11.2026 |

Das Portfolio folgt dem Strukturvorschlag der Portfolio-Guidelines. Jedes Kapitel ist eine eigene Markdown-Datei in diesem Ordner, damit jede Person ihr Kapitel über einen eigenen Pull Request ergänzen kann. Abbildungen liegen in `abbildungen/` (PlantUML-Quelle und gerendertes Bild, wie bei den übrigen Diagrammen). Zur Abgabe werden die Kapitel in der Reihenfolge der Tabelle zu einer PDF-Datei zusammengeführt.

| Datei | Kapitel | Umfang laut Guidelines | Issue | Verantwortlich |
|---|---|---|---|---|
| `00-titel-und-abstract.md` | Titel, Thema; Abstract deutsch und englisch | 1 + 1 Seite | Docs #174 | KujoBaraka |
| `01-einfuehrung.md` | Einführung in das Projekt | 2 bis 4 Seiten | Docs #174 | KujoBaraka |
| `02-api-design.md` | API-Design | 1 bis 3 Seiten | Docs #174 | KujoBaraka |
| `03-implementierung-architektur.md` | Implementierung: Architektur, getrennte Dienste, GraphQL | zusammen 5 bis 10 Seiten | Docs #175 | Paul02112004 |
| `04-implementierung-integration.md` | Implementierung: Integration Twelve Data, Queue, Caches | | Docs #176 | Pixxels02, KujoBaraka |
| `05-implementierung-mpa-spa.md` | Implementierung: MPA und SPA | | Docs #177 | carlstibbe |
| `06-reflexion-<name>.md` | Wissenschaftliche Reflexion, je Person eine Datei | je 1 bis 2 Seiten | Docs #178 | alle |
| `07-literatur.md` | Gemeinsames Literaturverzeichnis | – | Docs #174 | alle ergänzen |

**Gliederung der Implementierung.** Die Guidelines verlangen, sich für eine Gliederung zu entscheiden. Die Aufteilung der Issues im Sprint-Planning-Protokoll Sprint 5 gliedert die Implementierung **nach Komponenten** (Architektur und Dienste, Integration und Infrastruktur, Touchpoints). Innerhalb jedes Kapitels wird der zeitliche Verlauf über die Sprints beschrieben.

**Leitlinien aus den Guidelines für alle Kapitel**

- Das Portfolio beschreibt das Lernen, die Erkenntnisse, die Fehler und die Entscheidungen und bewertet, ob die Entscheidungen richtig waren. Es ist keine Produktbeschreibung.
- Bewertet werden Wissenschaftlichkeit und Objektivität, dokumentierte Lernfortschritte und Rückschläge sowie die Reflexion (20 Punkte).
- Die Implementierung wird „deutlich, detailliert, bis hin zu Code-Abschnitten“ erläutert.
- Grundlagen sind die ADRs, Abnahmeprotokolle, Sprint-Planning-Protokolle, Retrospektiven und KI-Logs im Repository; Verweise darauf mit Dateiname bzw. Issue-Nummer.
- Quellen werden im Text als (Autor Jahr) bzw. (Kurzname) zitiert und in `07-literatur.md` aufgeführt.

**Abgabe-E-Mail** (Betreff „Portfolio Webprogrammierung“, Inhalt als JSON laut Guidelines, keine Anhänge): verschickt der Gruppensprecher; als `linkToProjectCode` dienen die Links aus `00-titel-und-abstract.md`.
