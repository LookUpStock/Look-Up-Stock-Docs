# Abnahmeprotokoll – Sprint 2

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #62 Abnahmetests durchführen und Stories abnehmen |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Datum der Abnahme** | 11.10.2026 |
| **Stories** | US-01 Registrieren und anmelden (Docs #9), US-02 Anlagen suchen (Docs #8), US-03 Portfolio mit Gewichtungen erstellen (Docs #1), US-04 Monte-Carlo-Simulation durchführen (Docs #7, Kern-Story) |
| **Grundlagen** | [Abnahmetestfälle Sprint 1](abnahmetestfaelle-sprint-1.md), [Abnahmetestfälle Sprint 2](abnahmetestfaelle-sprint-2.md), [Testkonzept](testkonzept.md) Version 2 (Abschnitte 8 und 11) |
| **KI-Nutzung** | Durchführung mit KI-Unterstützung vorbereitet und ausgewertet, Tests vom Product Owner selbst durchgeführt, Entscheidungen vom Product Owner getroffen, siehe KI-Log |

Die Status der einzelnen Testfälle stehen in der Spalte „Status“ der beiden Dateien mit den Abnahmetestfällen. Dieses Protokoll fasst das Ergebnis je Story zusammen, begründet die Abnahmeentscheidung und listet die gefundenen Fehler.

---

## 1. Testumgebung und Durchführung

| | |
|---|---|
| **Frontend** | `https://lookupstockfrontend-96a9998805cb.herokuapp.com`, Stand `main` 91e761a (11.10.2026, 20:31, Merge FE #13) |
| **Backend** | `https://lookupstock-da2951e42089.herokuapp.com`, Stand `main` a78be4b (11.10.2026, 21:16, Merge BE #66 mit BE #50) |
| **Datenbank** | Heroku Postgres |
| **Finanzdaten** | echte Twelve Data API (der `MockMarketDataProvider` aus BE #53 ist noch nicht umgesetzt) |
| **Testnutzer** | `abnahme.a@example.com` / `geheim123` (Nutzer A), `abnahme.b@example.com` / `sicher456` (Nutzer B), am 11.10.2026 neu registriert |
| **Manuelle Tests** | Product Owner im Browser (Chrome); API-Prüfungen mit `fetch` in der Konsole der Entwicklertools; Datenbank über ein Heroku Dataclip (nur lesend) |
| **Automatisierte Tests** | Backend: CI-Lauf „Backend CI“ auf `main` nach dem Merge von BE #66 erfolgreich (11.10.2026, 21:16). Frontend: `npm test` lokal auf `main`, 48 von 48 Tests bestanden (die Frontend-CI führt die Tests nicht aus, siehe F7) |

**Bewertungsregel.** Ein manueller Testfall ist `bestanden`, wenn Oberfläche und API das erwartete Ergebnis zeigen. Ein automatisierter Testfall ist `bestanden`, wenn die zugeordneten Tests im letzten CI-Lauf auf `main` erfolgreich waren (Testkonzept, Abschnitt 8). Fehlt in den Tests ein einzelner Wert eines Grenzwerttestfalls, wurde die betroffene Prüfregel im Code gelesen; das ist in der Spalte „Status“ als „per Code-Review“ vermerkt. `offen` heißt: Der Testfall konnte nicht oder nur teilweise durchgeführt werden, meist weil die Umsetzung fehlt.

**Unterbrechung.** Zwischen ca. 21:00 und 21:20 Uhr war das Backend auf Heroku nicht erreichbar (Neustart nach mehreren Pushes auf `main`). Carl Stibbe hat es wieder in Betrieb genommen; alle Tests wurden danach durchgeführt.

---

## 2. Ergebnis je Story

| Story | Testfälle | bestanden | fehlgeschlagen | offen | Entscheidung |
|---|---|---|---|---|---|
| US-01 Registrieren und anmelden | 14 | 14 | 0 | 0 | **abgenommen** |
| US-02 Anlagen suchen | 9 | 6 | 2 | 1 | **nicht abgenommen** |
| US-03 Portfolio mit Gewichtungen erstellen | 11 | 7 | 2 | 2 | **nicht abgenommen** |
| US-04 Monte-Carlo-Simulation durchführen | 16 | 7 | 2 | 7 | **nicht abgenommen** |
| **Gesamt** | 50 | 34 | 6 | 10 | |

US-02 bis US-04 umfassen die Testfälle aus Sprint 1 und die Ergänzungen aus Sprint 2 (AT-02-09, AT-03-10, AT-03-11, AT-04-16).

---

## 3. Begründung

### US-01 Registrieren und anmelden – abgenommen

Alle 14 Testfälle sind bestanden. Registrieren, Anmelden mit beliebiger Groß- und Kleinschreibung der E-Mail, Abmelden, die Sperre des alten Tokens, die Weiterleitung geschützter Seiten und der Zugriffsschutz auf fremde Portfolios (NF-04) funktionieren in der Oberfläche und über die API. Die Datenbank enthält nur BCrypt-Hashes mit unterschiedlichem Salz (NF-03).

**Festlegung des Product Owners:** In AT-01-02 und AT-01-13 nennt das erwartete Ergebnis eine „Portfolioliste“ in der Oberfläche. Diese Liste gehört zum Portfolio-Editor von US-03 und fehlt dort (F2). Für US-01 entscheidend ist, dass nach der Anmeldung nur eigene Portfolios geliefert werden; das wurde über die API geprüft und ist erfüllt. Die fehlende Liste wird deshalb US-03 zugeordnet und verhindert die Abnahme von US-01 nicht.

**Offen für die Definition of Done:** Die KI-Log-Nummern in den Frontend-Dateien von FE #4 (`login.html`, `auth.js`, `api.js`, `style.css`, `auth.test.js`) sind noch als Platzhalter `[eintragen]` eingetragen und werden nachgetragen.

### US-02 Anlagen suchen – nicht abgenommen

Die Suche nach Ticker, ISIN und Name funktioniert, auch ohne Anmeldung (AT-02-09); bei keinem Treffer erscheint der richtige Hinweis. Nicht erfüllt sind:

- **AT-02-03:** Ein ausgewählter Treffer wird nicht in den Portfolio-Editor übernommen (F1).
- **AT-02-08 (NF-02):** Suchergebnisse werden nicht zwischengespeichert; jede Suche geht an Twelve Data (F5).
- **AT-02-07:** Der API-Ausfall ist automatisiert geprüft, kann manuell aber nicht herbeigeführt werden, weil der Schalter im `MockMarketDataProvider` (BE #53) fehlt.

### US-03 Portfolio mit Gewichtungen erstellen – nicht abgenommen

Im Backend sind alle Prüfregeln umgesetzt und getestet (Name, Summe 100 % ± 0,01, Gewichtung > 0, höchstens 2 Nachkommastellen, doppelte und unbekannte Anlagen). Ein über die API angelegtes Portfolio bleibt nach Ab- und Anmelden erhalten und ist nur für seinen Besitzer sichtbar. In der Oberfläche kann ein Portfolio aber weder gespeichert noch in einer Liste angezeigt oder für die Simulation ausgewählt werden (F2), und Anlagen lassen sich nicht aus der Suche übernehmen (F1).

### US-04 Monte-Carlo-Simulation durchführen – nicht abgenommen

Seit dem Merge von BE #50 (11.10.2026, 21:16) berechnet das Backend Simulationen. Über die API wurde geprüft:

- ohne Anmeldung → `401` (AT-04-16), mit dem Portfolio eines anderen Nutzers → `404` (AT-01-14);
- Portfolio mit Apple (AAPL, NASDAQ), Startkapital 10.000 €, Sparrate 100 €, 10 Jahre, 1.000 Pfade, Seed 42: `200`, Schätzzeitraum 11/2016 bis 10/2026, Rendite 29,1 % und Volatilität 27,1 % pro Jahr, in Monat 120 gilt `p5` < `p50` < `p95`, Laufzeit 260 ms.

Nicht erfüllt sind:

- Das Fächerdiagramm und die Ergebnisseite fehlen (FE #3); damit sind alle Testfälle aus Nutzersicht und der Hinweis „Keine Anlageberatung“ (AT-04-15) nicht durchführbar.
- Ein Portfolio mit EUNL (Xetra) wird mit `422` „weniger als 3 Jahre Kursdaten“ abgelehnt, obwohl der ETF seit 2009 gehandelt wird (F3). Damit ist auch das Basisportfolio der Testfälle nicht simulierbar.
- Derselbe Seed liefert beim ersten Lauf nach einem neuen Kursabruf ein anderes Ergebnis als bei den folgenden Läufen (F4, AT-04-13, NF-08).
- Die Performancemessung NF-01 (AT-04-14) wurde nicht durchgeführt.

---

## 4. Gefundene Fehler und fehlende Umsetzung

Priorität nach Testkonzept, Abschnitt 11. „Neuer Task“ heißt: Es fehlt kein fehlerhaft umgesetzter Code, sondern die Umsetzung selbst; dafür wird ein Task für Sprint 3 angelegt.

| ID | Repo | Beschreibung | Testfälle | Priorität | Art |
|---|---|---|---|---|---|
| F1 | FE | Ein angeklickter Suchtreffer wird nur markiert, aber nicht als Position in den Portfolio-Editor übernommen. Die Suche löst das Ereignis `asset-selected` aus, der Editor reagiert nicht darauf. | AT-02-03, AT-03-08 | hoch | Bug |
| F2 | FE | Der Portfolio-Editor speichert nicht im Backend (`POST /portfolios`); es gibt keine Liste der eigenen Portfolios und keine Auswahl für die Simulation. | AT-03-01, AT-03-07, AT-03-10 | hoch | neuer Task |
| F3 | BE | `POST /simulations` mit EUNL (Xetra) liefert `422` „weniger als 3 Jahre Kursdaten“, mit AAPL (NASDAQ) funktioniert die Simulation. Vermutete Ursache: Twelve Data liefert für Xetra im aktuellen Tarif keine Monatskurse; der Fehler „nicht gefunden“ wird in `TwelveDataProvider` als leere Liste behandelt. Die genaue Meldung steht im Heroku-Log („Twelve Data (time_series EUNL): keine Daten – …“). | AT-04-08, alle US-04-Testfälle mit dem Basisportfolio | hoch | Bug |
| F4 | BE | Gleicher Seed, unterschiedliches Ergebnis: Der erste Lauf rechnet mit den Kursen direkt von Twelve Data, die folgenden mit den Werten aus dem Cache, die auf 6 Nachkommastellen gerundet sind (`NUMERIC(19,6)`). Nachweis: Lauf 1 `p5` = 57344,24137…, Lauf 2 und 3 `p5` = 57344,24135… Lösung: Kurse vor der Schätzung immer auf 6 Nachkommastellen runden oder die Kurse mit voller Genauigkeit speichern; Ende-zu-Ende-Test über zwei Läufe ergänzen. | AT-04-13 (NF-08) | hoch | Bug |
| F5 | BE | Es gibt keinen Cache für Suchergebnisse; NF-02 (≤ 1 s bei Cache-Treffer) ist nicht umgesetzt. | AT-02-08 | hoch | neuer Task |
| F6 | BE | Aufräumen nach BE #64 und BE #65: ungenutzter Alt-Code `data/api/UserManager` und `data/impl/PropertyFileUserManagerImpl`, der Passwörter im Klartext in `Users.properties` speichern würde (widerspricht NF-03, wird aber nicht aufgerufen); doppelter Datasource-Block in `application.properties` (der zweite überschreibt die Heroku-Einstellung); leere Migration `V5__kursdaten_geladen.sql`. | – | mittel | Bug |
| F7 | FE | Die Frontend-CI installiert keine Abhängigkeiten und führt `npm test` nicht aus; der Lauf ist dadurch immer grün. Die Definition of Done verlangt Unit-Tests in der CI. | – | mittel | Bug |

Zusätzlich offen und bereits als Task vorhanden: Fächerdiagramm (FE #3), `MockMarketDataProvider` mit Ausfall-Schalter (BE #53).

---

## 5. Nachweise der manuellen Tests

| Prüfung | Testfälle | Beobachtung |
|---|---|---|
| Suche als Gast | AT-02-09, AT-02-01, AT-02-04 | Hinweis „Bitte mindestens 2 Zeichen eingeben.“; `EUNL` liefert Treffer mit Name und „EUNL · XETR · EUR“ |
| ISIN, Name, kein Treffer | AT-02-02, AT-02-06 | `IE00B4L5Y983` findet EUNL und weitere Handelsplätze desselben ETFs; `msci` liefert mehrere Treffer; `xyzxyz123` zeigt „Keine passende Anlage gefunden.“ |
| Registrieren | AT-01-01, AT-01-08 | Passwort mit 7 Zeichen: Meldung vor dem Absenden; danach „Registrierung erfolgreich. Bitte melde dich jetzt an.“, E-Mail im Anmeldeformular, nicht angemeldet |
| E-Mail bereits registriert | AT-01-04 | `Abnahme.A@Example.com` → „Diese E-Mail ist bereits registriert.“, Eingaben bleiben erhalten |
| Falsche Anmeldedaten | AT-01-05 | falsches Passwort und unbekannte E-Mail: dieselbe Meldung; API beide Male `401` mit identischem Body |
| Anmelden | AT-01-02 | `ABNAHME.A@example.com` angemeldet, Kopfzeile zeigt `abnahme.a@example.com`; `expiresAt` 12.10.2026, 21:40:37 (24 h nach der Anmeldung) |
| Treffer übernehmen | AT-02-03 | Treffer wird blau markiert, der Editor bleibt leer (F1) |
| Nur eigene Portfolios | AT-01-13, AT-03-10 | A legt „Abnahme A“ (EUNL 100 %) per API an → `201`; B: Liste leer, Details des Portfolios von A → `404` „Portfolio nicht gefunden.“; A nach erneuter Anmeldung: Portfolio in der Liste |
| Abmelden | AT-01-03 | „Du hast dich abgemeldet.“; alter Token → `401` „Bitte melde dich an.“; Simulationsseite leitet zur Anmeldung; Portfolioseite: Suche nutzbar, Editor zeigt „Bitte melde dich an. Zur Anmeldung“ |
| 360 px Breite | NF-10 | Anmelde- und Portfolioseite ohne seitliches Scrollen, Navigation bricht in zwei Zeilen um |
| Passwort-Hash | AT-01-11 | Dataclip auf `app_user`: beide Hashes beginnen mit `$2a$10$`, unterschiedliches Salz, kein Klartext |
| Simulation über die API | AT-04-16, AT-01-14, AT-04-03, AT-04-08, AT-04-13 | siehe Abschnitt 3, US-04 |

---

## 6. Abweichungen vom Testkonzept

- **Testdaten:** Getestet wurde mit der echten Twelve Data API statt mit dem `MockMarketDataProvider`, weil BE #53 noch offen ist. Deshalb waren die Ausfallszenarien (AT-02-07, AT-04-06) nur automatisiert prüfbar, und die Testanlagen mit kurzer Historie (AT-04-08) standen nicht zur Verfügung.
- **Testnutzer:** neue Konten `abnahme.a@example.com` und `abnahme.b@example.com` statt `anna@example.com` und `ben@example.com`.
- **AT-01-11:** Statt eines dritten Nutzers mit demselben Passwort wurde das Salz an den unterschiedlichen Zeichen nach `$2a$10$` geprüft.
- **API-Prüfungen** wurden mit `fetch` in der Konsole der Entwicklertools statt mit einem eigenen HTTP-Client durchgeführt.

---

## 7. Folgen für Sprint 3

- US-02, US-03 und US-04 werden in das Sprint Backlog von Sprint 3 übernommen.
- Für F1 bis F7 werden Issues angelegt (Bugs mit Label `bug` in BE bzw. FE, neue Tasks für F2 und F5). Nach dem Merge der Korrekturen werden die betroffenen Testfälle erneut durchgeführt (Testkonzept, Abschnitt 11).
- Erneut durchzuführen sind alle Testfälle mit dem Status `fehlgeschlagen` oder `offen`, außerdem AT-04-14 (NF-01) als Messung auf Heroku.
