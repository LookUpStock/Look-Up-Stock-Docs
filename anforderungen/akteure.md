# Akteursbeschreibung

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #15 Akteure, Use-Case-Diagramm v1, Glossar |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 1 |
| **Stand** | 04.10.2026 |
| **Grundlagen** | Vision und Product Backlog (US-01 bis US-10), Kurzformat-Tabellen v2, API-Design (Stand 02.10.2026), NFRs Version 2 |
| **KI-Nutzung** | Mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Die Tabelle folgt der Vorlage aus der Vorlesung (Folie 19) und ist um die Art des Akteurs und die zugehörigen Anwendungsfälle ergänzt. Primäre Akteure lösen Anwendungsfälle selbst aus, sekundäre Akteure werden vom System für einen Anwendungsfall benötigt. Das zugehörige Diagramm liegt in `diagramme/use-case-diagramm_v1.puml`.

| Akteur | Art | Beschreibung | Anwendungsfälle |
|---|---|---|---|
| **Gast** | primär | Person ohne Nutzerkonto oder ohne Anmeldung, die LookUpStock kennenlernen möchte. Der Gast kann nach Anlagen suchen, eine Beispielsimulation mit vorgegebenen Beispielportfolios ausprobieren und sich registrieren. Eigene Portfolios kann er nicht anlegen und nichts speichern. | US-01 Registrieren, US-02 Anlagen suchen, US-09 Beispielsimulation ausprobieren |
| **Registrierter Nutzer** | primär | Privatanleger ohne tiefes Finanzwissen mit Nutzerkonto. Er möchte verstehen, wie sich seine Geldanlage unter Unsicherheit entwickeln kann, um eine fundierte Anlageentscheidung zu treffen. Er sucht Anlagen, stellt Portfolios zusammen, simuliert sie, vergleicht sie und speichert Ergebnisse. Er sieht und ändert ausschließlich seine eigenen Daten (NF-04). | US-01 An- und abmelden, US-02 bis US-08, US-10 |
| **Finanzdaten-API (Twelve Data)** | sekundär | Externer Dienst, der Stammdaten zu Aktien und ETFs (Name, Ticker, ISIN, Börse, Währung) und historische Kursdaten als Monatswerte liefert. Nur das Backend greift darauf zu, der API-Key bleibt auf dem Server (NF-06). Ist der Dienst nicht erreichbar, verwendet LookUpStock Kursdaten aus dem Cache, die jünger als 24 Stunden sind (NF-07). | US-02 Anlagen suchen, US-04 Simulation durchführen, US-07 Vergleichen, US-09 Beispielsimulation |

**Hinweis zu Sprint 1:** Die Anmeldung (US-01) wird erst in Sprint 2 umgesetzt. Bis dahin sind die Sprint-1-Stories US-02 bis US-04 ohne Anmeldung nutzbar; in den Kurzformat-Tabellen heißt der Akteur deshalb vorerst „Nutzer“. Mit US-01 wird daraus der registrierte Nutzer, und Portfolios werden seinem Konto zugeordnet.
