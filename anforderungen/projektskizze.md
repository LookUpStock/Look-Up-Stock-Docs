# Projektskizze LookUpStock

| | |
|---|---|
| **Projekt** | LookUpStock – Monte-Carlo-Simulation und fairer Vergleich von Aktien, ETFs und Portfolios |
| **Kurs** | WWI25B, DHBW Mosbach, Fallstudie Software Engineering, Wintersemester 2026/27 |
| **Issue** | Docs #215 |
| **Stand** | 20.11.2026 (Abschlussfassung; Inhalte aus Sprint 1, ergänzt um den tatsächlichen Verlauf) |
| **Grundlagen** | Aufgabenbeschreibung Kapitel V (Projektskizze), Sprint-Planning-Protokoll Sprint 1, Product Backlog (Stand 02.10.2026, wiedergegeben in `change-request.md` Abschnitt 6), Portfolio Webprogrammierung Kapitel 1 |
| **KI-Nutzung** | Vision und Scrum-Begründung entstanden in Sprint 1 als eigene Entwürfe und wurden mit KI geprüft (KI-Log PKA-01, ME-01); diese Fassung wurde mit KI-Unterstützung aus den vorhandenen Dokumenten zusammengestellt und vom Team geprüft (siehe KI-Log) |

Diese Projektskizze fasst die drei Teile aus Kapitel V der Aufgabenbeschreibung zusammen: Vision (Abschnitt 1), Entscheidung für das Vorgehensmodell (Abschnitt 2) sowie erster Product Backlog und Sprintplanung (Abschnitte 3 und 4). Der Endstand des Product Backlogs steht in `product-backlog.md`.

---

## 1. Vision

### 1.1 Problem

Wer einen Sparplan auf einen ETF oder ein eigenes Portfolio anlegen möchte, sieht in Vergleichsportalen meist eine einzige Zahl: die Rendite der Vergangenheit. Diese Zahl verdeckt zwei Dinge, die für eine Anlageentscheidung wichtiger sind:

1. **Unsicherheit:** Die künftige Entwicklung ist unsicher. Ein ungünstiger Verlauf kann deutlich unter den eigenen Einzahlungen enden; eine Durchschnittsrendite zeigt das nicht.
2. **Unfaire Vergleiche:** Zwei Anlagen werden oft über unterschiedliche Zeiträume oder mit unterschiedlichen Annahmen betrachtet. Unterschiede im Ergebnis entstehen dann nicht nur aus der Anlage selbst.

Zielgruppe sind Privatanleger ohne tiefes Finanzwissen (Akteur „Registrierter Nutzer“, `akteure.md`).

### 1.2 Lösungsansatz

LookUpStock ist ein Web-Service, mit dem Nutzer Aktien und ETFs suchen, daraus gewichtete Portfolios zusammenstellen und deren mögliche Entwicklung simulieren:

- **Unsicherheit sichtbar machen:** Eine Monte-Carlo-Simulation schätzt aus der Kurshistorie Rendite und Schwankung jeder Anlage und rechnet daraus Tausende möglicher Verläufe. Das Ergebnis erscheint als Bandbreite (Fächerdiagramm mit 5., 50. und 95. Perzentil) und als Kennzahlen wie Median-Endwert, Value at Risk, maximaler Drawdown und Zielwahrscheinlichkeit.
- **Fair vergleichen:** Beim Vergleich zweier Anlagen oder Portfolios verwenden beide Seiten dieselben Zufallszahlen (Seed) und denselben Schätzzeitraum. Unterschiede entstehen so nur aus der Zusammensetzung.
- **Kursdaten von einem externen Anbieter:** Die Finanzdaten kommen über eine externe API (sekundärer Akteur „Finanzdaten-API“, gewählt wurde in BE #16 Twelve Data).
- **Rechtliche Abgrenzung:** LookUpStock ist ausdrücklich keine Anlageberatung (NF-14).

### 1.3 Abgrenzung

Nicht Teil des Projekts sind Handel und Depotverwaltung. Die Simulation berücksichtigt keine Kosten, Steuern, Inflation (E21) und Wechselkursschwankungen (E16). Das Simulationsmodell (normalverteilte Monatsrenditen je Anlage, keine Korrelation) ist bewusst einfach gehalten; seine Grenzen stehen in `simulationsmodell.md`.

### 1.4 Besonderheit des Projekts

Das Projekt wird zugleich in der Fallstudie Software Engineering und im Portfolio Webprogrammierung bewertet. Beide Anforderungsquellen fließen in denselben Product Backlog. Wie sich das auf die Architektur ausgewirkt hat, beschreiben ADR-07 und die Abschluss-Retrospektive (Sprint Review 6, Abschnitt 7).

---

## 2. Entscheidung für das Vorgehensmodell

### 2.1 Herleitung

Das Team hat ein schwergewichtiges Vorgehen (Wasserfallmodell) und ein agiles Vorgehen (Scrum) anhand der Rahmenbedingungen des Projekts verglichen:

| Rahmenbedingung | Wasserfall | Scrum |
|---|---|---|
| **Anforderungen sind zu Beginn unvollständig.** Der Kunde äußert nur eine Vorstellung des Systems; offene Fragen (z. B. Pfadanzahl, Länge der Kurshistorie, Seed als Eingabefeld) werden erst im Verlauf geklärt. | Verlangt vollständige Anforderungen vor dem Entwurf; jede spätere Klärung erzeugt einen Änderungsantrag. | Der Product Owner klärt Lücken laufend und hält sie als Festlegungen fest (E1 ff. in den Kurzformat-Tabellen). |
| **Ein Change Request kommt sicher**, etwa zur Hälfte der Bearbeitungszeit (Kapitel III.1). | Formaler Änderungsantrag und Anpassung des Projektplans. | Neue Einträge im Product Backlog, Neupriorisierung durch den Product Owner, Wirkung im nächsten Sprint Planning. |
| **Kurzer Zeitraum von rund fünf Wochen** bei gleichzeitigen Vorlesungen. | Ein lauffähiger Prototyp entsteht erst spät; Fehler in Analyse oder Entwurf zeigen sich erst bei der Umsetzung. | Nach jedem Sprint gibt es ein abnahmefähiges Inkrement; Fehler zeigen sich in der wöchentlichen Abnahme. |
| **Technisches Risiko:** Simulation und externe Kursdaten-API sind dem Team neu. | Risiko wird erst in der Implementierungsphase sichtbar. | Die riskanteste Story (US-04) und die API-Auswahl liegen in Sprint 1. |
| **Kunde ist im Team verfügbar.** Mert Eminel ist zugleich Kunde und Product Owner. | Kundenkontakt vor allem zu Beginn und bei der Abnahme. | Der Product Owner nimmt jede Woche ab und priorisiert neu. |
| **Diagramme sollen iterativ wachsen** (Kapitel V: Use-Case-Diagramm, Domänenmodell, Klassen- und ER-Diagramm je Sprint). | Diagramme entstehen einmal vollständig in der Entwurfsphase. | Diagramme werden je Sprint fortgeschrieben; ältere Stände liegen in `diagramme/archiv/`. |

### 2.2 Entscheidung und Begründung

Das Team hat sich für **Scrum** entschieden. Ausschlaggebend waren die unvollständigen Anforderungen, der sicher erwartete Change Request und das technische Risiko der Simulation: Scrum macht alle drei früh sichtbar und lässt den Plan wöchentlich anpassen, ohne einen formalen Änderungsprozess.

Gegen Wasserfall sprach vor allem, dass ein vollständiger Entwurf vor der ersten Zeile Code bei einer neuen Rechenlogik und einer noch nicht getesteten API zu viele Annahmen erfordert hätte.

### 2.3 Ausgestaltung

| Element | Festlegung |
|---|---|
| **Rollen** | Product Owner (zugleich Kunde): Mert Eminel. Scrum Master: Carl Stibbe. Entwickler: Paul Kaufmann, Peter Orban (zugleich Gruppensprecher). Die Projektleitung verteilt sich laut Aufgabenbeschreibung auf PO, SM und Entwickler. Unabhängig von der Rolle arbeitet jedes Mitglied an Analyse, Umsetzung und Dokumentation mit. |
| **Sprintlänge** | Eine Woche. Eine KI schlug zweiwöchige Sprints vor; das Team hat das verworfen, weil bei fünf Wochen Laufzeit sonst nur zwei Rückkopplungen mit dem Product Owner möglich gewesen wären (KI-Log PKA-01, ME-01). |
| **Ereignisse** | Sprint Planning mit Protokoll und Sprint Backlog, Umsetzung, Abnahme durch den Product Owner anhand der Abnahmetestfälle, Sprint Review mit Retrospektive. Alle Protokolle liegen in `sprints/sprint-<n>/`. |
| **Artefakte** | Product Backlog mit MoSCoW-Priorität und Story Points, Sprint Backlog als GitHub Project, Definition of Done (Sprint-Planning-Protokoll Sprint 1, Abschnitt 6, fortgeschrieben in `CONTRIBUTING.md`). |
| **Werkzeuge** | GitHub Issues und GitHub Projects, drei Repositories (Docs, BE, FE), Pull Requests mit Review, GitHub Actions als CI, PlantUML für Diagramme, Figma für Wireframes. |
| **Kern-Stories** | US-04 (Simulation) und US-07 (Vergleich), mit dem Dozenten abgestimmt (Docs #27). Für sie entstehen Aktivitäts- und Sequenzdiagramme. |
| **Puffer für den Change Request** | In Sprint 4 wurde Kapazität für die Analyse der Ereigniskarte reserviert (Sprint-Planning-Protokoll Sprint 4, Abschnitt 2). |

### 2.4 Anwendung im Projektverlauf

Das Team hat Scrum über fünf einwöchige Sprints und einen kurzen sechsten Sprint angewendet. Der sechste Sprint war nicht geplant; er wurde eingeschoben, weil die Abnahme von Sprint 5 zwei Fehler hoher Priorität fand (B8, B9). Der Change Request CR-01 wurde ohne formalen Änderungsantrag als neue Stories US-11 bis US-13 aufgenommen (`change-request.md`). Ob Scrum für das Projekt geeignet war, bewertet die Abschluss-Retrospektive in Sprint Review 6.

---

## 3. Erster Product Backlog (Stand 02.10.2026)

Grundlage für die Sprintplanung war der folgende Product Backlog. Priorisiert wurde nach MoSCoW durch den Product Owner; geschätzt wurde in Story Points. Die Begründung der Priorität je Story und der Endstand stehen in `product-backlog.md`.

| ID | User Story | Prio | SP | geplanter Sprint |
|---|---|---|---|---|
| US-01 | Als Nutzer möchte ich mich registrieren und anmelden, um meine Portfolios zu speichern. | Must | 3 | 2 |
| US-02 | Als Nutzer möchte ich Anlagen per Ticker, ISIN oder Name suchen, um sie meinem Portfolio hinzuzufügen. | Must | 2 | 1 |
| US-03 | Als Nutzer möchte ich ein Portfolio aus mehreren Anlagen mit Gewichtungen erstellen können. | Must | 5 | 1 |
| US-04 | Als Nutzer möchte ich für ein Portfolio Startkapital, Sparrate und Anlagehorizont eingeben und eine Monte-Carlo-Simulation starten, um die Bandbreite möglicher Verläufe zu sehen. **(Kern-Story)** | Must | 8 | 1 |
| US-05 | Als Nutzer möchte ich Kennzahlen wie Median-Endwert, Value at Risk, max. Drawdown und Zielwahrscheinlichkeit sehen, um das Risiko einzuschätzen. | Must | 2 | 3 |
| US-06 | Als Nutzer möchte ich Portfolios bearbeiten und löschen. | Should | 2 | 3 |
| US-07 | Als Nutzer möchte ich zwei Anlagen oder Portfolios unter gleichen Annahmen vergleichen, um mich zwischen ihnen zu entscheiden. **(Kern-Story)** | Must | 5 | 3 |
| US-08 | Als Nutzer möchte ich Simulationen und Vergleiche speichern und später wieder aufrufen. | Should | 3 | 3 |
| US-09 | Als Gast möchte ich ohne Konto eine Beispielsimulation ausprobieren. | Could | 1 | 4 |
| US-10 | Als Nutzer möchte ich mehr als zwei Portfolios gleichzeitig vergleichen. | [PRÜFEN: ursprüngliche Priorität] | 8 | 4 |

Die nicht-funktionalen Anforderungen NF-01 bis NF-16 stehen nach FURPS+ mit messbaren Abnahmekriterien in `nfr_furps.md`. NF-01 (Simulation in höchstens 3 Sekunden) wurde wegen seines Aufwands als eigener Backlog-Eintrag mit 3 SP geführt.

---

## 4. Sprintplanung

### 4.1 Projektplan

| Sprint | Zeitraum | Geplanter Inhalt (Stand 02.10.2026) | Meilensteine |
|---|---|---|---|
| 1 | 28.09.–04.10. | Setup (Repositories, Projektgerüste, CI, API-Auswahl, Server), US-02, US-03, US-04, Analyse- und Design-Artefakte, ADR-01 bis ADR-03, Testkonzept | Kern-Stories mit dem Dozenten abgestimmt |
| 2 | 05.10.–11.10. | US-01, Übertrag aus Sprint 1 | Deployment auf dem Server |
| 3 | 12.10.–18.10. | US-05, US-06, US-07, US-08, NF-01 | Kern-Story US-07 |
| 4 | 19.10.–25.10. | US-09, US-10, Puffer für den Change Request | Zwischenpräsentation mit Ereigniskarte (23.10.) |
| 5 | 26.10.–01.11. | Abschluss und Anforderungen der Webprogrammierung; keine neuen Stories außer Ausnahmen (Sprint-Planning-Protokoll Sprint 4) | |
| – | 02.11.–22.11. | Portfolio Webprogrammierung, Präsentationen, Projektdokumentation | Abgabe Portfolio 08.11., Präsentationen 06.11. und 09.11., Abgabe Fallstudie 22.11. |

Sprint 1 enthält neben drei User Stories den einmaligen Setup-Aufwand; das Sprint Backlog mit 64,5 geplanten Stunden, Verantwortlichen und Abhängigkeiten steht im Sprint-Planning-Protokoll Sprint 1.

### 4.2 Abweichungen vom Plan

| Plan | Tatsächlich | Grund |
|---|---|---|
| Eigener Server mit Docker Compose | Heroku mit drei Apps und Heroku Postgres (ab Sprint 2) | Geringerer Betriebsaufwand, Deployment direkt aus GitHub |
| US-02 bis US-04 in Sprint 1 abgenommen | Abnahme erst in Sprint 3 (Nachtest) | Umfang des ersten Sprints mit Setup unterschätzt |
| US-05 und US-07 in Sprint 3 | Abnahme in Sprint 4 | Befunde aus der Abnahme Sprint 3 |
| US-10 in Sprint 4 | Won't | Vorrang für Befunde, Change Request und Webprogrammierung (Sprint-Planning-Protokoll Sprint 4) |
| „Vergleiche speichern“ (Teil von US-08) | Won't | Kapazität für CR-01 (E43) |
| Ein Backend in drei Schichten (ADR-01) | Zwei Dienste mit Queue (ADR-07, Sprint 5) | Guidelines der Webprogrammierung verlangen eine nicht monolithische Architektur |
| Fünf Sprints | Sechster, kurzer Sprint 03.11.–09.11. | Fehler B8 und B9 aus der Abnahme Sprint 5 |

---

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 20.11.2026 | Erstfassung als eigenes Dokument; Vision und Scrum-Begründung aus Sprint 1 zusammengeführt, um Projektplan und Abweichungen ergänzt | Docs #215 |
