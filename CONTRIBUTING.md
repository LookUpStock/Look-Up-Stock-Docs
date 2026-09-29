# Mitarbeit an LookUpStock

Dieses Dokument legt fest, wie wir im Team mit den Repositories arbeiten. Es gilt für alle drei Repositories der Organisation **LookUpStock**.

## Repositories

| Repository | Inhalt |
|---|---|
| `Look-Up-Stock-BackEnd` | REST-API, Simulationslogik, Datenbankzugriff, Unit-Tests |
| `Look-Up-Stock-FrontEnd` | Benutzeroberfläche, Unit-Tests der Oberfläche |
| `Look-Up-Stock-Docs` | User Stories, Sprint-Dokumentation, Diagramme, ADRs, Testkonzept, KI-Log, Deployment |

Im Folgenden werden die Repositories mit **BE**, **FE** und **Docs** abgekürzt.

## Ordnerstruktur

**BE und FE**

```
src/                  Quellcode
tests/                Automatisierte Tests
.github/workflows/    CI-Pipeline
Dockerfile
README.md
```

**Docs**

```
sprints/sprint-1/     Sprint Planning, Sprint Review, Screenshots
diagramme/            Use-Case-, Domänen-, Klassen-, ER-, Aktivitäts- und Sequenzdiagramme
adr/                  Architecture Decision Records (ADR-01.md, ADR-02.md, ...)
api/                  Beschreibung der Schnittstelle zwischen FE und BE
testkonzept/          Testkonzept und Abnahmetestfälle
ki-log/               KI-Log und KI-Reflexion
deployment/           docker-compose.yml und Server-Konfiguration
```

Diagramme werden nicht überschrieben, sondern versioniert abgelegt (z. B. `klassendiagramm_v1.png`, `klassendiagramm_v2.png`), damit das iterative Wachstum und der Stand vor und nach dem Change Request nachvollziehbar bleiben.

## Branching-Strategie (GitHub Flow)

- `main` ist immer lauffähig und geschützt. Direkte Pushes auf `main` sind nicht erlaubt.
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

Am Ende die Issue-Nummer angeben.

```
Portfolio-Endpunkt mit Gewichtungsprüfung ergänzt (#33)
Unit-Tests für Monte-Carlo-Engine hinzugefügt (#41)
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

Diese Regeln sind in BE und FE als Branch-Schutz für `main` eingerichtet.

## Umgang mit Issues

- Jede Aufgabe ist ein Issue mit verantwortlicher Person, Milestone (Sprint) und passenden Labels.
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

## KI-Nutzung

- Jede KI-Nutzung, deren Ergebnis in ein abgegebenes Artefakt einfließt, wird im KI-Log (`Docs/ki-log/`) eingetragen.
- Wesentliche KI-generierte Codeabschnitte werden im Quellcode per Kommentar mit der Nummer aus dem KI-Log gekennzeichnet:

```
// KI-Log Nr. 14: Grundgerüst der Monte-Carlo-Schleife mit KI erstellt, angepasst und getestet
```

- Keine personenbezogenen oder vertraulichen Daten (z. B. aus dem Ausbildungsbetrieb) in KI-Werkzeuge eingeben.

## Zugangsdaten und Secrets

- API-Keys, Datenbankpasswörter und andere Zugangsdaten werden **niemals** committet.
- Lokal liegen sie in einer `.env`-Datei, die in `.gitignore` eingetragen ist. Eine `.env.example` ohne echte Werte zeigt, welche Variablen benötigt werden.
- In der CI-Pipeline werden sie als GitHub Secrets hinterlegt.
