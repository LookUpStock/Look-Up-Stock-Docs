# Wireframes

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #209 Wireframes aus Figma als PNG exportieren; Dashboard und Export: Docs #164 |
| **Werkzeug** | Figma (Datei „LookUpStock – Wireframes“, erstellt von Carl Stibbe) |
| **Export** | 05.11.2026, PNG in doppelter Auflösung |

Die Wireframes zeigen den Aufbau der Seiten der MPA vor der Umsetzung: Bereiche, Eingabefelder und Schaltflächen, ohne Farben und Texte. Bis Sprint 6 lagen sie nur in Figma; damit sie ohne Figma-Zugang Teil der Abgabe sind, liegen sie hier als Bild. Die SPA (ADR-08) hat kein eigenes Wireframe, weil sie die Bausteine der Seiten „Portfolios“ und „Simulation“ wiederverwendet.

**Figma:**
- Seiten 01 bis 04: https://www.figma.com/design/932zcNJBZPQQGf495GYMxB/LookUpStock-%E2%80%93-Wireframes?node-id=0-1
- Seite 05 (Change Request CR-01): https://www.figma.com/design/932zcNJBZPQQGf495GYMxB/LookUpStock-%E2%80%93-Wireframes?node-id=15-2

| Datei | Seite | User Stories | Abweichungen der Umsetzung |
|---|---|---|---|
| [`01-simulation.png`](01-simulation.png) | Simulation (`index.html`) | US-04, US-05, US-08 | Zusätzliches optionales Feld Zielbetrag (E27); „Zeitraum“ heißt Anlagehorizont. Unter dem Fächerdiagramm stehen die Kennzahlen aus US-05 statt drei Kacheln. Über dem Formular kam die Beispielsimulation für Gäste hinzu (US-09), darunter die Liste der gespeicherten Simulationen (US-08). |
| [`02-portfolios.png`](02-portfolios.png) | Portfolios (`portfolio.html`) | US-02, US-03, US-06 | Die Suche startet ohne Schaltfläche ab zwei Zeichen. Unter den Positionen steht die Gewichtssumme mit Status (H9, H13); die Schaltflächen heißen „Normalisieren“, „Gleichgewichten“ und „Speichern“. Gespeicherte Portfolios haben „Bearbeiten“ und „Löschen“ (US-06). |
| [`03-vergleich.png`](03-vergleich.png) | Vergleich (`compare.html`) | US-07 | Statt der drei Felder im Wireframe gibt es dieselben Eingaben wie bei der Simulation (Startkapital, Sparrate, Anlagehorizont, Pfade, Seed, Zielbetrag). Die Ergebnisse stehen wie im Wireframe als zwei Fächerdiagramme nebeneinander (E35), mit gemeinsamer Y-Achse (H1) und der Wahrscheinlichkeit, dass der erste Eintrag besser abschneidet (E31). |
| [`04-anmeldung.png`](04-anmeldung.png) | Anmeldung (`login.html`) | US-01 | Umgesetzt wie im Wireframe; der Hinweis oben erscheint nur nach dem Abmelden oder wenn eine Anmeldung fehlt oder abgelaufen ist (E14). Unter dem Passwort der Registrierung steht „Mindestens 8 Zeichen“. |
| [`05-dashboard-csv-export.png`](05-dashboard-csv-export.png) | Auswertungen (`dashboard.html`) | US-11, US-12 (CR-01) | Umgesetzt wie im Wireframe. „vorläufig“ steht beim ausgewählten Zeitraum, darunter der Beginn der Erfassung. Die Navigation zeigt statt „Geschäftsführung“ die E-Mail-Adresse und zusätzlich „Mein Bereich“. |
