# Kurzformat-Tabellen – Sprint 4

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 4 (19.10.2026 – 25.10.2026), Umsetzung in Sprint 5 (26.10.2026 – 01.11.2026) |
| **Issue** | Docs #131 Change Request: Auswirkungsanalyse, Entscheidung, Backlog vorher und nachher, Kurzformat und Akzeptanzkriterien; Docs #2 US-09 Beispielsimulation (Version 2); Version 3: Anpassung an die Umsetzung in Sprint 5 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 3 |
| **Stand** | 01.11.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Sprint-Planning-Protokoll Sprint 4 (Akzeptanzkriterien US-09), Umsetzung BE #93, #103 und FE #38, API-Design Abschnitt „Gastmodus“, Change Request CR-01 (`change-request.md`), Ereigniskarte Nr. 8 „Reporting für die Geschäftsführung“ (erhalten am 23.10.2026), Kurzformat-Tabellen Sprint 1 bis Sprint 3 (E1 bis E35), NFRs Version 4 (NF-04, NF-14 b), ADR-06, ER-Diagramm v4 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt `kurzformat_sprint-3.md` fort. Sie enthält US-09 Beispielsimulation ausprobieren (Stretch in Sprint 4, Festlegungen E44 bis E47) und die neuen Stories aus dem Change Request CR-01: US-11 Nutzungskennzahlen anzeigen und US-12 Nutzungskennzahlen exportieren. US-13 (automatischer Monatsbericht) ist Won't und erhält keine Tabelle. Analyse, Optionen und Entscheidung stehen in `change-request.md`.

US-09 ist öffentlich und richtet sich an den **Gast**. US-11 und US-12 sind dem Akteur **Geschäftsführung** vorbehalten (Akteursbeschreibung v4). Ohne gültige Anmeldung antwortet das Backend mit 401 und die Oberfläche führt zur Anmeldung (E14); angemeldete Nutzer ohne die Rolle Geschäftsführung erhalten 403 (E40).

---

## US-09 Beispielsimulation ausprobieren

| **Name** | **Beispielsimulation ausprobieren** |
|---|---|
| **Art** | User Story (US-09), Stretch in Sprint 4 |
| **Kurzbeschreibung** | Ein Gast probiert auf der Startseite ohne Konto eine Monte-Carlo-Simulation mit einem vorgegebenen Beispielportfolio und festen Werten aus, um LookUpStock kennenzulernen, bevor er sich registriert. |
| **Auslöser** | Ein Gast öffnet die Startseite und möchte sehen, was LookUpStock zeigt. |
| **Akteure** | Gast (primär), Finanzdaten-API Twelve Data (sekundär) |
| **Eingehende Information** | Auswahl eines der Beispielportfolios (E45). Startkapital, Sparrate, Anlagehorizont und Pfade sind fest vorgegeben (E46). |
| **Vorbedingungen** | • Keine Anmeldung nötig; die Startseite ist öffentlich (E44).<br>• Für die Anlagen der Beispielportfolios liefert die Finanzdaten-API Kursdaten oder sie liegen im Cache (NF-07). |
| **Nachbedingungen** | • Das Ergebnis zeigt das Fächerdiagramm, die Kennzahlen aus US-05 (ohne Zielwahrscheinlichkeit), Pfade, Seed und Schätzzeitraum (wie US-04).<br>• Die Hinweise E16 und E21 stehen unter dem Ergebnis (NF-14 a).<br>• Ein Hinweis mit Link lädt zur Registrierung ein (E47).<br>• Es wird nichts gespeichert; eine Speichern-Funktion gibt es nicht (E47).<br>• Gleicher Seed ergibt identische Ergebnisse (NF-08).<br>• Sind die Kursdaten nicht abrufbar, erscheint „Kursdaten sind gerade nicht verfügbar.“ (503); zu kurze Historie wie US-04 (422). |
| **Essenzielle Schritte** | • Startseite ohne Anmeldung öffnen<br>• Beispielportfolio wählen<br>• Beispielsimulation mit den festen Werten berechnen<br>• Ergebnis mit Hinweisen und Einladung zur Registrierung anzeigen |

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

## Festlegungen des Product Owners (E36 bis E47)

Anlass: Change Request CR-01 (Ereigniskarte „Reporting für die Geschäftsführung“), Analyse am 25.10.2026. Die Nummerierung setzt E1 bis E35 fort. Technische Details der Umsetzung entscheidet das Backend-Team; der Product Owner übernimmt die umgesetzten Regeln danach in diese Tabelle.

| ID | Story | Festlegung | Begründung |
|---|---|---|---|
| E36 | US-11 | **Anonyme Erfassung:** Ein Nutzungsereignis besteht nur aus der Ereignisart und dem Zeitpunkt. Es werden weder Nutzer, IP-Adresse, Suchbegriff noch Anlage gespeichert. | NF-14 b erlaubt als personenbezogene Daten nur E-Mail-Adresse und Passwort-Hash. Kennzahlen mit Nutzerbezug wie „aktive Nutzer“ entfallen deshalb; als Annäherung dient die Zahl der Anmeldungen. |
| E37 | US-11 | **Gezählt werden nur erfolgreich abgeschlossene Aktionen** (Antwort 2xx): Registrierung, Anmeldung, Portfolio angelegt, Simulation durchgeführt, Simulation gespeichert, Vergleich durchgeführt, Beispielsimulation durchgeführt, Anlagensuche (gültige Suchanfrage, auch wenn das Ergebnis aus dem Cache kommt). Eine gespeicherte Simulation zählt als durchgeführte und als gespeicherte Simulation. | Abgelehnte Eingaben (400, 422) oder Ausfälle (503) sind keine Nutzung des Funktionsumfangs. Eindeutige Regel verhindert Doppel- und Fehlzählungen. |
| E38 | US-11 | **Auswertungszeitraum ist ein Kalendermonat** nach deutscher Zeit (Europe/Berlin). Ohne Auswahl wird der laufende Monat angezeigt und als „vorläufig“ gekennzeichnet. Monate vor dem Beginn der Erfassung und künftige Monate werden mit „Für diesen Monat liegen keine Auswertungen vor.“ abgelehnt (400). | „Monatlich“ laut Ereigniskarte. Nullwerte für Monate ohne Erfassung würden fälschlich „keine Nutzung“ bedeuten. Anders als bei den Kursdaten (E22) ist der laufende Monat sinnvoll, weil die Geschäftsführung den Zwischenstand sehen möchte. |
| E39 | US-11 | **Neun Kennzahlen je Monat:** neue Registrierungen, Anmeldungen, angelegte Portfolios, durchgeführte Simulationen, gespeicherte Simulationen, durchgeführte Vergleiche, Beispielsimulationen (Gäste), Anlagensuchen, Nutzerkonten insgesamt am Monatsende. | Die ersten acht stammen aus den Nutzungsereignissen (E37), die letzte aus den Nutzerkonten (Zählung, kein Personenbezug). Deckt alle Funktionen von US-01 bis US-09 ab. |
| E40 | US-11, US-12 | **Zugriff nur mit der Rolle Geschäftsführung.** Angemeldete Nutzer ohne die Rolle erhalten 403 „Dafür fehlt die Berechtigung.“; ohne Anmeldung 401 (E14). Der Navigationseintrag „Auswertungen“ ist nur mit der Rolle sichtbar. Das Team vergibt die Rolle ohne Oberfläche in der Datenbank; im Code heißt sie `MANAGER` (Migration V9), und `POST /auth/login` liefert sie im Feld `role`, damit das Frontend den Navigationseintrag einblenden kann. | Die Kennzahlen sind Unternehmenszahlen. Die Prüfung muss im Backend erfolgen, ein ausgeblendeter Link allein schützt nicht (NF-04). 403 statt 404, weil es nicht um eine fremde Ressource geht, sondern um eine fehlende Berechtigung. |
| E41 | US-12 | **Export als CSV-Datei:** Dateiname `lookupstock-nutzung-JJJJ-MM.csv`, Trennzeichen Semikolon, Zeichensatz UTF-8 mit BOM, Spalten `Monat;Kennzahl;Wert`, je Kennzahl eine Zeile, gleiche Bezeichnungen und Werte wie im Dashboard. | „Exportierbare Datei“ laut Ereigniskarte. CSV ist ohne zusätzliche Bibliothek erzeugbar; Semikolon und BOM sorgen dafür, dass Excel mit deutscher Einstellung Spalten und Umlaute richtig erkennt. |
| E42 | US-13 | **Nicht umgesetzt (Won't):** automatischer monatlicher Versand (US-13), Export als PDF oder Excel-Datei, Diagramme oder Zeitreihen über mehrere Monate, nutzerbezogene Auswertungen (z. B. aktive Nutzer, Ranglisten), Auswertungen für die Zeit vor dem Beginn der Erfassung, Rollenverwaltung in der Oberfläche. | Entscheidung „teilweise umsetzen“ (CR-01, Abschnitt 5): Kapazität von Sprint 5 und Abgabe der Webprogrammierung am 08.11.2026; Datenschutz (E36); rückwirkende Daten existieren nicht. |
| E43 | US-08 | **„Vergleiche speichern“ wird nicht umgesetzt (Won't).** Ersetzt E34 (geplant für Sprint 5). US-08 umfasst damit nur das Speichern, Aufrufen und Löschen von Simulationen. | Umpriorisierung zugunsten von CR-01: Komfortfunktion mit geringerem Nutzen als das Reporting; macht Kapazität in Sprint 5 frei. Der Vergleich selbst (US-07) bleibt unverändert. |
| E44 | US-09 | **Startseite öffentlich.** Ohne gültige Anmeldung zeigt die Startseite den Bereich „Beispielsimulation ausprobieren“. Bei „Simulation starten“ und „Gespeicherte Simulationen“ steht stattdessen „Bitte melde dich an.“ mit Link zur Anmeldung. Angemeldete Nutzer sehen die Startseite wie bisher, ohne den Beispielbereich. | PO-Entscheidung vom 25.10.2026: Ein Gast soll LookUpStock ohne Umweg über die Anmeldeseite kennenlernen. Die eigene Simulation (US-04) und gespeicherte Simulationen (US-08) bleiben geschützt. |
| E45 | US-09 | **Zwei Beispielportfolios:** „Beispiel: Weltportfolio“ (URTH 100 %) und „Beispiel: Welt 70 / Schwellenländer 30“ (URTH 70 %, IEMG 30 %). Die Zusammensetzung wird bei der Auswahl angezeigt. Die Portfolios sind fest vorgegeben, gehören keinem Nutzer und stehen nicht in der Datenbank. | Anlagen an US-Börsen wie in der PO-Entscheidung vom 18.10.2026, weil Twelve Data im kostenlosen Tarif keine Xetra-Kurse liefert (umgesetzt in #103). |
| E46 | US-09 | **Feste Werte:** Startkapital 5.000 €, Sparrate 100 € pro Monat, Anlagehorizont 15 Jahre, 10.000 Pfade, ohne Seed (das System erzeugt einen und zeigt ihn an). Der Gast kann die Werte nicht ändern. Das Backend lehnt mehr als 10.000 Pfade mit „In der Beispielsimulation sind höchstens 10.000 Simulationspfade möglich.“ ab (400). | PO-Entscheidung vom 25.10.2026: Werte wie im Beispiel des API-Designs („Gastmodus“), passend zu den Testfällen. 10.000 Pfade sind die Obergrenze des öffentlichen Endpunkts und halten die Last klein. |
| E47 | US-09 | **Nichts wird gespeichert;** `id`, `name` und `createdAt` der Antwort sind `null` (API-Design, Abschnitt „Gastmodus“). Unter dem Ergebnis steht „Eigene Portfolios zusammenstellen, simulieren, vergleichen und speichern? Jetzt kostenlos registrieren“ mit Link zur Registrierung. | Akzeptanzkriterium aus dem Sprint-Planning-Protokoll Sprint 4: Hinweis lädt zur Registrierung ein; Speichern ist nur mit Konto möglich (US-08). |

---

## Auswirkungen auf bestehende Stories

| Story | Feld | Bisher | Ab Sprint 5 | Grundlage |
|---|---|---|---|---|
| US-01 Registrieren und anmelden | Nachbedingungen | Konto angelegt bzw. angemeldet | Zusätzlich: Die Registrierung bzw. Anmeldung wird als Nutzungsereignis gezählt; für den Nutzer ändert sich nichts | E36, E37 |
| | Eingehende Information | E-Mail-Adresse und Passwort | **Unverändert.** Die Rolle Geschäftsführung wird nicht bei der Registrierung gewählt, sondern vom Team vergeben | E40 |
| US-02, US-03, US-04, US-07, US-09 | Nachbedingungen | – | Zusätzlich: Eine erfolgreiche Suche, Portfolio-Anlage, Simulation, ein Vergleich bzw. eine Beispielsimulation wird als Nutzungsereignis gezählt; Antworten und Anzeige bleiben unverändert | E37 |
| US-01 Registrieren und anmelden | Vorbedingungen | Die Startseite (Simulation) setzt die Anmeldung voraus; ohne Anmeldung Weiterleitung zur Anmeldeseite | Die Startseite ist öffentlich; geschützt bleiben die Bereiche „Simulation starten“ und „Gespeicherte Simulationen“ mit Hinweis und Link zur Anmeldung | E44 |
| US-08 Simulation speichern und aufrufen | Umfang | Simulationen speichern; Vergleiche speichern folgt (E34) | Nur Simulationen; Vergleiche speichern Won't. Eine gespeicherte Simulation wird zusätzlich als Nutzungsereignis gezählt | E37, E43 |

---

## Akzeptanzkriterien

Zur Übernahme in das Story-Issue US-09 (Docs #2) und in die Story-Issues US-11 und US-12 (werden im Sprint-5-Planning angelegt). Für US-11 und US-12 gilt: Ohne gültige Anmeldung 401, ohne die Rolle Geschäftsführung 403.

**US-09 Beispielsimulation ausprobieren**

- Ohne Anmeldung zeigt die Startseite den Bereich „Beispielsimulation ausprobieren“; die eigene Simulation und die gespeicherten Simulationen zeigen „Bitte melde dich an.“ mit Link. *(E44)*
- Der Gast wählt eines von zwei Beispielportfolios; die Zusammensetzung und die festen Werte werden angezeigt. *(E45, E46)*
- Das Ergebnis zeigt Fächerdiagramm, Kennzahlen aus US-05, Pfade, Seed und Schätzzeitraum sowie die Hinweise E16 und E21 (NF-14 a).
- Ein Hinweis mit Link lädt zur Registrierung ein; es gibt keine Speichern-Funktion und `id`, `name` und `createdAt` der Antwort sind `null`. *(E47)*
- Gleicher Seed ergibt identische Ergebnisse (NF-08); mehr als 10.000 Pfade werden mit 400 abgelehnt. *(E46)*
- Angemeldete Nutzer sehen die Startseite unverändert, ohne Beispielbereich. *(E44)*

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
| 2 | 25.10.2026 | US-09, US-01 | Neu: Kurzformat-Tabelle US-09, Festlegungen E44 bis E47 (Startseite öffentlich, Beispielportfolios, feste Werte, nichts gespeichert), Auswirkung auf US-01, Akzeptanzkriterien US-09 | Docs #2, PO-Entscheidung vom 25.10.2026, Umsetzung BE #93, #103, FE #38 |
| 3 | 01.11.2026 | US-11, US-12, US-09 | E40: Vergabe der Rolle per Datenbank, technischer Name `MANAGER`, Feld `role` bei der Anmeldung; E47 und Akzeptanzkriterium US-09: `id`, `name` und `createdAt` sind `null` statt „keine `id`“ | Anpassung an die Umsetzung (BE #110, BE #121, API-Design), Abnahmeprotokoll Sprint 4, Abschnitt 6 |
