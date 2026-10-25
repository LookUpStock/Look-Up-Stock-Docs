# Kurzformat-Tabellen – Sprint 4

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 4 (19.10.2026 – 25.10.2026), Umsetzung in Sprint 5 (26.10.2026 – 01.11.2026) |
| **Issue** | Docs #131 Change Request: Auswirkungsanalyse, Entscheidung, Backlog vorher und nachher, Kurzformat und Akzeptanzkriterien |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 1 |
| **Stand** | 25.10.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Change Request CR-01 (`change-request.md`), Ereigniskarte Nr. 8 „Reporting für die Geschäftsführung“ (erhalten am 23.10.2026), Kurzformat-Tabellen Sprint 1 bis Sprint 3 (E1 bis E35), NFRs Version 4 (NF-04, NF-14 b), ADR-06, ER-Diagramm v4 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt `kurzformat_sprint-3.md` fort. Sie enthält die neuen Stories aus dem Change Request CR-01: US-11 Nutzungskennzahlen anzeigen und US-12 Nutzungskennzahlen exportieren. US-13 (automatischer Monatsbericht) ist Won't und erhält keine Tabelle. Analyse, Optionen und Entscheidung stehen in `change-request.md`.

Beide Stories sind dem Akteur **Geschäftsführung** vorbehalten (Akteursbeschreibung v4). Ohne gültige Anmeldung antwortet das Backend mit 401 und die Oberfläche führt zur Anmeldung (E14); angemeldete Nutzer ohne die Rolle Geschäftsführung erhalten 403 (E40).

---

## US-11 Nutzungskennzahlen anzeigen

| **Name** | **Nutzungskennzahlen anzeigen** |
|---|---|
| **Art** | User Story (US-11), aus Change Request CR-01 |
| **Kurzbeschreibung** | Die Geschäftsführung sieht für einen gewählten Kalendermonat, wie oft die Funktionen von LookUpStock erfolgreich genutzt wurden, um die Entwicklung des Produkts beurteilen zu können. |
| **Auslöser** | Die Geschäftsführung möchte die Nutzung eines Monats auswerten, z. B. zum Monatsende oder für eine Besprechung. |
| **Akteure** | Geschäftsführung (primär) |
| **Eingehende Information** | Kalendermonat (Monat und Jahr); ohne Auswahl der laufende Monat (E38) |
| **Vorbedingungen** | • Die Geschäftsführung ist angemeldet.<br>• Ihr Nutzerkonto hat die Rolle Geschäftsführung (E40).<br>• Die Erfassung der Nutzungsereignisse ist eingeführt (E36, E37). |
| **Nachbedingungen** | • Das Dashboard zeigt für den gewählten Monat die neun Nutzungskennzahlen aus E39 mit Bezeichnung und Wert.<br>• Der Monat und der Beginn der Erfassung werden angezeigt; ist der laufende Monat gewählt, sind die Werte als „vorläufig“ gekennzeichnet (E38).<br>• Für einen Monat vor dem Beginn der Erfassung oder in der Zukunft erscheint „Für diesen Monat liegen keine Auswertungen vor.“ (400); es werden keine Nullwerte angezeigt (E38).<br>• Die Werte enthalten keine personenbezogenen Daten (E36, NF-14 b).<br>• Ohne die Rolle Geschäftsführung erscheint „Dafür fehlt die Berechtigung.“ (403); der Eintrag „Auswertungen“ ist in der Navigation nur mit der Rolle sichtbar (E40). |
| **Essenzielle Schritte** | • Dashboard aufrufen<br>• Monat wählen<br>• Kennzahlen des Monats aus den erfassten Nutzungsereignissen ermitteln<br>• Kennzahlen anzeigen |

---

## US-12 Nutzungskennzahlen exportieren

| **Name** | **Nutzungskennzahlen exportieren** |
|---|---|
| **Art** | User Story (US-12), aus Change Request CR-01, erweitert US-11 |
| **Kurzbeschreibung** | Die Geschäftsführung lädt die Kennzahlen des angezeigten Monats als CSV-Datei herunter, um sie in einer Tabellenkalkulation weiterzuverarbeiten oder weiterzugeben. |
| **Auslöser** | Die Geschäftsführung möchte die Zahlen eines Monats außerhalb von LookUpStock verwenden, z. B. in einem Bericht. |
| **Akteure** | Geschäftsführung (primär) |
| **Eingehende Information** | Der im Dashboard gewählte Monat (US-11) |
| **Vorbedingungen** | • Wie US-11.<br>• Im Dashboard ist ein Monat mit Auswertung angezeigt. |
| **Nachbedingungen** | • Der Browser lädt die Datei `lookupstock-nutzung-JJJJ-MM.csv` herunter (E41).<br>• Die Datei enthält dieselben Kennzahlen und Werte wie das Dashboard, je Kennzahl eine Zeile mit Monat, Bezeichnung und Wert (E41).<br>• Die Datei lässt sich in Excel mit deutscher Spracheinstellung öffnen; Umlaute werden korrekt angezeigt (E41).<br>• Fehlerfälle wie US-11 (401, 403, 400). |
| **Essenzielle Schritte** | • Export im Dashboard auslösen<br>• Datei mit den Kennzahlen des Monats erzeugen<br>• Datei herunterladen |

---

## Festlegungen des Product Owners (E36 bis E43)

Anlass: Change Request CR-01 (Ereigniskarte „Reporting für die Geschäftsführung“), Analyse am 25.10.2026. Die Nummerierung setzt E1 bis E35 fort. Technische Details der Umsetzung entscheidet das Backend-Team; der Product Owner übernimmt die umgesetzten Regeln danach in diese Tabelle.

| ID | Story | Festlegung | Begründung |
|---|---|---|---|
| E36 | US-11 | **Anonyme Erfassung:** Ein Nutzungsereignis besteht nur aus der Ereignisart und dem Zeitpunkt. Es werden weder Nutzer, IP-Adresse, Suchbegriff noch Anlage gespeichert. | NF-14 b erlaubt als personenbezogene Daten nur E-Mail-Adresse und Passwort-Hash. Kennzahlen mit Nutzerbezug wie „aktive Nutzer“ entfallen deshalb; als Annäherung dient die Zahl der Anmeldungen. |
| E37 | US-11 | **Gezählt werden nur erfolgreich abgeschlossene Aktionen** (Antwort 2xx): Registrierung, Anmeldung, Portfolio angelegt, Simulation durchgeführt, Simulation gespeichert, Vergleich durchgeführt, Beispielsimulation durchgeführt, Anlagensuche (gültige Suchanfrage, auch wenn das Ergebnis aus dem Cache kommt). Eine gespeicherte Simulation zählt als durchgeführte und als gespeicherte Simulation. | Abgelehnte Eingaben (400, 422) oder Ausfälle (503) sind keine Nutzung des Funktionsumfangs. Eindeutige Regel verhindert Doppel- und Fehlzählungen. |
| E38 | US-11 | **Auswertungszeitraum ist ein Kalendermonat** nach deutscher Zeit (Europe/Berlin). Ohne Auswahl wird der laufende Monat angezeigt und als „vorläufig“ gekennzeichnet. Monate vor dem Beginn der Erfassung und künftige Monate werden mit „Für diesen Monat liegen keine Auswertungen vor.“ abgelehnt (400). | „Monatlich“ laut Ereigniskarte. Nullwerte für Monate ohne Erfassung würden fälschlich „keine Nutzung“ bedeuten. Anders als bei den Kursdaten (E22) ist der laufende Monat sinnvoll, weil die Geschäftsführung den Zwischenstand sehen möchte. |
| E39 | US-11 | **Neun Kennzahlen je Monat:** neue Registrierungen, Anmeldungen, angelegte Portfolios, durchgeführte Simulationen, gespeicherte Simulationen, durchgeführte Vergleiche, Beispielsimulationen (Gäste), Anlagensuchen, Nutzerkonten insgesamt am Monatsende. | Die ersten acht stammen aus den Nutzungsereignissen (E37), die letzte aus den Nutzerkonten (Zählung, kein Personenbezug). Deckt alle Funktionen von US-01 bis US-09 ab. |
| E40 | US-11, US-12 | **Zugriff nur mit der Rolle Geschäftsführung.** Angemeldete Nutzer ohne die Rolle erhalten 403 „Dafür fehlt die Berechtigung.“; ohne Anmeldung 401 (E14). Der Navigationseintrag „Auswertungen“ ist nur mit der Rolle sichtbar. Das Team vergibt die Rolle ohne Oberfläche (Konfiguration bzw. Datenbank, Vorgehen im README des Backends). | Die Kennzahlen sind Unternehmenszahlen. Die Prüfung muss im Backend erfolgen, ein ausgeblendeter Link allein schützt nicht (NF-04). 403 statt 404, weil es nicht um eine fremde Ressource geht, sondern um eine fehlende Berechtigung. |
| E41 | US-12 | **Export als CSV-Datei:** Dateiname `lookupstock-nutzung-JJJJ-MM.csv`, Trennzeichen Semikolon, Zeichensatz UTF-8 mit BOM, Spalten `Monat;Kennzahl;Wert`, je Kennzahl eine Zeile, gleiche Bezeichnungen und Werte wie im Dashboard. | „Exportierbare Datei“ laut Ereigniskarte. CSV ist ohne zusätzliche Bibliothek erzeugbar; Semikolon und BOM sorgen dafür, dass Excel mit deutscher Einstellung Spalten und Umlaute richtig erkennt. |
| E42 | US-13 | **Nicht umgesetzt (Won't):** automatischer monatlicher Versand (US-13), Export als PDF oder Excel-Datei, Diagramme oder Zeitreihen über mehrere Monate, nutzerbezogene Auswertungen (z. B. aktive Nutzer, Ranglisten), Auswertungen für die Zeit vor dem Beginn der Erfassung, Rollenverwaltung in der Oberfläche. | Entscheidung „teilweise umsetzen“ (CR-01, Abschnitt 5): Kapazität von Sprint 5 und Abgabe der Webprogrammierung am 08.11.2026; Datenschutz (E36); rückwirkende Daten existieren nicht. |
| E43 | US-08 | **„Vergleiche speichern“ wird nicht umgesetzt (Won't).** Ersetzt E34 (geplant für Sprint 5). US-08 umfasst damit nur das Speichern, Aufrufen und Löschen von Simulationen. | Umpriorisierung zugunsten von CR-01: Komfortfunktion mit geringerem Nutzen als das Reporting; macht Kapazität in Sprint 5 frei. Der Vergleich selbst (US-07) bleibt unverändert. |

---

## Auswirkungen auf bestehende Stories

| Story | Feld | Bisher | Ab Sprint 5 | Grundlage |
|---|---|---|---|---|
| US-01 Registrieren und anmelden | Nachbedingungen | Konto angelegt bzw. angemeldet | Zusätzlich: Die Registrierung bzw. Anmeldung wird als Nutzungsereignis gezählt; für den Nutzer ändert sich nichts | E36, E37 |
| | Eingehende Information | E-Mail-Adresse und Passwort | **Unverändert.** Die Rolle Geschäftsführung wird nicht bei der Registrierung gewählt, sondern vom Team vergeben | E40 |
| US-02, US-03, US-04, US-07, US-09 | Nachbedingungen | – | Zusätzlich: Eine erfolgreiche Suche, Portfolio-Anlage, Simulation, ein Vergleich bzw. eine Beispielsimulation wird als Nutzungsereignis gezählt; Antworten und Anzeige bleiben unverändert | E37 |
| US-08 Simulation speichern und aufrufen | Umfang | Simulationen speichern; Vergleiche speichern folgt (E34) | Nur Simulationen; Vergleiche speichern Won't. Eine gespeicherte Simulation wird zusätzlich als Nutzungsereignis gezählt | E37, E43 |

---

## Akzeptanzkriterien

Zur Übernahme in die Story-Issues US-11 und US-12 (werden im Sprint-5-Planning angelegt). Für beide Stories gilt: Ohne gültige Anmeldung 401, ohne die Rolle Geschäftsführung 403.

**US-11 Nutzungskennzahlen anzeigen**

- Die Geschäftsführung kann einen Kalendermonat wählen; ohne Auswahl wird der laufende Monat angezeigt und als „vorläufig“ gekennzeichnet. *(E38)*
- Das Dashboard zeigt die neun Kennzahlen aus E39 mit Bezeichnung und Wert sowie den Beginn der Erfassung. *(E39)*
- Gezählt werden nur erfolgreich abgeschlossene Aktionen; eine abgelehnte Eingabe (400, 422) oder ein Ausfall (503) erhöht keinen Wert. *(E37)*
- Nach einer erfolgreichen Simulation mit Speichern steigen „durchgeführte Simulationen“ und „gespeicherte Simulationen“ jeweils um 1. *(E37)*
- Für einen Monat vor dem Beginn der Erfassung oder in der Zukunft erscheint „Für diesen Monat liegen keine Auswertungen vor.“ (400). *(E38)*
- Ein angemeldeter Nutzer ohne die Rolle erhält 403 „Dafür fehlt die Berechtigung.“ und sieht den Navigationseintrag „Auswertungen“ nicht. *(E40)*
- In der Datenbank enthalten Nutzungsereignisse nur Ereignisart und Zeitpunkt, keinen Bezug zu einem Nutzerkonto. *(E36, NF-14 b)*
- Bestehende Funktionen (US-01 bis US-09) liefern unveränderte Antworten; NF-01 bleibt erfüllt.

**US-12 Nutzungskennzahlen exportieren**

- Aus dem Dashboard lässt sich die Datei `lookupstock-nutzung-JJJJ-MM.csv` für den angezeigten Monat herunterladen. *(E41)*
- Die Datei enthält je Kennzahl eine Zeile `Monat;Kennzahl;Wert` mit denselben Werten wie das Dashboard. *(E41)*
- Die Datei öffnet sich in Excel mit deutscher Spracheinstellung in drei Spalten, Umlaute werden korrekt angezeigt. *(E41)*
- Fehlerfälle wie US-11 (401, 403, 400).

---

## Änderungshistorie

| Version | Datum | Story | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 25.10.2026 | US-11, US-12, US-08 | Neu: Kurzformat-Tabellen US-11 und US-12, Festlegungen E36 bis E43, Auswirkungen auf bestehende Stories, Akzeptanzkriterien | Docs #131, Change Request CR-01 |
