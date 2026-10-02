# Nicht-funktionale Anforderungen nach FURPS+

**Projekt:** LookUpStock · **Issue:** Docs #16 · **Verantwortlich:** Mert Eminel (Product Owner)
**Version:** 2 · **Stand:** 02.10.2026
**Grundlagen:** NFR-Entwurf von Paul Kaufmann, Product Backlog, API-Design (Stand 30.09.2026, Token im JSON-Body), ADR-01 bis ADR-03, PO-Entscheidungen vom 02.10.2026

Die Nummerierung NF-01 bis NF-15 stammt aus dem Entwurf und bleibt unverändert, weil die ADRs bereits darauf verweisen. NF-16 wurde ergänzt.

Jede Anforderung ist operationalisiert: Sie hat ein messbares Abnahmekriterium und einen Testfall (`TF-NF-xx`), der im Testkonzept beschrieben wird.

**Begriff „Cache“:** In der eigenen Datenbank gespeicherte Kursdaten und Suchergebnisse, die bei Ausfall der Finanzdaten-API verwendet werden. Der Begriff wird so ins Glossar übernommen.

## Übersicht

| NF | FURPS+ | Titel | Prüfung ab | Testfall |
|---|---|---|---|---|
| NF-01 | Performance | Performance der Simulation | Sprint 3 | TF-NF-01 |
| NF-02 | Performance | Performance der Suche | Sprint 1 | TF-NF-02 |
| NF-03 | Functionality (Sicherheit) | Passwortschutz | Sprint 2 | TF-NF-03 |
| NF-04 | Functionality (Sicherheit) | Zugriffsschutz | Sprint 2 | TF-NF-04 |
| NF-05 | Functionality (Sicherheit) | Verschlüsselte Übertragung | Sprint 2 | TF-NF-05 |
| NF-06 | Functionality (Sicherheit) | Schutz des API-Keys | Sprint 1 | TF-NF-06 |
| NF-07 | Reliability | Ausfallsicherheit | Sprint 1 | TF-NF-07 |
| NF-08 | Reliability | Reproduzierbarkeit | Sprint 1 | TF-NF-08 |
| NF-09 | Usability | Bedienbarkeit | Sprint 3 | TF-NF-09 |
| NF-10 | Usability | Darstellung auf verschiedenen Geräten | Sprint 1 | TF-NF-10 |
| NF-11 | Supportability | Testabdeckung | Sprint 1 | TF-NF-11 |
| NF-12 | Supportability | Austauschbare Datenquelle | Sprint 1 | TF-NF-12 |
| NF-13 | + Implementierung | Einfaches Deployment | Sprint 1 | TF-NF-13 |
| NF-14 | + Recht | Rechtliche Abgrenzung | Sprint 1 / 2 | TF-NF-14 |
| NF-15 | + Schnittstelle | Dokumentierte Schnittstelle | Sprint 1 | TF-NF-15 |
| NF-16 | Functionality (Sicherheit) | Begrenzte Sitzungsdauer | Sprint 2 | TF-NF-16 |

## Performance

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-01 | Eine Simulation mit 10.000 Pfaden ist mit 5 Anlagen (Messfall) in höchstens 3 Sekunden berechnet. | Gemessen wird `runtimeMs` aus der Antwort von `POST /simulations` auf dem Projektserver. Messbedingung: Portfolio mit 5 Anlagen, `horizonYears` = 40, `paths` = 10.000. Die Kursdaten liegen bereits im Cache, die Abrufzeit bei Twelve Data zählt nicht mit. In 10 Durchläufen liegt jeder Wert bei höchstens 3.000 ms. Die 5 Anlagen und 40 Jahre sind der ungünstigste Messfall. Eine Höchstzahl an Anlagen pro Portfolio gibt es nicht. | TF-NF-01 (automatisierter Test, Nachweis-NFR des Projekts) |
| NF-02 | Die Anlagensuche antwortet schnell genug, damit sich die Suche beim Tippen flüssig anfühlt. | 95 % der Anfragen an `GET /assets/search` antworten in höchstens 1 s, wenn die Ergebnisse bereits im Cache liegen. Gemessen mit 20 Suchbegriffen auf dem Projektserver; die Ergebnisse werden vorher über den `MockMarketDataProvider` in den Cache geladen. Suchen, die erst bei Twelve Data abgefragt werden, sind nicht Teil dieser Anforderung, da deren Antwortzeit außerhalb des Einflusses des Projekts liegt (analog zu NF-01). | TF-NF-02 |

## Sicherheit

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-03 | Passwörter werden nur als gesalzener Hash gespeichert und haben mindestens 8 Zeichen. | In der Datenbank steht kein Passwort im Klartext. Die Werte haben das BCrypt-Format (`$2a$…`). `POST /auth/register` mit einem Passwort unter 8 Zeichen liefert 400 mit der Meldung aus dem API-Design. | TF-NF-03 |
| NF-04 | Jeder Nutzer kann nur seine eigenen Portfolios, Simulationen und Vergleiche sehen und ändern. | Nutzer B ruft `…/details`, `PUT` und `DELETE` mit IDs von Nutzer A auf und erhält jeweils 404. Die Daten von A bleiben unverändert. `…/list` zeigt nur eigene Einträge. | TF-NF-04 |
| NF-05 | Die Anwendung ist nur über HTTPS erreichbar, damit Zugangsdaten und Token, die im JSON-Body stehen, nicht mitgelesen werden können. | Ein Aufruf über `http://` wird auf `https://` umgeleitet. Das Zertifikat ist im Browser gültig, ohne Warnung. | TF-NF-05 |
| NF-06 | Der Schlüssel für die Finanzdaten-API (Twelve Data) liegt nur im Backend und gelangt nie in den Browser oder ins Repository. | Im Netzwerk-Tab des Browsers und im Frontend-Code taucht der Key nicht auf. `.env` steht in `.gitignore`, es gibt nur eine `.env.example` ohne echten Wert. In der CI liegt der Key als GitHub Secret. | TF-NF-06 |
| NF-16 | Eine Anmeldung ist begrenzt gültig und lässt sich beenden. | `expiresAt` liegt 24 h nach dem Login. Ein abgelaufener Token liefert 401. Nach `POST /auth/logout` liefert derselbe Token bei jedem geschützten Endpunkt 401. | TF-NF-16 |

## Zuverlässigkeit

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-07 | Fällt die Finanzdaten-API aus, arbeitet die Anwendung mit den Daten aus dem Cache weiter, statt abzustürzen. | Mit simuliertem Ausfall von Twelve Data (der `MockMarketDataProvider` liefert Fehler) gilt: Simulation und Suche laufen mit den Daten aus dem Cache weiter. Fehlen Kursdaten zu einer Anlage auch im Cache, kommt `503 UPSTREAM_UNAVAILABLE` mit verständlicher deutscher `message`, und das Frontend zeigt sie an. | TF-NF-07 |
| NF-08 | Mit gleichem Seed und gleichen Eingaben liefert die Simulation exakt dasselbe Ergebnis. | Zwei Aufrufe von `POST /simulations` mit identischen Parametern und `seed` = 42 liefern identische `timeline`- und `metrics`-Werte. | TF-NF-08 |

## Bedienbarkeit

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-09 | Nutzer ohne Vorkenntnisse können ohne Anleitung ein Portfolio anlegen und simulieren. | Usability-Test mit 3 fachfremden Personen: Mindestens 2 legen ohne Hilfe in unter 5 Minuten ein Portfolio an und starten eine Simulation. | TF-NF-09 |
| NF-10 | Alle Seiten sind auch auf dem Smartphone bedienbar. | Ab 360 px Breite gibt es kein seitliches Scrollen, und alle Bedienelemente sind erreichbar. Geprüft in der aktuellen Version von Chrome und Firefox (Geräteansicht). | TF-NF-10 |

## Wartbarkeit und Testbarkeit

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-11 | Die Kernlogik (Simulation und Portfolio-Prüfung) ist durch automatisierte Tests abgedeckt. | Die Zeilenabdeckung der Kernlogik liegt bei mindestens 80 %, gemessen mit JaCoCo in der CI-Pipeline des Backends. Sie wird in jedem Sprint Review berichtet (Qualitätsmetrik). | TF-NF-11 |
| NF-12 | Das Backend greift nur über eine Schnittstelle auf Kursdaten zu, sodass der Anbieter tauschbar ist. | Alle Tests laufen ohne Internetzugang mit einer Attrappe der Datenquelle (`MockMarketDataProvider`). Ein Anbieterwechsel betrifft nur die Implementierung dieser Schnittstelle. | TF-NF-12 |

## Implementierung, Recht, Schnittstelle

| NF | Anforderung | Abnahmekriterium | Testfall |
|---|---|---|---|
| NF-13 | Alle Komponenten lassen sich auf dem Server mit einem einzigen Befehl starten. | Auf dem Server startet `docker compose up -d` Frontend, Backend und Datenbank. Danach ist die Anwendung ohne weitere manuelle Schritte erreichbar. | TF-NF-13 |
| NF-14 | Die Anwendung grenzt sich von Anlageberatung ab und speichert nur nötige personenbezogene Daten. | (a) Jede Ergebnisseite (Simulation, Vergleich, Gastmodus) zeigt den Hinweis, dass es sich um keine Anlageberatung handelt. (b) Ein Nutzerkonto speichert nur E-Mail-Adresse und Passwort-Hash. | TF-NF-14 |
| NF-15 | Frontend und Backend kommunizieren ausschließlich über die im API-Design beschriebene JSON-Schnittstelle unter `/api/v1`. | Jeder Endpunkt entspricht in Pfad, Methode und Feldern dem API-Design. Alle Fehler haben das Format `{ "error": { "code", "message", "details" } }`. | TF-NF-15 |

## Änderungen in Version 2 (02.10.2026)

Anlass: Entscheidungen des Product Owners vom 02.10.2026 nach Hinweisen aus dem Testkonzept.

| NF | Änderung | Grund |
|---|---|---|
| NF-01 | „bis zu 5 Anlagen“ durch „mit 5 Anlagen (Messfall)“ ersetzt; Hinweis ergänzt, dass es keine Höchstzahl an Anlagen gibt | PO-Entscheidung E3: keine Obergrenze, die Formulierung „bis zu“ klang nach einer Begrenzung. |
| NF-02 | Nur noch der Fall mit Ergebnissen im Cache (höchstens 1 s); Suchen über Twelve Data ausgenommen | PO-Entscheidung: Gegen die Testattrappe gemessen hätte der Fall ohne Cache keine Aussagekraft, gegen Twelve Data widerspräche er den synthetischen Testdaten. Gleiche Abgrenzung wie bei NF-01. |
| NF-01, NF-02, NF-07 | Einheitlich „Cache“ statt „eigene Datenbank“; Begriff oben erklärt | PO-Entscheidung: Testkonzept und API-Design verwenden „Cache“. Gemeint ist derselbe Speicher in der eigenen Datenbank. |
| NF-07 | Fehlercode `503 UPSTREAM_UNAVAILABLE` ergänzt | Abgleich mit dem Testkonzept |
| NF-06 | Anbieter „Twelve Data“ genannt | PO-Entscheidung E7 |
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
