# 2 API-Design

Das vollständige API-Design mit allen Ein- und Ausgaben, Fehlerfällen und Beispielen steht in `api/api-design.md` im Docs-Repository. Dieses Kapitel beschreibt den Aufbau, die Entscheidungen dahinter und was das Team beim Entwerfen und Ändern der Schnittstelle gelernt hat.

## 2.1 Vorgehen: Vertrag vor dem Code

Das API-Design entstand in Sprint 1 vor der ersten Zeile Backend-Code und lag am 02.10.2026 als erste Fassung vor. Jeder Endpunkt ist in einer Tabelle mit Methode, Ressource, eingehenden Daten (Data In) und ausgehenden Daten mit allen Statuscodes (Data Out) beschrieben. Frontend und Backend konnten so parallel entwickelt werden, weil beide Seiten dieselben Pfade, Felder und Fehlertexte aus dem Dokument kannten.

Der Vertrag hatte aber eine Schwäche: Er war ein Textdokument, gegen das nichts automatisch geprüft wurde. Ein Abgleich des Product Owners in Sprint 2 fand mehrere Abweichungen zwischen Dokument und Code, zum Beispiel eine Gültigkeit des Tokens von 7 Tagen statt der festgelegten 24 Stunden und abweichende Fehlertexte. Im selben Sprint entfernte ein Merge den Pfad `/api/v1` aus allen Aufrufen des Frontends. Das Team reagierte organisatorisch: Fachliche Regeln werden als nummerierte Entscheidungen des Product Owners (E1 bis E47) festgehalten und jede Schnittstellenänderung im Dokument mit Datum und Issue vermerkt. In einem künftigen Projekt würde das Team die Schnittstelle zusätzlich maschinenlesbar beschreiben, etwa als OpenAPI-Spezifikation, und Abweichungen mit Vertragstests in der CI finden statt in der manuellen Abnahme.

## 2.2 Ressourcen und Endpunkte

Die öffentliche Schnittstelle des core-service liegt unter der Basis-URL `/api/v1` und verwendet JSON. Ressourcen sind Substantive im Plural; die Aktion ergibt sich aus der HTTP-Methode.

| Ressource | Endpunkte | Anmeldung |
|---|---|---|
| Anmeldung | `POST /auth/register`, `POST /auth/login`, `POST /auth/logout` | nur Abmelden |
| Anlagen | `GET /assets/search?q=&limit=`, `GET /assets/{assetId}` | nein |
| Portfolios | `POST /portfolios`, `GET /portfolios`, `GET /portfolios/{id}`, `PUT /portfolios/{id}`, `DELETE /portfolios/{id}` | ja |
| Simulationen | `POST /simulations` (berechnen; mit `save: true` zusätzlich speichern), `GET /simulations`, `GET /simulations/{id}`, `DELETE /simulations/{id}` | ja |
| Vergleiche | `POST /comparisons` | ja |
| Gastmodus | `GET /demo/portfolios`, `POST /demo/simulations` | nein |
| Auswertungen | `GET /reports/usage?month=`, `GET /reports/usage/export?month=` | ja, Rolle `MANAGER` |
| GraphQL | `POST /graphql` mit Queries und Mutations | je nach Operation |

Eine bewusste Entscheidung betrifft die Berechnungen. Eine Simulation oder ein Vergleich ist kein bloßes Lesen vorhandener Daten, sondern erzeugt ein neues Ergebnis aus umfangreichen Eingaben (Portfolio, Startkapital, Sparrate, Horizont, Pfade, Seed, Zielbetrag). Deshalb laufen beide als `POST` auf eine eigene Ressource. Eine Simulation wird nur gespeichert, wenn der Nutzer das ausdrücklich verlangt; die Antwort ist dann `201 Created` statt `200 OK`. Diese Kombination aus Berechnen und optionalem Speichern in einem Endpunkt hielt das Frontend einfach, vermischt aber zwei Zuständigkeiten in einer Anfrage. Ein getrennter Endpunkt zum Speichern eines bereits berechneten Ergebnisses wäre semantisch klarer gewesen.

Die Beispielsimulation für Gäste erhielt einen eigenen Pfad `/demo`, statt die Simulation optional ohne Anmeldung zu erlauben. So bleiben die Regeln je Endpunkt eindeutig: `/simulations` verlangt immer eine Anmeldung, `/demo` nie, und für Gäste gilt eine eigene Obergrenze von 10.000 Pfaden.

**Statuscodes und Fehler.** Alle Fehler haben das Format `{ "error": { "code", "message", "details" } }`. `message` ist ein deutscher Text, den das Frontend unverändert anzeigt; dadurch liegen die Prüfregeln und ihre Meldungen an einer Stelle im Backend. Das Team unterscheidet `400` (ungültige Eingabe), `401` (nicht angemeldet), `403` (angemeldet, aber ohne Berechtigung), `404` (nicht vorhanden oder fremd), `409` (E-Mail bereits registriert), `422` (Eingabe gültig, aber nicht verarbeitbar, z. B. weniger als drei Jahre Kurshistorie) und `503` (Kursdaten nicht verfügbar). Zwei Entscheidungen waren nicht selbstverständlich: Fremde Portfolios liefern `404` statt `403`, damit nicht erkennbar ist, ob eine ID überhaupt existiert (NF-04). Bei den Auswertungen ist es umgekehrt: Wer ohne die Rolle Geschäftsführung zugreift, erhält `403`, weil keine fremde Ressource abgefragt wird, sondern eine Berechtigung fehlt (E40).

## 2.3 Zustandslosigkeit: der größte Umbau der Schnittstelle

Nach REST enthält jede Anfrage alle Informationen, die der Server zu ihrer Bearbeitung braucht; der Server hält keinen Sitzungszustand zwischen Anfragen (Fielding 2000, Kap. 5.1.3). Die erste Fassung der Anmeldung (ADR-04, Sprint 2) erfüllte das zwar formal, weil jede Anfrage das Token mitschickte, aber sie schickte es im JSON-Body. Da Browser bei `GET` keinen Body senden, mussten lesende Abfragen mit Anmeldung als `POST` auf eigene Pfade ausweichen: `POST /portfolios/list`, `POST /portfolios/{id}/details`, `POST /simulations/list`. Selbst `DELETE` und `POST /auth/logout` brauchten nur wegen des Tokens einen Body. Das Token wurde an 10 Stellen in den Controllern einzeln geprüft.

Die Rückmeldung des Dozenten in Sprint 4 machte deutlich, dass diese Lösung die Semantik von HTTP verletzt: `GET` ist sicher und idempotent, `POST` nicht (RFC 9110, Abschn. 9.2). Mit ADR-06 wurde die Schnittstelle umgebaut:

| Vorher (ADR-04) | Nachher (ADR-06) |
|---|---|
| `POST /portfolios/list` mit `{ "token": "…" }` | `GET /portfolios` mit Header `Authorization: Bearer <token>` |
| `POST /portfolios/{id}/details` | `GET /portfolios/{id}` |
| `DELETE /portfolios/{id}` mit Body | `DELETE /portfolios/{id}` ohne Body |
| Token-Prüfung in jedem Controller | zentrale Auswertung des Headers (`CurrentUserArgumentResolver`, Parameter `@CurrentUser`) |

Das Token steht seitdem im Header nach dem Bearer-Schema (RFC 6750); Bodys enthalten nur noch fachliche Daten. Der Umbau kostete in Sprint 4 Backend, Frontend, Testfälle und Diagramme. Die vorhandenen Controller-Tests sicherten jede Änderung ab; die Abnahme Sprint 4 bestätigte den Umbau (AT-ADR6-01 bis AT-ADR6-05 bestanden).

Objektiv bleibt eine Einschränkung: Das Token ist ein zufälliger Wert, den der Server in der Tabelle `auth_token` nachschlägt. Der Server hält damit Zustand über Anmeldungen, wenn auch keinen Gesprächszustand zwischen Anfragen. Ein signiertes Token wie ein JSON Web Token wäre vollständig zustandslos prüfbar gewesen, hätte aber das sofortige Sperren beim Abmelden (NF-16) erschwert. Das Team hat diesen Zielkonflikt bewusst zugunsten des sofortigen Abmeldens entschieden. Reproduzierbar ist dagegen jede Berechnung: Bei gleichem Seed und gleichen Eingaben liefert `POST /simulations` dasselbe Ergebnis (NF-08), unabhängig davon, welche Anfragen vorher kamen.

Die wichtigste Erkenntnis aus diesem Umbau: Die Anmeldung ist eine Querschnittsentscheidung, die jeden Endpunkt berührt. Sie gehört an den Anfang des API-Designs und nicht in Sprint 2, wenn bereits Endpunkte existieren. Würde das Team neu beginnen, stünde der `Authorization`-Header vom ersten Entwurf an fest.

## 2.4 GraphQL und interne Schnittstellen

GraphQL bietet über `POST /graphql` vergleichbare Funktionen wie REST: Queries für Anlagensuche, Portfolios und gespeicherte Simulationen (Sprint 4) und Mutations zum Anlegen, Ändern und Löschen von Portfolios, zum Simulieren und zum Vergleichen (Sprint 5). Beide Schnittstellen rufen dieselben Services auf. Prüfregeln, Meldungen und Anmeldung sind deshalb identisch; ein Fehler steht bei GraphQL im Feld `errors` des Ergebnisses (GraphQL Specification 2021, Abschn. 7.1.2), mit demselben Text wie bei REST und dem Code in `extensions.code`; die HTTP-Antwort ist dabei `200 OK`. Die Auswertungen gibt es bewusst nur über REST, weil der CSV-Download ein Dateiformat und keine Graphstruktur liefert.

Mit der Aufteilung in Dienste (ADR-07) kamen zwei interne Schnittstellen hinzu: die Endpunkte des marktdaten-service (`GET /internal/search`, `GET /internal/prices`, geschützt mit dem Header `X-Internal-Api-Key`) und die Themen der Queue (`nutzung.ereignis`, `kursdaten.vorladen`) mit festgelegtem JSON-Inhalt. Beide sind im Abschnitt „Interne Schnittstellen“ des API-Designs beschrieben. Selbstkritisch ist festzuhalten, dass diese Verträge in Sprint 5 parallel zur Umsetzung entstanden und nicht, wie die Guidelines es verlangen, vorher definiert waren. Für die öffentliche Schnittstelle hat das Vorgehen „Vertrag zuerst“ funktioniert; für die internen Schnittstellen fehlte dafür nach der späten Architekturentscheidung die Zeit.

## 2.5 Zusammenfassung der Erkenntnisse

| Entscheidung | Bewertung im Rückblick | Was das Team künftig anders machen würde |
|---|---|---|
| API-Design als Dokument vor dem Code | richtig, ermöglichte paralleles Arbeiten | maschinenlesbare Spezifikation und Vertragstests in der CI |
| Token im Body (ADR-04) | falsch, verletzte die HTTP-Semantik und erzwang einen Umbau | Anmeldung über den `Authorization`-Header vom ersten Entwurf an |
| Serverseitig gespeichertes Token | vertretbar, bewusster Zielkonflikt mit dem sofortigen Abmelden | Entscheidung früher als ADR dokumentieren |
| Deutsche Fehlertexte aus dem Backend | richtig, eine Quelle für Regeln und Meldungen | beibehalten |
| Berechnen und Speichern in einem Endpunkt | funktioniert, aber semantisch unscharf | Speichern als eigener Endpunkt |
| Interne Schnittstellen parallel zum Code | erzwungen durch die späte Architekturentscheidung | interne Verträge zusammen mit der Architektur festlegen |
