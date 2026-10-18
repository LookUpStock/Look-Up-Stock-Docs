# Akteursbeschreibung

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #57 Akteurbeschreibung und Use-Case-Diagramm v2 (Version 1: Docs #15); Docs #98 Verweis auf Use-Case-Diagramm v3 (Version 3) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 3 |
| **Stand** | 18.10.2026 |
| **Grundlagen** | Vision und Product Backlog (US-01 bis US-10), Kurzformat-Tabellen Sprint 1 und Sprint 2, Domänenmodell v3, API-Design (Stand 02.10.2026), NFRs Version 2 |
| **KI-Nutzung** | Mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Die Tabelle folgt der Vorlage aus der Vorlesung (Folie 19) und ist um die Art des Akteurs und die zugehörigen Anwendungsfälle ergänzt. Primäre Akteure lösen Anwendungsfälle selbst aus, sekundäre Akteure werden vom System für einen Anwendungsfall benötigt. Das zugehörige Diagramm liegt in `diagramme/use-case-diagramm_v3.puml`.

| Akteur | Art | Beschreibung | Anwendungsfälle |
|---|---|---|---|
| **Gast** | primär | Person ohne Nutzerkonto oder ohne gültige Anmeldung, die LookUpStock kennenlernen möchte. Der Gast kann nach Anlagen suchen, eine Beispielsimulation mit vorgegebenen Beispielportfolios ausprobieren und sich mit E-Mail-Adresse und Passwort registrieren. Eigene Portfolios kann er nicht anlegen und nichts speichern. Ruft er eine geschützte Funktion auf, wird er zur Anmeldung geführt. Nach der Registrierung und Anmeldung wird er zum registrierten Nutzer. | US-01 Registrieren, US-02 Anlagen suchen, US-09 Beispielsimulation ausprobieren |
| **Registrierter Nutzer** | primär | Privatanleger ohne tiefes Finanzwissen mit Nutzerkonto. Er möchte verstehen, wie sich seine Geldanlage unter Unsicherheit entwickeln kann, um eine fundierte Anlageentscheidung zu treffen. Er meldet sich mit E-Mail-Adresse und Passwort an; die Anmeldung gilt 24 Stunden oder bis er sich abmeldet (NF-16). Angemeldet sucht er Anlagen, stellt Portfolios zusammen, simuliert sie, vergleicht sie und speichert Ergebnisse. Er sieht und ändert ausschließlich seine eigenen Daten (NF-04). | US-01 An- und abmelden, US-02 bis US-08, US-10 |
| **Finanzdaten-API (Twelve Data)** | sekundär | Externer Dienst, der Stammdaten zu Aktien und ETFs (Name, Ticker, ISIN, Börse, Währung) und historische Kursdaten als Monatswerte liefert. Nur das Backend greift darauf zu, der API-Key bleibt auf dem Server (NF-06). Ist der Dienst nicht erreichbar, verwendet LookUpStock Kursdaten aus dem Cache, die jünger als 24 Stunden sind (NF-07). | US-02 Anlagen suchen, US-04 Simulation durchführen, US-07 Vergleichen, US-09 Beispielsimulation |

## Öffentliche und geschützte Anwendungsfälle (ab Sprint 2)

Mit US-01 ist die Anmeldung umgesetzt. Damit gilt:

| Bereich | Anwendungsfälle | Akteur |
|---|---|---|
| **Öffentlich** (ohne Anmeldung) | US-01 Registrieren, US-01 Anmelden, US-02 Anlagen suchen, US-09 Beispielsimulation | Gast; „Anlagen suchen“ auch registrierter Nutzer |
| **Geschützt** (Vorbedingung: angemeldet) | US-01 Abmelden, US-03 bis US-08, US-10 | Registrierter Nutzer |

Die Anmeldung ist im Use-Case-Diagramm bewusst **kein** `<<include>>` der geschützten Anwendungsfälle, sondern deren Vorbedingung: Sie wird einmal durchgeführt und gilt dann für mehrere Anwendungsfälle. In den Kurzformat-Tabellen steht sie deshalb im Feld „Vorbedingungen“ (siehe `kurzformat_sprint-2.md`, Abschnitt „Auswirkungen von US-01 auf US-02 bis US-04“).

Im Domänenmodell (v3) entspricht dem registrierten Nutzer der Fachbegriff **Nutzer**, der Portfolios besitzt und Anmeldungen hat. Der Gast ist kein eigener Fachbegriff, da über ihn nichts gespeichert wird.

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 04.10.2026 | Erstfassung mit Gast, registriertem Nutzer und Finanzdaten-API | Docs #15 |
| 2 | 08.10.2026 | Gast: Registrierung mit E-Mail und Passwort, Weiterleitung zur Anmeldung bei geschützten Funktionen. Registrierter Nutzer: Anmeldung mit E-Mail und Passwort, 24 Stunden gültig (NF-16). Neuer Abschnitt „Öffentliche und geschützte Anwendungsfälle“. Hinweis zu Sprint 1 entfernt, da US-01 umgesetzt ist. Verweis auf Use-Case-Diagramm v2 und Domänenmodell v3 | Docs #57, US-01 in Sprint 2 |
| 3 | 18.10.2026 | Verweis auf das Use-Case-Diagramm v3 aktualisiert (v2 liegt jetzt in `diagramme/archiv/`); Akteure und Anwendungsfälle unverändert | Docs #98 |
