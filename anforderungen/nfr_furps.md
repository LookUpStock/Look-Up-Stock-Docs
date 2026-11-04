# Nicht-funktionale Anforderungen nach FURPS+

**Projekt:** LookUpStock · **Issue:** Docs #16 (Version 4: Docs #131, Version 5: Docs #213) · **Verantwortlich:** Mert Eminel (Product Owner)
**Version:** 5 · **Stand:** 04.11.2026
**Grundlagen:** NFR-Entwurf von Paul Kaufmann, Product Backlog, API-Design (Stand 02.10.2026, Token im JSON-Body), ADR-01 bis ADR-03, Testkonzept und Abnahmetestfälle Sprint 1, PO-Entscheidungen vom 02.10.2026, ADR-06 (Token im Header), Change Request CR-01 und Kurzformat-Tabellen Sprint 4 (E36, E40), ADR-07 (getrennte Dienste), Abnahmeprotokoll Sprint 5, Testkonzept Version 6

Die Nummerierung NF-01 bis NF-15 stammt aus dem Entwurf und bleibt unverändert, weil die ADRs bereits darauf verweisen. NF-16 wurde ergänzt.

Jede Anforderung ist operationalisiert: Sie hat ein messbares Abnahmekriterium und einen Verweis auf den Nachweis. Die Testfall-IDs (`AT-…`) stehen in den Dateien `testkonzept/abnahmetestfaelle-sprint-<n>.md`, die übrigen Prüfmethoden und die vollständige Zuordnung aller Nachweise in Abschnitt 10 von `testkonzept/testkonzept.md`.

**Begriff „Cache“:** Zwischengespeicherte Kursdaten und Suchergebnisse, die bei Ausfall der Finanzdaten-API verwendet werden. Seit ADR-07 liegen beide im marktdaten-service (Kursdaten in der eigenen Datenbank, Schema `marktdaten`). Der Begriff steht so im Glossar.

## Übersicht

| NF | FURPS+ | Titel | Prüfung ab | Nachweis |
|---|---|---|---|---|
| NF-01 | Performance | Performance der Simulation | Sprint 1 | AT-04-14 |
| NF-02 | Performance | Performance der Suche | Sprint 1 | AT-02-08 |
| NF-03 | Functionality (Sicherheit) | Passwortschutz | Sprint 2 | AT-01-08, AT-01-11 |
| NF-04 | Functionality (Sicherheit) | Zugriffsschutz | Sprint 2 | AT-01-13, AT-01-14 |
| NF-05 | Functionality (Sicherheit) | Verschlüsselte Übertragung | ab Server-Deployment | AT-NF05-01 |
| NF-06 | Functionality (Sicherheit) | Schutz des API-Keys | Sprint 1 | Checkliste im Code-Review; AT-ADR7-02 |
| NF-07 | Reliability | Ausfallsicherheit | Sprint 1 | AT-02-07, AT-04-06, AT-04-07; AT-ADR7-03, AT-ADR7-07 |
| NF-08 | Reliability | Reproduzierbarkeit | Sprint 1 | AT-04-13 |
| NF-09 | Usability | Bedienbarkeit | Sprint 3 | Bedienbarkeitstest (AT-NF09-01) |
| NF-10 | Usability | Darstellung auf verschiedenen Geräten | Sprint 1 | Sichtprüfung im Abnahmetest |
| NF-11 | Supportability | Testabdeckung | Sprint 1 | JaCoCo-Bericht |
| NF-12 | Supportability | Austauschbare Datenquelle | Sprint 1 | grüner CI-Lauf ohne Internet |
| NF-13 | + Implementierung | Einfaches Deployment | ab Server-Deployment | AT-NF13-01 |
| NF-14 | + Recht | Rechtliche Abgrenzung | Sprint 1 / 2 | AT-04-15, AT-01-12 |
| NF-15 | + Schnittstelle | Dokumentierte Schnittstelle | Sprint 1 | Review bei jeder Schnittstellenänderung |
| NF-16 | Functionality (Sicherheit) | Begrenzte Sitzungsdauer | Sprint 2 | AT-01-02, AT-01-03, AT-01-06, AT-01-10 |

## Performance

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-01 | Eine Simulation mit 10.000 Pfaden ist mit 5 Anlagen (Messfall) in höchstens 3 Sekunden berechnet. | Gemessen wird `runtimeMs` aus der Antwort von `POST /simulations` auf dem Projektserver. Messbedingung: Portfolio mit 5 Anlagen, `horizonYears` = 40, `paths` = 10.000. Die Kursdaten liegen bereits im Cache, die Abrufzeit bei Twelve Data zählt nicht mit. In 20 Durchläufen liegen mindestens 19 Werte (95 %) bei höchstens 3.000 ms. Die 5 Anlagen und 40 Jahre (höchster erlaubter Anlagehorizont) sind der ungünstigste Messfall. Eine Höchstzahl an Anlagen pro Portfolio gibt es nicht. | AT-04-14 (Nachweis-NFR des Projekts) |
| NF-02 | Die Anlagensuche antwortet schnell genug, damit sich die Suche beim Tippen flüssig anfühlt. | 95 % der Anfragen an `GET /assets/search` antworten in höchstens 1 s, wenn die Ergebnisse bereits im Cache liegen. Gemessen mit 20 Suchbegriffen auf dem Projektserver; die Ergebnisse werden vorher durch eine erste Suche in den Cache geladen (seit ADR-07 im marktdaten-service). Suchen, die erst bei Twelve Data abgefragt werden, sind nicht Teil dieser Anforderung, da deren Antwortzeit außerhalb des Einflusses des Projekts liegt (analog zu NF-01). | AT-02-08 |

## Sicherheit

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-03 | Passwörter werden nur als gesalzener Hash gespeichert und haben mindestens 8 Zeichen. | In der Datenbank steht kein Passwort im Klartext. Die Werte haben das BCrypt-Format (`$2a$…`). `POST /auth/register` mit einem Passwort unter 8 Zeichen liefert 400 mit der Meldung aus dem API-Design. | Unit-Test und Prüfung der Datenbankinhalte: AT-01-08, AT-01-11 |
| NF-04 | Jeder Nutzer kann nur seine eigenen Portfolios und Simulationen sehen und ändern. Die Nutzungskennzahlen (US-11, US-12) sind nur mit der Rolle Geschäftsführung abrufbar. | (a) Nutzer B ruft `GET /…/{id}`, `PUT` und `DELETE` mit IDs von Nutzer A auf und erhält jeweils 404. Die Daten von A bleiben unverändert. `GET /portfolios` und `GET /simulations` zeigen nur eigene Einträge.<br>(b) Ein angemeldeter Nutzer ohne die Rolle Geschäftsführung erhält beim Abruf und beim Export der Nutzungskennzahlen 403; die Prüfung erfolgt im Backend (E40). | (a) Integrations- und Abnahmetest: AT-01-13, AT-01-14 (`PUT` und `DELETE` mit US-06 in Sprint 3); (b) Abnahmetest zu US-11 (Sprint 5) |
| NF-05 | Die Anwendung ist nur über HTTPS erreichbar, damit Zugangsdaten und der Token im Header `Authorization` (ADR-06) nicht mitgelesen werden können. | Ein Aufruf über `http://` wird auf `https://` umgeleitet. Das Zertifikat ist im Browser gültig, ohne Warnung. | Manuelle Prüfung nach dem Server-Deployment: AT-NF05-01 (Sprint 6) |
| NF-06 | Der Schlüssel für die Finanzdaten-API (Twelve Data) liegt nur im marktdaten-service (ADR-07) und gelangt nie in den Browser, in den core-service oder ins Repository. | Im Netzwerk-Tab des Browsers und im Frontend-Code taucht der Key nicht auf. Er steht nur in der Konfigurationsvariable `TWELVE_DATA_API_KEY` der Heroku-App des marktdaten-service. Der core-service ruft den marktdaten-service mit einem eigenen internen Schlüssel (`INTERNAL_API_KEY`) auf. `.env` steht in `.gitignore`, es gibt nur eine `.env.example` ohne echten Wert. Die CI läuft ohne echten Key (Platzhalter `ci-dummy-key`, NF-12). | Checkliste im Code-Review; Prüfung über NF-12; AT-ADR7-02 |
| NF-16 | Eine Anmeldung ist begrenzt gültig und lässt sich beenden. | `expiresAt` liegt 24 h nach dem Login. Ein abgelaufener Token liefert 401. Nach `POST /auth/logout` liefert derselbe Token bei jedem geschützten Endpunkt 401. | AT-01-02, AT-01-03, AT-01-06, AT-01-10 |

## Zuverlässigkeit

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-07 | Fällt die Finanzdaten-API aus, arbeitet die Anwendung mit den Daten aus dem Cache weiter, statt abzustürzen. | Geprüft mit simuliertem Ausfall von Twelve Data (Mock des Anbieters ist nicht erreichbar oder antwortet nicht). Seit ADR-07 liegt der Kurs-Cache im marktdaten-service:<br>(a) Daten im Cache, die jünger als 24 h sind, werden weiterverwendet; Suche und Simulation antworten mit `200 OK`.<br>(b) Daten, die älter als 24 h sind, werden nicht verwendet.<br>(c) Liegen keine verwendbaren Daten im Cache oder ist der marktdaten-service selbst nicht erreichbar, antwortet der core-service innerhalb von 5 s mit `503 UPSTREAM_UNAVAILABLE` und verständlicher deutscher `message`; das Frontend zeigt sie an und bleibt bedienbar. | AT-02-07, AT-04-06, AT-04-07; seit Sprint 5 AT-ADR7-03 (`MarktdatenClientTest`) und AT-ADR7-07 (`PriceServiceTest`) |
| NF-08 | Mit gleichem Seed und gleichen Eingaben liefert die Simulation exakt dasselbe Ergebnis. | Zwei Aufrufe von `POST /simulations` mit identischen Parametern und `seed` = 42 liefern identische `timeline`- und `estimates`-Werte. | AT-04-13; Unit-Test der `MonteCarloEngine` |

## Bedienbarkeit

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-09 | Nutzer ohne Vorkenntnisse können ohne Anleitung ein Portfolio anlegen und simulieren. | Bedienbarkeitstest mit 3 fachfremden Personen: Mindestens 2 legen ohne Hilfe in unter 5 Minuten ein Portfolio an und starten eine Simulation. | AT-NF09-01; Ablauf im Testkonzept, Ergebnis in `testkonzept/bedienbarkeitstest-nf-09.md` |
| NF-10 | Alle Seiten sind auch auf dem Smartphone bedienbar. | Ab 360 px Breite gibt es kein seitliches Scrollen, und alle Bedienelemente sind erreichbar. Geprüft in der aktuellen Version von Chrome und Firefox (Geräteansicht). | Sichtprüfung aller Seiten im Abnahmetest |

## Wartbarkeit und Testbarkeit

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-11 | Die Kernlogik (Simulation und Portfolio-Prüfung) ist durch automatisierte Tests abgedeckt. | Die Zeilenabdeckung der Kernlogik liegt bei mindestens 80 %, gemessen mit JaCoCo in der CI-Pipeline des Backends. Sie wird in jedem Sprint Review berichtet (Qualitätsmetrik). | JaCoCo-Bericht (Testkonzept Abschnitt 9) |
| NF-12 | Das Backend greift nur über eine Schnittstelle auf Kursdaten zu, sodass der Anbieter tauschbar ist. | Alle Tests laufen ohne Internetzugang mit einer Attrappe der Datenquelle (`MockMarketDataProvider` oder Mockito-Mocks des Interface `MarketDataProvider`). Ein Anbieterwechsel betrifft nur die Implementierung dieser Schnittstelle. | grüner CI-Lauf ohne API-Key und ohne Internetzugang |

## Implementierung, Recht, Schnittstelle

| NF | Anforderung | Abnahmekriterium | Nachweis |
|---|---|---|---|
| NF-13 | Jede Komponente lässt sich ohne manuelle Zwischenschritte auf dem Server bereitstellen. | Frontend, core-service und marktdaten-service laufen als eigene Heroku-Apps mit einer gemeinsamen Heroku-Postgres-Datenbank. Ein Deploy ist je App ein einziger Schritt (Deploy von `main`). Die Datenbankstruktur entsteht beim Start über die Flyway-Migrationen, die Konfiguration kommt nur aus Konfigurationsvariablen. Nach dem Deploy ist die Anwendung ohne weitere manuelle Schritte erreichbar. | AT-NF13-01 (Sprint 6) |
| NF-14 | Die Anwendung grenzt sich von Anlageberatung ab und speichert nur nötige personenbezogene Daten. | (a) Jede Ergebnisseite (Simulation, Vergleich, Gastmodus) zeigt den Hinweis, dass es sich um keine Anlageberatung handelt. (b) Ein Nutzerkonto speichert nur E-Mail-Adresse und Passwort-Hash. Nutzungsereignisse für die Nutzungskennzahlen enthalten nur Ereignisart und Zeitpunkt, keinen Bezug zu einem Nutzerkonto, keine IP-Adresse und keine Eingaben (E36). | (a) AT-04-15; (b) AT-01-12; Nutzungsereignisse: Abnahmetest zu US-11 (Sprint 5) |
| NF-15 | Frontend und Backend kommunizieren ausschließlich über die im API-Design beschriebene JSON-Schnittstelle unter `/api/v1` (REST und GraphQL). | Jeder Endpunkt entspricht in Pfad, Methode und Feldern dem API-Design. Alle Fehler haben das Format `{ "error": { "code", "message", "details" } }`. | Review bei jeder Schnittstellenänderung |

## Änderungen in Version 5 (04.11.2026)

Anlass: ADR-07 (getrennte Dienste), Abnahmeprotokoll Sprint 5 und Abgleich mit dem Testkonzept Version 6 (Docs #213).

| NF | Änderung | Grund |
|---|---|---|
| NF-06 | Key nur im marktdaten-service, interner Schlüssel zwischen den Diensten; CI mit Platzhalter statt „Key als GitHub Secret“ | ADR-07; die CI-Workflows setzen `TWELVE_DATA_API_KEY: ci-dummy-key`, ein Secret wird nicht verwendet |
| NF-07 | Kurs-Cache im marktdaten-service; 503 auch, wenn der marktdaten-service nicht erreichbar ist; Nachweise aus Sprint 5 ergänzt | ADR-07, AT-ADR7-03, AT-ADR7-07 |
| NF-02 | Cache wird vor der Messung durch eine erste Suche gefüllt, nicht über den `MockMarketDataProvider` | Der Mock läuft nur in automatisierten Tests; gemessen wird auf dem Projektserver mit Twelve Data (Abnahmeprotokoll Sprint 3) |
| NF-13 | `docker compose up` durch die Bereitstellung als Heroku-Apps ersetzt; Nachweis AT-NF13-01 | Das Team hat sich für Heroku entschieden (Docs #64); ein Docker-Deployment gibt es nicht. Die Anforderung (Bereitstellung ohne manuelle Zwischenschritte) bleibt, das Abnahmekriterium beschreibt das tatsächliche Deployment. |
| NF-05 | Nachweis AT-NF05-01 | Die Prüfung nach dem Deployment war noch keinem Testfall zugeordnet |
| NF-09, NF-12, NF-15 | Nachweis von NF-09 benannt; Attrappe um Mockito-Mocks ergänzt; GraphQL als Teil der Schnittstelle | Abgleich mit dem Testkonzept und dem Code |
| alle | Verweis auf die Testfall-Dateien aller Sprints statt nur Sprint 1 und 2 | Nachweise stehen inzwischen in den Dateien bis Sprint 6 |

## Änderungen in Version 4 (25.10.2026)

Anlass: Change Request CR-01 „Reporting für die Geschäftsführung“ (Docs #131) und ADR-06.

| NF | Änderung | Grund |
|---|---|---|
| NF-04 | Teil (b) ergänzt: Nutzungskennzahlen nur mit der Rolle Geschäftsführung, sonst 403. Pfade in Teil (a) auf GET umgestellt (`GET /…/{id}` statt `…/details`, `GET /portfolios` statt `…/list`). „Vergleiche“ gestrichen, weil Vergleiche nicht gespeichert werden (E43) | CR-01 (E40), ADR-06 |
| NF-05 | Token steht im Header `Authorization`, nicht mehr im JSON-Body | ADR-06 |
| NF-14 (b) | Nutzungsereignisse ohne Personenbezug ergänzt | CR-01 (E36): Die Erfassung der Nutzung darf keine weiteren personenbezogenen Daten einführen |

## Änderungen in Version 3 (08.10.2026)

Anlass: Abnahmetestfälle Sprint 2 (Docs #61) und Abgleich mit den Abnahmetestfällen Sprint 1.

| NF | Änderung | Grund |
|---|---|---|
| NF-01, NF-02, NF-07, NF-08, NF-14 (a) | Testfall-IDs korrigiert (z. B. AT-04-34 → AT-04-14) | Die bisherigen IDs stammten aus einem Entwurf des Testkonzepts und existierten in `abnahmetestfaelle-sprint-1.md` nicht. |
| NF-03, NF-04, NF-14 (b), NF-16 | Testfälle aus Sprint 2 eingetragen statt „wird mit US-01 ergänzt“ | US-01 in Sprint 2 umgesetzt |
| NF-16 | Gültigkeit von 24 h bestätigt (Umsetzung BE #30 sah zunächst 7 Tage vor) | PO-Entscheidung vom 08.10.2026: kurze Gültigkeit wegen sensibler Finanzdaten |

## Änderungen in Version 2 (02.10.2026)

Anlass: Entscheidungen des Product Owners vom 02.10.2026 und Abgleich mit dem Testkonzept.

| NF | Änderung | Grund |
|---|---|---|
| NF-01 | „bis zu 5 Anlagen“ durch „mit 5 Anlagen (Messfall)“ ersetzt; Hinweis ergänzt, dass es keine Höchstzahl an Anlagen gibt | PO-Entscheidung E3 |
| NF-01 | 20 Durchläufe, davon mindestens 95 % ≤ 3.000 ms (vorher: 10 Durchläufe, jeder ≤ 3.000 ms); Prüfung ab Sprint 1 | Abgleich mit dem Testkonzept; robuster gegen einzelne Ausreißer, Simulation wird bereits in Sprint 1 umgesetzt |
| NF-01 | Anlagehorizont 40 Jahre und Projektserver als Messumgebung festgelegt | PO-Entscheidung: 40 Jahre ist der höchste erlaubte Wert und damit der ungünstigste Fall |
| NF-02 | Nur noch der Fall mit Ergebnissen im Cache (höchstens 1 s); Suchen über Twelve Data ausgenommen | PO-Entscheidung: Gegen die Testattrappe gemessen hätte der Fall ohne Cache keine Aussagekraft, gegen Twelve Data widerspräche er den synthetischen Testdaten. Gleiche Abgrenzung wie bei NF-01. |
| NF-01, NF-02, NF-07 | Einheitlich „Cache“ statt „eigene Datenbank“; Begriff oben erklärt | PO-Entscheidung: Testkonzept und API-Design verwenden „Cache“. Gemeint ist derselbe Speicher in der eigenen Datenbank. |
| NF-07 | Antwortzeit höchstens 5 s, Weiterverwendung von Daten jünger als 24 h, Fehlercode `503 UPSTREAM_UNAVAILABLE` | Abgleich mit dem Testkonzept; macht die Anforderung messbar |
| NF-08 | `metrics` durch `estimates` ersetzt | Abgleich mit dem Testkonzept; Kennzahlen gehören zu US-05 (Sprint 2) |
| NF-09 | Mindestens 2 von 3 Testpersonen bestätigt | PO-Entscheidung |
| NF-06 | Anbieter „Twelve Data“ genannt | PO-Entscheidung E7 |
| alle | Eigene Testfall-Nummern `TF-NF-xx` durch die IDs und Prüfmethoden aus dem Testkonzept ersetzt | Die Nummern `TF-NF-xx` existierten im Testkonzept nicht. |
| NF-02, NF-07, NF-12 | Testattrappe als `MockMarketDataProvider` benannt | Abgleich mit dem Testkonzept |

## Änderungen in Version 1 gegenüber dem Entwurf

Der Entwurf von Paul Kaufmann wurde KI-gestützt erstellt und für Version 1 geprüft und überarbeitet:

| NF | Änderung | Grund |
|---|---|---|
| alle | Abnahmekriterium und Testfall ergänzt | Die Aufgabenstellung verlangt operationalisierte Anforderungen. |
| NF-01 | „bis zu 5 Anlagen“ und Messbedingungen ergänzt | Abgleich mit dem Product Backlog; ohne Bedingungen nicht messbar |
| NF-02 | Anteil von 95 % ergänzt | Messbarkeit nach Vorlesungsvorgabe |
| NF-03 | Hash-Verfahren und Mindestlänge ergänzt | „Hash“ allein ist nicht prüfbar; die Mindestlänge steht im API-Design. |
| NF-05 | Umleitung von HTTP auf HTTPS ergänzt | Der Token steht seit dem neuen API-Design im JSON-Body. |
| NF-07 | „Cache“ durch „eigene Datenbank“ ersetzt (in Version 2 wieder vereinheitlicht) | Widerspruch zum Projektkonzept (Fallback auf eigene Datenbank) |
| NF-14 | In zwei Abnahmekriterien aufgeteilt | Die Anforderung enthält zwei Themen. |
| NF-15 | „REST-Schnittstelle“ durch „JSON-Schnittstelle unter `/api/v1`“ ersetzt | Das API-Design nutzt für geschützte Abfragen POST statt GET und ist daher nicht rein REST-konform. |
| NF-16 | Neu ergänzt | Ablauf und Sperre des Tokens sind im API-Design festgelegt, fehlten im Entwurf. |
