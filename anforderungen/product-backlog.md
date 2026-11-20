# Product Backlog – Endstand

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #215 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 05.11.2026 (nach der Abnahme Sprint 6) |
| **Grundlagen** | Product Backlog vom 02.10.2026 (`projektskizze.md`, Abschnitt 3), Product Backlog vorher und nachher CR-01 (`change-request.md`, Abschnitte 6 und 7), Sprint Reviews 1 bis 6, Abnahmeprotokolle Sprint 2 bis 6 |
| **KI-Nutzung** | Aus den genannten Dokumenten mit KI-Unterstützung zusammengestellt, vom Product Owner geprüft (siehe KI-Log) |

## 1. Priorisierung

Der Product Owner priorisiert nach **MoSCoW**:

| Stufe | Bedeutung im Projekt |
|---|---|
| **Must** | Ohne diese Story erfüllt LookUpStock seinen Zweck nicht (Unsicherheit sichtbar machen, fair vergleichen) oder eine andere Must-Story ist nicht nutzbar. |
| **Should** | Wichtig für eine angenehme Nutzung; es gibt aber eine Umgehung, und der Kernnutzen bleibt ohne sie erhalten. |
| **Could** | Zusätzlicher Nutzen, wird nur bei freier Kapazität umgesetzt (Stretch). |
| **Won't** | Bewusst nicht umgesetzt; die Entscheidung ist begründet und dokumentiert. |

Innerhalb einer Stufe gilt: Stories, von denen andere abhängen, und Stories mit hohem technischem Risiko kommen zuerst. Deshalb lagen Suche, Portfolio und Simulation in Sprint 1, obwohl die Anmeldung (US-01) ebenfalls Must ist.

## 2. User Stories

| ID | User Story | Prio | SP | Begründung der Priorität | Sprint | Endstand |
|---|---|---|---|---|---|---|
| US-01 | Als Nutzer möchte ich mich registrieren und anmelden, um meine Portfolios zu speichern. | Must | 3 | Portfolios und gespeicherte Simulationen gehören einem Nutzer; ohne Anmeldung kein Zugriffsschutz (NF-04). Ein KI-Vorschlag stufte US-01 niedriger ein, der PO hat Must beibehalten (KI-Log ME-02). Zeitlich nach US-02 bis US-04, weil diese das größere Risiko trugen. | 2 | abgenommen (Sprint 2) |
| US-02 | Als Nutzer möchte ich Anlagen per Ticker, ISIN oder Name suchen, um sie meinem Portfolio hinzuzufügen. | Must | 2 | Voraussetzung für US-03; zugleich der erste Test der externen Finanzdaten-API. | 1 | abgenommen (Sprint 3, Nachtest) |
| US-03 | Als Nutzer möchte ich ein Portfolio aus mehreren Anlagen mit Gewichtungen erstellen können. | Must | 5 | Voraussetzung für Simulation und Vergleich von Portfolios. | 1 | abgenommen (Sprint 3, Nachtest) |
| US-04 | Als Nutzer möchte ich für ein Portfolio Startkapital, Sparrate und Anlagehorizont eingeben und eine Monte-Carlo-Simulation starten, um die Bandbreite möglicher Verläufe zu sehen. **Kern-Story** | Must | 8 | Kernnutzen des Produkts und größtes technisches Risiko; deshalb in Sprint 1. | 1 | abgenommen (Sprint 3, Nachtest) |
| US-05 | Als Nutzer möchte ich Kennzahlen wie Median-Endwert, Value at Risk, max. Drawdown und Zielwahrscheinlichkeit sehen, um das Risiko einzuschätzen. | Must | 2 | Das Fächerdiagramm allein lässt Risiko nicht in Zahlen ablesen; erst die Kennzahlen machen die Unsicherheit für Fachfremde greifbar. | 3, 4 | abgenommen (Sprint 4, 25.10.2026) |
| US-06 | Als Nutzer möchte ich Portfolios bearbeiten und löschen. | Should | 2 | Komfort: Ohne die Funktion lässt sich ein neues Portfolio anlegen; der Kernnutzen bleibt. | 3 | abgenommen (Sprint 3) |
| US-07 | Als Nutzer möchte ich zwei Anlagen oder Portfolios unter gleichen Annahmen vergleichen, um mich zwischen ihnen zu entscheiden. **Kern-Story** | Must | 5 | Zweiter Teil des Produktversprechens (fairer Vergleich); mit dem Dozenten als Kern-Story abgestimmt. | 3, 4 | abgenommen (Sprint 4, 25.10.2026) |
| US-08 | Als Nutzer möchte ich Simulationen speichern und später wieder aufrufen. | Should | 3 | Komfort: Eine Simulation lässt sich mit gleichem Seed jederzeit wiederholen (NF-08). Der Teil „Vergleiche speichern“ wurde zugunsten von CR-01 auf Won't gesetzt (E43). | 3 | Simulationen speichern abgenommen (Sprint 3); Vergleiche speichern Won't |
| US-09 | Als Gast möchte ich ohne Konto eine Beispielsimulation ausprobieren. | Could | 1 | Erleichtert den Einstieg, ist für registrierte Nutzer aber ohne Nutzen; deshalb nur als Stretch. | 4 | abgenommen (Sprint 4) |
| US-10 | Als Nutzer möchte ich mehr als zwei Portfolios gleichzeitig vergleichen. | Won't | 8 | Hoher Aufwand bei geringem Zusatznutzen gegenüber US-07; Vorrang hatten die Befunde aus Sprint 3, der Change Request und die Webprogrammierung (Sprint-Planning-Protokoll Sprint 4). | – | nicht umgesetzt |
| US-11 | Als Geschäftsführung möchte ich monatliche Kennzahlen zur Nutzung von LookUpStock in einem Dashboard sehen, um die Entwicklung des Produkts beurteilen zu können. | Must | 5 | Kern des Change Requests CR-01; die Erfassung musste früh beginnen, weil Nutzungsdaten nicht nachträglich entstehen. | 5, 6 | abgenommen (Sprint 6, 05.11.2026) |
| US-12 | Als Geschäftsführung möchte ich die Kennzahlen eines Monats als Datei exportieren, um sie weiterzuverarbeiten und weiterzugeben. | Should | 2 | „Exportierbare Datei“ aus CR-01; baut auf US-11 auf, die Werte sind auch ohne Export im Dashboard sichtbar. | 5 | abgenommen (Sprint 5) |
| US-13 | Als Geschäftsführung möchte ich den Monatsbericht automatisch zugeschickt bekommen. | Won't | 3 | Wenig Mehrwert gegenüber dem Abruf auf Knopfdruck, erfordert aber einen zusätzlichen externen Dienst (E42, CR-01 Option B). | – | nicht umgesetzt |

## 3. Weitere Einträge

| ID | Eintrag | Prio | Endstand |
|---|---|---|---|
| NF-01 | Eine Simulation mit 10.000 Pfaden und 5 Anlagen dauert höchstens 3 Sekunden. Als eigener Eintrag mit 3 SP geführt, weil der Nachweis eine eigene Messung erfordert. | Must | nachgewiesen (Sprint 3, AT-04-14; erneut Sprint 5 und 6: 20 von 20 Läufen innerhalb der Zielzeit) |
| H14 | Vergleichsdiagramm bei stark unterschiedlichen Endwerten schwer lesbar (gemeinsame Y-Achse). Mögliche Lösung: Umschalter oder logarithmische Achse. | Could | offen; aus der Abnahme Sprint 6 übernommen, vor der Abgabe bewusst nicht umgesetzt |

Alle nicht-funktionalen Anforderungen NF-01 bis NF-16 mit messbaren Abnahmekriterien und Verweis auf die Testfälle stehen in `nfr_furps.md`.

## 4. Zusammenfassung

| | Anzahl | Story Points |
|---|---|---|
| User Stories im Backlog | 13 | 49 |
| davon umgesetzt und abgenommen | 11 (US-01 bis US-09, US-11, US-12) | 38 |
| davon Won't | 2 (US-10, US-13) sowie der Teil „Vergleiche speichern“ von US-08 | 11 |
| offene Einträge mit niedriger Priorität | 1 (H14) | – |

## 5. Veränderung im Projektverlauf

| Datum | Änderung | Quelle |
|---|---|---|
| 02.10.2026 | Erster Product Backlog mit US-01 bis US-10 | `projektskizze.md`, Abschnitt 3 |
| 19.10.2026 | US-10 auf Won't | Sprint-Planning-Protokoll Sprint 4 |
| 25.10.2026 | CR-01: US-11 (Must), US-12 (Should), US-13 (Won't) neu; „Vergleiche speichern“ (US-08) auf Won't | `change-request.md`, Abschnitte 6 und 7 |
| 05.11.2026 | H14 mit niedriger Priorität aufgenommen | Sprint Review 6, Abschnitt 6 |

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 20.11.2026 | Erstfassung des Endstands mit Begründung der Priorisierung | Docs #215 |
