# Mitarbeit an LookUpStock

Dieses Dokument legt fest, wie wir im Team mit den Repositories arbeiten. Es gilt für alle drei Repositories der Organisation **LookUpStock**.

## Repositories

| Repository | Inhalt |
|---|---|
| `Look-Up-Stock-BackEnd` | Zwei Dienste (ADR-07): `core-service` mit REST- und GraphQL-API, Simulation, Vergleich, Anmeldung und Auswertungen; `marktdaten-service` mit Anbindung an Twelve Data und Kurs-Cache; Datenbankmigrationen, Tests |
| `Look-Up-Stock-FrontEnd` | Benutzeroberfläche als MPA und SPA (ADR-08), Unit-Tests der Oberfläche |
| `Look-Up-Stock-Docs` | Anforderungen, Sprint-Dokumentation, Diagramme, ADRs, API-Design, Testkonzept, Portfolio Webprogrammierung, KI-Log |

Im Folgenden werden die Repositories mit **BE**, **FE** und **Docs** abgekürzt.

## Ordnerstruktur

**BE** (je Dienst ein eigenes Maven-Projekt)

```
core-service/                   Dienst für Anmeldung, Portfolios, Simulation, Vergleich, Auswertungen
  src/main/java/                Quellcode (Paket com.lookupstock.backend und Unterpakete)
  src/main/resources/           Konfiguration, GraphQL-Schema, Flyway-Migrationen (db/migration)
  src/test/java/                Automatisierte Tests
  .mvn/wrapper/, mvnw, mvnw.cmd Maven Wrapper
  pom.xml                       Maven-Projektdatei
  .env.example                  Vorgabe für Umgebungsvariablen
  Procfile, system.properties   Start und Java-Version auf Heroku
  README.md                     Start, Konfiguration, Endpunkte
marktdaten-service/             Dienst für Suche und Kurse über Twelve Data (gleicher Aufbau, Paket com.lookupstock.marktdaten)
.github/workflows/              CI je Dienst (backend-ci.yml, marktdaten-ci.yml)
README.md                       Überblick über beide Dienste und Deployment
```

**FE**

```
src/                  Seiten der MPA (*.html), SPA-Einstieg app.html, css/, js/ (js/spa/ für die SPA)
tests/                Automatisierte Tests (node:test)
package.json          Start auf Heroku (serve) und Testbefehle
.github/workflows/    CI-Pipeline
README.md
```

**Docs**

```
anforderungen/        Kurzformat-Tabellen, Akteure, Glossar, NFRs, Simulationsmodell, Change Request
diagramme/            Use-Case-, Domänen-, Klassen-, ER-, Aktivitäts-, Sequenz- und Architekturdiagramme (aktueller Stand)
diagramme/archiv/     ältere Diagrammversionen
adr/                  Architecture Decision Records (ADR-01.md bis ADR-08.md)
api/                  API-Design: Schnittstelle zwischen FE und BE und zwischen den Diensten
testkonzept/          Testkonzept, Abnahmetestfälle und Abnahmeprotokolle je Sprint, Testabdeckung
sprints/sprint-<n>/   Sprint-Planning-Protokoll und Sprint Review je Sprint
ki-log/               KI-Log je Teammitglied
wireframes/           Wireframes als PNG-Export aus Figma
Portfolio-Webprogrammierung-LookUpStock.pdf
                      Portfolio Webprogrammierung
```

Dateinamen in `sprints/`: `Sprint-Planning-Protokoll-Sprint-<n>.docx` und `Sprint-Review-Sprint-<n>.docx`.

Diagramme werden nicht überschrieben, sondern versioniert abgelegt (z. B. `klassendiagramm_v1.png`, `klassendiagramm_v2.png`), damit das iterative Wachstum und der Stand vor und nach dem Change Request nachvollziehbar bleiben. Ersetzte Versionen werden im selben Pull Request nach diagramme/archiv/ verschoben; im Ordner diagramme/ liegt nur der aktuelle Stand.

## Branching-Strategie (GitHub Flow)

- `main` ist immer lauffähig und geschützt. Direkte Pushes auf `main` sind nicht möglich (Branch-Schutz, siehe Pull Requests).
- Für jedes Issue wird ein eigener Branch von `main` erstellt.
- Änderungen gelangen ausschließlich über einen Pull Request nach `main`.

**Branch-Namen**

```
feature/<issue-nr>-<kurzbeschreibung>    neue Funktion,  z. B. feature/27-such-endpunkt
fix/<issue-nr>-<kurzbeschreibung>        Fehlerbehebung, z. B. fix/33-gewichtung
docs/<issue-nr>-<kurzbeschreibung>       Dokumentation,  z. B. docs/17-domaenenmodell
```

Kleinbuchstaben, Wörter mit Bindestrich trennen, keine Umlaute.

## Commit-Nachrichten

Die Issue-Nummer angeben, am Ende in Klammern oder als Präfix mit dem Repository.

```
Portfolio-Endpunkt mit Gewichtungsprüfung ergänzt (#33)
Unit-Tests für Monte-Carlo-Engine hinzugefügt (#41)
Docs #213: Sequenz- und Aktivitätsdiagramme US-04 und US-07 v3
```

Jedes Teammitglied committet seine Arbeit selbst, damit die Commit-Historie die Beiträge aller nachvollziehbar macht.

## Pull Requests

1. Pull Request von deinem Branch nach `main` öffnen.
2. In der Beschreibung kurz erklären, was geändert wurde, und das Issue verknüpfen, z. B. `closes #33`. Bei Issues aus einem anderen Repository: `closes LookUpStock/Look-Up-Stock-Docs#8`.
3. Mindestens ein anderes Teammitglied als Reviewer eintragen.
4. Gemergt wird erst, wenn
   - mindestens ein Approval vorliegt und
   - die CI-Pipeline erfolgreich durchgelaufen ist.
5. Nach dem Merge wird der Branch gelöscht.

Diese Regeln sind in allen drei Repositories als Branch-Schutz für `main` eingerichtet (Docs #207): Merge nur über einen Pull Request mit mindestens einem Approval; im FE muss zusätzlich die CI durchlaufen. Im BE ist die CI kein Pflicht-Check, weil die beiden Workflows Pfadfilter haben und ein PR, der nur einen Dienst ändert, sonst auf den anderen Check warten würde; ein Merge mit roter CI ist trotzdem nicht erlaubt.

## Umgang mit Issues

- Jede Aufgabe ist ein Issue mit verantwortlicher Person und passenden Labels. Welche Issues zu einem Sprint gehören, legt das Sprint-Planning-Protokoll fest; Milestones wurden nur in Sprint 1 und 2 genutzt.
- Tasks hängen als Sub-Issues unter ihrer User Story.
- Status im Project „Docs“ pflegen: **TODO → In Progress → In Review → Done**.
  - *In Progress*, sobald du mit der Arbeit beginnst.
  - *In Review*, sobald der Pull Request offen ist.
  - *Done*, nach dem Merge.
- Beim Schließen eines Issues die tatsächlich benötigte Zeit im Feld **istdauer** eintragen (halbe Stunden genügen).

## Definition of Done

Eine User Story gilt als erledigt, wenn

- alle Akzeptanzkriterien erfüllt sind,
- der Code im Repository liegt und von einem zweiten Teammitglied gesichtet wurde,
- die Unit-Tests in der CI-Pipeline erfolgreich laufen,
- die Abnahmetestfälle dokumentiert und bestanden sind,
- die betroffenen Diagramme aktualisiert sind,
- jede KI-Nutzung im KI-Log eingetragen ist.

## Kommentare im Code

- Kommentare stehen nur dort, wo der Code eine Entscheidung nicht selbst erklärt (z. B. warum `308` statt `301`). Was der Code tut, zeigen Namen und Tests.
- Kommentare werden auf Englisch geschrieben.
- Flyway-Migrationen (`db/migration/`) werden nach dem Merge nicht mehr geändert, auch ihre Kommentare nicht: Flyway prüft eine Prüfsumme je Migration, jede Änderung lässt den Start auf Heroku scheitern.

## KI-Nutzung

- Jede KI-Nutzung, deren Ergebnis in ein abgegebenes Artefakt einfließt, wird im KI-Log eingetragen. Jedes Teammitglied führt ein eigenes KI-Log nach der gemeinsamen Vorlage und legt es als `ki-log/KI-Log-<Vorname>-<Nachname>.docx` im Docs-Repository ab; am Projektende werden die Logs zusammengeführt. Die Einträge sind je Person fortlaufend nummeriert (z. B. `ME-12` im KI-Log von Mert Eminel).
- Keine personenbezogenen oder vertraulichen Daten (z. B. aus dem Ausbildungsbetrieb) in KI-Werkzeuge eingeben.

## Zugangsdaten und Secrets

- API-Keys, Datenbankpasswörter und andere Zugangsdaten werden **niemals** committet.
- Lokal liegen sie in einer `.env`-Datei, die in `.gitignore` eingetragen ist. Eine `.env.example` ohne echte Werte zeigt, welche Variablen benötigt werden.
- Die CI-Pipelines brauchen keine echten Zugangsdaten: Sie laufen mit Platzhaltern und ohne Zugriff auf Twelve Data (NF-12). Werden doch Zugangsdaten nötig, werden sie als GitHub Secrets hinterlegt.
- Auf Heroku stehen alle Zugangsdaten nur in den Konfigurationsvariablen der jeweiligen App; der Key von Twelve Data nur beim marktdaten-service (NF-06).
- Ist ein Schlüssel versehentlich committet worden, genügt Löschen nicht, weil er in der Git-Historie bleibt: Er wird sofort beim Anbieter ersetzt (Abnahmeprotokoll Sprint 5, B10).
