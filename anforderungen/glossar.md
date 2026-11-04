# Glossar

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #15 (Version 1), Docs #57 (Version 2), Docs #72 (Version 3), Docs #58 (Version 4), Docs #83 (Version 5), Docs #131 (Version 6); Version 7: Anpassung an die Umsetzung in Sprint 5; Version 8: Zählung gespeicherter Simulationen (Docs #213) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 8 |
| **Stand** | 04.11.2026 |
| **Grundlagen** | Kurzformat-Tabellen Sprint 1 und Sprint 2 (Version 3, E11 bis E23), Domänenmodell v3, Aktivitätsdiagramm US-04 v2, NFRs Version 3, API-Design (Stand 02.10.2026), ADR-01 bis ADR-04, Testkonzept, `simulationsmodell.md` (Version 2), Kurzformat-Tabellen Sprint 3 (E24 bis E35), ADR-05, ADR-06, Change Request CR-01 und Kurzformat-Tabellen Sprint 4 (E36 bis E43); ab Version 7 zusätzlich ADR-07, ADR-08, API-Design (Stand 01.11.2026) und die Umsetzung BE #105 bis #114, FE #51 bis #53 |
| **KI-Nutzung** | Mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Das Glossar erklärt Begriffe, die für eine einheitliche Beschreibung nötig sind. Es richtet sich an Entwickler, Fachnutzer und Fachfremde. Die Spalte „Im Code / in der API“ ordnet die deutschen Fachbegriffe den englischen Namen in Code und Schnittstelle zu, damit Dokumentation und Prototyp dieselbe Sprache sprechen. Das Glossar wächst mit jedem Sprint; Begriffe späterer Stories werden ergänzt, sobald sie festgelegt sind.

## Geldanlage und Simulation

| Begriff | Erklärung | Im Code / in der API |
|---|---|---|
| **Aktie** | Anteil an einem Unternehmen, der an einer Börse gehandelt wird. | `type` |
| **Anlage** | Oberbegriff für eine Aktie oder einen ETF, die in LookUpStock gesucht und in ein Portfolio aufgenommen werden kann. | `Asset`, `assetId` |
| **Anlagehorizont** | Zeitraum in ganzen Jahren, über den die Entwicklung eines Portfolios simuliert wird. Erlaubt sind 1 bis 40 Jahre. | `horizonYears` |
| **Börse** | Handelsplatz, an dem eine Anlage gehandelt wird, z. B. XETRA. | `exchange` |
| **ETF** | Exchange Traded Fund: ein börsengehandelter Fonds, der meist einen Index nachbildet, z. B. den MSCI World. | `type` |
| **Einzahlungen** | Summe aus Startkapital und allen Sparraten über den Anlagehorizont: Startkapital + Anlagehorizont × 12 × Sparrate (E19). | `totalContributions` |
| **Endwert** | Portfoliowert am Ende des Anlagehorizonts. Angezeigt wird er im ungünstigen (5. Perzentil), mittleren (Median) und günstigen Verlauf (95. Perzentil); die Werte stimmen mit dem letzten Verlaufspunkt des Fächerdiagramms überein (E24). | `p5EndValue`, `medianEndValue`, `p95EndValue` |
| **Fächerdiagramm** | Diagramm, das die Bandbreite der simulierten Verläufe über den Anlagehorizont zeigt: einen ungünstigen (5. Perzentil), einen mittleren (50. Perzentil) und einen günstigen Verlauf (95. Perzentil). Die Fläche dazwischen weitet sich mit der Zeit wie ein Fächer. | `timeline` |
| **Gesamtrendite** | Wertentwicklung einschließlich Dividenden, die als wiederangelegt gelten. Die Kurshistorie ist dafür um Splits und Dividenden bereinigt (E20). | – |
| **Gewichtung** | Anteil einer Anlage am Portfolio in Prozent. Jede Gewichtung ist größer als 0 % und hat höchstens 2 Nachkommastellen; zusammen ergeben alle Gewichtungen 100 % (Toleranz ± 0,01 Prozentpunkte). In Ergebnissen wird der Anteil als Dezimalzahl angegeben (0,70 = 70 %). In der Simulation gilt die Gewichtung über den gesamten Anlagehorizont (siehe Rebalancing). | `weight`, `getWeights()` |
| **Handelswährung** | Währung, in der eine Anlage an ihrer Börse gehandelt wird, z. B. EUR an XETRA oder USD an der NASDAQ. Renditen werden in der Handelswährung verwendet und nicht in Euro umgerechnet; Wechselkursschwankungen werden nicht berücksichtigt (E16). | `currency` |
| **ISIN** | International Securities Identification Number: weltweit eindeutige, zwölfstellige Kennnummer eines Wertpapiers, z. B. IE00B4L5Y983. | `isin` |
| **Kennzahlen** | Werte, die das Ergebnis einer Simulation zum Ende des Anlagehorizonts zusammenfassen (US-05, E24): Einzahlungen, Endwert im ungünstigen, mittleren und günstigen Verlauf, Value at Risk (95 %), maximaler Drawdown im Median und, falls ein Zielbetrag angegeben ist, die Zielwahrscheinlichkeit. Sie stehen unter dem Fächerdiagramm. | `metrics` |
| **Kurshistorie** | Folge der historischen Monatswerte einer Anlage. Für eine Simulation sind je Anlage mindestens 3 Jahre nötig, also mindestens 36 Monatswerte (PO-Entscheidung E8). | – |
| **Korrelation** | Maß dafür, wie stark sich die Kurse zweier Anlagen gemeinsam bewegen. Die Simulation berücksichtigt sie **nicht**: Jede Anlage wird unabhängig von den anderen simuliert (E18). Bei Anlagen, die sich meist gleichläufig bewegen, wird die Schwankung des Portfolios dadurch unterschätzt. | – |
| **Maximaler Drawdown** | Größter zwischenzeitlicher Rückgang des Portfoliowerts von einem bisherigen Höchststand, in Prozent. Beispiel: Steigt das Portfolio auf 120.000 € und fällt danach auf 90.000 €, beträgt der Drawdown 25 %. Er wird für jeden Simulationspfad bestimmt; angezeigt wird der Median über alle Pfade. Die laufenden Einzahlungen sind enthalten und mildern Rückgänge ab (E26). | `maxDrawdownMedian` |
| **Median** | Mittlerer Wert: Die Hälfte der simulierten Verläufe liegt darüber, die Hälfte darunter. Entspricht dem 50. Perzentil. | `p50` |
| **Monatswert** | Schlusskurs einer Anlage am Ende eines Monats, bereinigt um Splits und Dividenden (E20). LookUpStock rechnet mit Monatswerten, weil auch die Simulation in Monatsschritten rechnet. Nur abgeschlossene Monate zählen; der laufende Monat wird nicht verwendet (E22). | – |
| **Monte-Carlo-Simulation** | Verfahren, das sehr viele mögliche Zukunftsverläufe mit Zufallszahlen erzeugt. Grundlage sind die aus der Vergangenheit geschätzte Rendite und Volatilität. Statt einer einzigen Prognose zeigt das Ergebnis, wie breit die möglichen Entwicklungen streuen. | `MonteCarloEngine` |
| **Nominal** | Ohne Berücksichtigung von Inflation. Die Simulation rechnet nominal und ohne Kosten und Steuern (E21). | – |
| **Perzentil** | Wert, unter dem ein bestimmter Anteil aller simulierten Verläufe liegt. Beim 5. Perzentil liegen 5 % der Verläufe darunter (ungünstiger Fall), beim 95. Perzentil 95 % (günstiger Fall). | `p5`, `p50`, `p95` |
| **Portfolio** | Zusammenstellung aus einer oder mehreren Anlagen mit ihren Gewichtungen, z. B. 70 % MSCI World und 30 % Schwellenländer. Ein Portfolio hat einen Namen mit höchstens 100 Zeichen. | `Portfolio` |
| **Position** | Eine Anlage innerhalb eines Portfolios zusammen mit ihrer Gewichtung. Jede Anlage kommt in einem Portfolio höchstens einmal vor. | `Position` |
| **Rebalancing** | Zurücksetzen der Anteile im Portfolio auf die festgelegten Gewichtungen. In der Simulation geschieht das jeden Monat; die Sparrate wird nach denselben Gewichtungen aufgeteilt (E15). | – |
| **Rendite** | Durchschnittliche jährliche Wertentwicklung einer Anlage als Gesamtrendite, geschätzt aus ihrer Kurshistorie im Schätzzeitraum. In der API als Dezimalzahl (0,078 = 7,8 % pro Jahr). Berechnung siehe `simulationsmodell.md`. | `annualReturn` |
| **Schätzung** | Die aus der Kurshistorie ermittelten Werte Rendite und Volatilität einer Anlage. Sie sind die Grundlage der Simulation. | `estimates`, `ParameterEstimator` |
| **Schätzzeitraum** | Zeitraum der Kurshistorie, aus dem Rendite und Volatilität geschätzt werden. Er umfasst die letzten bis zu 120 abgeschlossenen Monate (10 Jahre), gilt gemeinsam für alle Anlagen einer Simulation und ist so lang wie die kürzeste vorhandene Kurshistorie (E17). | `historyFrom`, `historyTo` |
| **Seed** | Startwert für die Zufallszahlen einer Simulation. Mit demselben Seed und denselben Eingaben liefert die Simulation exakt dasselbe Ergebnis (NF-08). Ohne Angabe erzeugt das System einen Seed und zeigt ihn an. | `seed` |
| **Simulationspfad** | Ein einzelner simulierter Verlauf des Portfoliowerts, Monat für Monat über den gesamten Anlagehorizont. Die Monatsrenditen sind normalverteilt; fällt der Wert rechnerisch unter 0, wird er auf 0 gesetzt (E23). Standardmäßig werden 10.000 Pfade berechnet, wählbar sind 1.000 bis 50.000. | `paths` |
| **Sparrate** | Betrag in Euro, der jeden Monat zusätzlich in das Portfolio eingezahlt wird, jeweils am Monatsende, erstmals in Monat 1 (E19). | `monthlyContribution` |
| **Startkapital** | Betrag in Euro, der zu Beginn einmalig angelegt wird. Startkapital und Sparrate dürfen nicht beide 0 sein. | `startCapital` |
| **Ticker** | Kurzes Börsenkürzel einer Anlage, z. B. EUNL. | `symbol` |
| **Vergleich** | Gegenüberstellung von zwei Einträgen unter gleichen Eingaben (Kern-Story US-07); ein Eintrag ist ein eigenes Portfolio oder eine einzelne Anlage, die wie ein Portfolio mit 100 % gerechnet wird (E30). Beide Einträge werden mit denselben Zufallszahlen und einem gemeinsamen Schätzzeitraum berechnet, damit Unterschiede auf die Zusammensetzung zurückgehen und nicht auf Zufall oder unterschiedliche Daten (E29). Zusätzlich wird angezeigt, mit welcher Wahrscheinlichkeit der erste Eintrag am Ende mehr wert ist als der zweite (E31). Mehr als zwei Einträge (US-10) werden nicht umgesetzt. | `comparisons`, `ComparisonService`, `probabilityAOutperformsB` |
| **Verlaufspunkt** | Werte des 5., 50. und 95. Perzentils für einen Monat. Ein Fächerdiagramm besteht aus einem Verlaufspunkt je Monat, von Monat 0 bis zum Ende des Anlagehorizonts. | Eintrag in `timeline` |
| **Value at Risk (95 %)** | Betrag, um den der Endwert im ungünstigen Verlauf (5. Perzentil) unter den Einzahlungen liegt. Beispiel: 58.000 € eingezahlt, ungünstiger Endwert 52.000 €, Value at Risk 6.000 €. Liegt der ungünstige Endwert über den Einzahlungen, beträgt er 0 €. Vereinfachte Form des Value at Risk aus dem Risikomanagement, bezogen auf die eingezahlten Euro (E25). | `valueAtRisk95` |
| **Volatilität** | Maß dafür, wie stark der Kurs einer Anlage schwankt, als Jahreswert geschätzt aus der Kurshistorie im Schätzzeitraum. Hohe Volatilität bedeutet eine breitere Bandbreite möglicher Ergebnisse. | `annualVolatility` |
| **Zielbetrag** | Freiwillig angegebener Betrag in Euro, den der Nutzer am Ende des Anlagehorizonts erreichen möchte, z. B. 100.000 €. Er muss größer als 0 sein (E27). | `targetAmount` |
| **Zielwahrscheinlichkeit** | Anteil der Simulationspfade, deren Endwert mindestens den Zielbetrag erreicht. Ohne Zielbetrag wird sie nicht angezeigt (E27). | `targetProbability` |

## Anwendung und Technik

| Begriff | Erklärung | Im Code / in der API |
|---|---|---|
| **Anmeldung** | Zeitraum, in dem ein registrierter Nutzer nach Eingabe von E-Mail-Adresse und Passwort geschützte Funktionen nutzen kann. Sie gilt 24 Stunden oder bis zum Abmelden (NF-16). Technisch wird sie durch einen Token nachgewiesen. | `POST /auth/login`, `/auth/logout` |
| **API-Key** | Geheimer Zugangsschlüssel für die Finanzdaten-API. Er liegt nur im Backend, seit ADR-07 nur beim marktdaten-service, und gelangt nie in den Browser oder ins Repository (NF-06). | Umgebungsvariable `TWELVE_DATA_API_KEY` |
| **Backend** | Serverseitiger Teil von LookUpStock: Prüfregeln, Simulation, Datenbank und Zugriff auf die Finanzdaten-API. Seit ADR-07 aufgeteilt in zwei Dienste: core-service und marktdaten-service. | Repository `Look-Up-Stock-BackEnd`, Ordner `core-service/` und `marktdaten-service/` |
| **Beispielsimulation** | Simulation mit vorgegebenen Beispielportfolios, die ein Gast ohne Konto ausprobieren kann. Ergebnisse werden nicht gespeichert (US-09). | `/demo/simulations` |
| **Cache** | Zwischengespeicherte Kursdaten und Suchergebnisse. Seit ADR-07 liegen beide im marktdaten-service: Kursdaten in der eigenen Datenbank (Schema `marktdaten`), Suchergebnisse im Arbeitsspeicher. Sie werden verwendet, solange sie jünger als 24 Stunden sind; Kursdaten auch dann, wenn die Finanzdaten-API nicht erreichbar ist (NF-07). Fehlende Kursdaten werden nach dem Anlegen oder Ändern eines Portfolios über die Queue vorgeladen. | `PriceService`, Tabelle `marktdaten.price_cache`; `SearchCache` (bis Sprint 4 `PriceCache`, Tabelle `price_history` im Backend) |
| **core-service** | Dienst mit den Funktionen von LookUpStock: Anmeldung, Portfolios, Simulation, Vergleich, gespeicherte Simulationen, Beispielsimulation und Auswertungen. Einziger Ansprechpartner des Frontends über REST und GraphQL (ADR-07). | Ordner `core-service/`, Heroku-App `lookupstock` |
| **CSV-Export** | Herunterladen der Nutzungskennzahlen eines Monats als CSV-Datei, die sich in einer Tabellenkalkulation öffnen lässt. Je Kennzahl eine Zeile mit Monat, Bezeichnung und Wert, getrennt durch Semikolon (US-12, E41). | `GET /reports/usage/export`, `UsageReportCsvWriter`, Datei `lookupstock-nutzung-JJJJ-MM.csv` |
| **Dashboard** | Seite, auf der die Geschäftsführung die Nutzungskennzahlen eines gewählten Kalendermonats sieht (US-11). Nur mit der Rolle Geschäftsführung erreichbar (E40). | `dashboard.html`, `GET /reports/usage` |
| **Finanzdaten-API** | Externer Dienst, der Stammdaten und historische Kursdaten liefert. Im Projekt ist das Twelve Data (PO-Entscheidung E7). Nur der marktdaten-service greift darauf zu (ADR-07). | Interface `MarketDataProvider`; im marktdaten-service `TwelveDataProvider`, im core-service `MarktdatenClient` |
| **Frontend** | Teil von LookUpStock, der im Browser läuft: Suchfeld, Portfolio-Editor, Eingabemaske und Fächerdiagramm. Es besteht aus der MPA und dem SPA-Touchpoint. | Repository `Look-Up-Stock-FrontEnd` |
| **Gespeicherte Simulation** | Simulation, die ein registrierter Nutzer unter einem Namen mit höchstens 100 Zeichen gespeichert hat (US-08). Gespeichert werden Parameter, Schätzungen, Verlauf und Kennzahlen; beim Aufrufen wird das Ergebnis unverändert angezeigt, ohne Neuberechnung (E32). Wird das zugehörige Portfolio gelöscht, wird sie mitgelöscht (E28). | `SavedSimulation`, Tabelle `simulation` |
| **Geschäftsführung** | Leitung des Unternehmens, das LookUpStock betreibt. Sie wertet die Nutzung des Systems aus (US-11, US-12), siehe Akteursbeschreibung (`akteure.md`). Technisch ein Nutzerkonto mit der Rolle Geschäftsführung. | Rolle `MANAGER` (`UserRole`) |
| **Gast** | Person ohne Nutzerkonto oder ohne gültige Anmeldung, siehe Akteursbeschreibung (`akteure.md`). | – |
| **Geschützte Funktion** | Funktion, die nur mit gültiger Anmeldung nutzbar ist, z. B. Portfolio erstellen oder Simulation durchführen. Öffentlich sind nur Registrieren, Anmelden, Anlagen suchen und die Beispielsimulation. | Header `Authorization: Bearer <token>`, `@CurrentUser` (ADR-06) |
| **GraphQL** | Zweite Schnittstelle neben REST mit vergleichbarem Funktionsumfang: Abfragen (Queries) zum Lesen und Mutations zum Anlegen, Ändern, Löschen, Simulieren und Vergleichen. Gleiche Prüfregeln, Meldungen und Anmeldung wie REST. | `POST /graphql`, `QueryController`, `MutationController` |
| **marktdaten-service** | Dienst, der die Finanzdaten-API kapselt: Anlagensuche, historische Kurse und deren Cache. Nur für den core-service erreichbar, geschützt mit einem internen Schlüssel (ADR-07). | Ordner `marktdaten-service/`, Heroku-App `lookupstock-marktdaten`, Header `X-Internal-Api-Key` |
| **Mock** | Testattrappe, die eine echte Komponente nachahmt. Der Mock der Finanzdaten-API liefert feste, künstliche Kursreihen, damit Tests ohne Internet und mit immer gleichen Ergebnissen laufen (NF-12). | `MockMarketDataProvider` |
| **MPA (Multi-Page-Application)** | Touchpoint, bei dem jede Navigation eine neue HTML-Seite lädt: Simulation, Portfolios, Vergleich, Anmeldung und Auswertungen. | `index.html`, `portfolio.html`, `compare.html`, `login.html`, `dashboard.html` |
| **Nutzerkonto** | Konto eines registrierten Nutzers, bestehend aus E-Mail-Adresse und Passwort. Die E-Mail-Adresse ist eindeutig, Groß- und Kleinschreibung spielen keine Rolle (E11). Das Passwort hat mindestens 8 Zeichen und wird nur als Hash gespeichert (NF-03). | `User`, Tabelle `app_user` |
| **Nutzungsereignis** | Eintrag, der festhält, dass eine Funktion erfolgreich genutzt wurde, z. B. eine Simulation oder eine Anmeldung. Er besteht nur aus Ereignisart und Zeitpunkt, ohne Bezug zu einem Nutzer, IP-Adresse, Suchbegriff oder Anlage (E36, E37, NF-14 b). Grundlage der Nutzungskennzahlen. Wird über die Queue übertragen und erscheint deshalb mit einigen Sekunden Verzögerung (ADR-07). | `UsageEvent`, `UsageEventType`, Tabelle `usage_event` |
| **Nutzungskennzahlen** | Zahlen zur Nutzung von LookUpStock in einem Kalendermonat: neue Registrierungen, Anmeldungen, angelegte Portfolios, durchgeführte und gespeicherte Simulationen, Vergleiche, Beispielsimulationen, Anlagensuchen und Nutzerkonten am Monatsende (E39). Eine über die Oberfläche gespeicherte Simulation zählt einmal als durchgeführte (beim Simulieren) und einmal als gespeicherte (beim Speichern), nicht doppelt als durchgeführte (E37, B8). Nicht zu verwechseln mit den Kennzahlen einer Simulation (US-05). | `UsageMetric`, `UsageReport`, `GET /reports/usage` |
| **Passwort-Hash** | Aus dem Passwort berechneter Wert, aus dem sich das Passwort nicht zurückrechnen lässt. Bei der Anmeldung wird das eingegebene Passwort erneut gehasht und verglichen (NF-03). | BCrypt, `password_hash` |
| **Queue** | Warteschlange für Nachrichten zwischen den Komponenten, damit der Absender nicht auf die Verarbeitung warten muss (asynchrone Kommunikation, ADR-07). Umgesetzt als Tabelle in der Datenbank mit den Themen Nutzungsereignis und Vorladen der Kurse. | Tabelle `queue_message`, `MessagePublisher`, Themen `nutzung.ereignis`, `kursdaten.vorladen` |
| **Registrierter Nutzer** | Person mit Nutzerkonto, siehe Akteursbeschreibung (`akteure.md`). Im Domänenmodell als „Nutzer“ bezeichnet. | `User` |
| **Rolle** | Berechtigung eines Nutzerkontos über die Funktionen des registrierten Nutzers hinaus. Bisher gibt es nur die Rolle Geschäftsführung; sie wird vom Team vergeben, nicht bei der Registrierung gewählt (E40). Ohne die Rolle liefern die Auswertungen 403. Die Rolle wird in der Datenbank gesetzt; das Frontend erhält sie bei der Anmeldung. | `UserRole` (`USER`, `MANAGER`), Spalte `role` in `app_user`, Feld `role` von `POST /auth/login` |
| **SPA (Single-Page-Application)** | Touchpoint, der einmal geladen wird und Ansichten ohne Neuladen wechselt (clientseitiges Routing). In LookUpStock eine eigene Einstiegsseite zum Ansehen und Simulieren von Portfolios und zum Aufrufen gespeicherter Simulationen (ADR-08). | `app.html`, Hash-Router `src/js/spa/router.js` |
| **Token** | Nachweis der Anmeldung, den das System nach dem Login ausgibt. Er wird bei geschützten Anfragen im Header `Authorization` mitgeschickt, nicht im Request-Body (ADR-06), und ist 24 Stunden gültig (NF-16). | `token`, Header `Authorization: Bearer <token>` |
| **Twelve Data** | Anbieter, der im Projekt als Finanzdaten-API genutzt wird. | – |

## Projektbegriffe

| Begriff | Erklärung |
|---|---|
| **Abnahmetest** | Test, mit dem der Product Owner prüft, ob eine User Story ihre Akzeptanzkriterien erfüllt. Die Abnahmetestfälle stehen im Testkonzept. |
| **Akzeptanzkriterium** | Prüfbare Bedingung, die eine User Story erfüllen muss, damit sie als fertig gilt, z. B. „Die Suche startet erst ab zwei eingegebenen Zeichen.“ |
| **Definition of Done** | Gemeinsame Regel des Teams, wann eine User Story erledigt ist: implementiert, getestet und im Repository integriert. |
| **Kern-Story** | Eine der zwei mit dem Dozenten abgestimmten User Stories, die mit Aktivitäts- und Sequenzdiagramm detailliert ausgearbeitet werden: US-04 Simulation und US-07 Vergleich. |
| **MoSCoW** | Priorisierung der User Stories in Must (zwingend), Should (wichtig), Could (bei freier Kapazität) und Won't (nicht in diesem Projekt). |
| **NFR** | Nicht-funktionale Anforderung: beschreibt, wie gut das System etwas tun muss, z. B. wie schnell. Die NFRs sind nach FURPS+ gegliedert und haben je ein messbares Abnahmekriterium. |
| **PO-Entscheidung** | Vom Product Owner festgelegte fachliche Regel, nummeriert als E1, E2 usw. Die Entscheidungen stehen in den Kurzformat-Tabellen. |
| **Product Owner** | Rolle in Scrum, die festlegt, was gebaut wird, das Product Backlog priorisiert und die Ergebnisse abnimmt. Im Projekt zugleich der Kunde. |
| **Sprint** | Fester Arbeitszeitraum in Scrum, im Projekt eine Woche. Am Ende steht ein lauffähiges Zwischenergebnis. |
| **Story Points** | Schätzung des relativen Aufwands einer User Story nach der Fibonacci-Folge (1, 2, 3, 5, 8). Sie geben die Komplexität im Vergleich zu anderen Stories an, nicht die Arbeitszeit. |
| **User Story** | Anforderung aus Sicht des Nutzers nach dem Muster „Als … möchte ich …, um …“, nummeriert als US-01 bis US-10. |

## Änderungshistorie

| Version | Datum | Änderung | Anlass |
|---|---|---|---|
| 1 | 04.10.2026 | Erstfassung | Docs #15 |
| 2 | 08.10.2026 | Neu: Anmeldung, Geschützte Funktion, Nutzerkonto, Passwort-Hash. Gast und registrierter Nutzer präzisiert. Kennzahlen: Sprint 3 statt Sprint 2 (US-05 verschoben) | Docs #57, US-01, Domänenmodell v3 |
| 3 | 08.10.2026 | Neu: Einzahlungen, Gesamtrendite, Handelswährung, Korrelation, Nominal, Rebalancing. Ergänzt: Gewichtung, Monatswert, Rendite, Schätzzeitraum, Simulationspfad, Sparrate, Volatilität. Kennzahlen: US-05 liegt in Sprint 3 | PO-Festlegungen E15 bis E23 (Docs #72) |
| 4 | 11.10.2026 | Einträge aus Version 2 (Anmeldung, Geschützte Funktion, Nutzerkonto, Passwort-Hash, Gast, Registrierter Nutzer) wiederhergestellt, die beim Zusammenführen von Version 3 verloren gegangen waren. Korrelation und Simulationspfad an die geänderten Festlegungen E18 und E23 angepasst | Docs #58, PO-Entscheidung vom 11.10.2026 |
| 5 | 18.10.2026 | Neu: Endwert, Maximaler Drawdown, Value at Risk (95 %), Zielbetrag, Zielwahrscheinlichkeit, Gespeicherte Simulation. Kennzahlen ausformuliert. Vergleich an US-07 angepasst (zwei Einträge, einzelne Anlage, gemeinsamer Schätzzeitraum, Wahrscheinlichkeit für den ersten Eintrag) | Docs #83, Festlegungen E24 bis E34 |
| 6 | 25.10.2026 | Neu: CSV-Export, Dashboard, Geschäftsführung, Nutzungsereignis, Nutzungskennzahlen, Rolle. Geändert: Token und Geschützte Funktion (Übertragung im Header, ADR-06), Vergleich (US-10 wird nicht umgesetzt), Cache (Suchergebnisse im Arbeitsspeicher, Bezeichnungen im Code) | Docs #131, Change Request CR-01, ADR-06, Sprint-Planning-Protokoll Sprint 4 |
| 7 | 01.11.2026 | Neu: core-service, GraphQL, marktdaten-service, MPA, Queue, SPA. Geändert: API-Key, Backend, Cache, Finanzdaten-API und Frontend (Aufteilung in zwei Dienste, ADR-07); Bezeichnungen im Code für CSV-Export, Dashboard, Geschäftsführung, Nutzungsereignis, Nutzungskennzahlen und Rolle aus der Umsetzung übernommen | Umsetzung Sprint 5 (BE #105 bis #114, FE #51 bis #53), ADR-07, ADR-08 |
| 8 | 04.11.2026 | Geändert: Nutzungskennzahlen (Zählung einer gespeicherten Simulation nach E37, B8) | Docs #213, BE #128 |
