# Glossar

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Issue** | Docs #15 (Version 1), Docs #57 (Version 2), Docs #72 (Version 3), Docs #58 (Version 4) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 4 |
| **Stand** | 11.10.2026 |
| **Grundlagen** | Kurzformat-Tabellen Sprint 1 und Sprint 2 (Version 3, E11 bis E23), Domänenmodell v3, Aktivitätsdiagramm US-04 v2, NFRs Version 3, API-Design (Stand 02.10.2026), ADR-01 bis ADR-04, Testkonzept, `simulationsmodell.md` (Version 2) |
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
| **Fächerdiagramm** | Diagramm, das die Bandbreite der simulierten Verläufe über den Anlagehorizont zeigt: einen ungünstigen (5. Perzentil), einen mittleren (50. Perzentil) und einen günstigen Verlauf (95. Perzentil). Die Fläche dazwischen weitet sich mit der Zeit wie ein Fächer. | `timeline` |
| **Gesamtrendite** | Wertentwicklung einschließlich Dividenden, die als wiederangelegt gelten. Die Kurshistorie ist dafür um Splits und Dividenden bereinigt (E20). | – |
| **Gewichtung** | Anteil einer Anlage am Portfolio in Prozent. Jede Gewichtung ist größer als 0 % und hat höchstens 2 Nachkommastellen; zusammen ergeben alle Gewichtungen 100 % (Toleranz ± 0,01 Prozentpunkte). In Ergebnissen wird der Anteil als Dezimalzahl angegeben (0,70 = 70 %). In der Simulation gilt die Gewichtung über den gesamten Anlagehorizont (siehe Rebalancing). | `weight`, `getWeights()` |
| **Handelswährung** | Währung, in der eine Anlage an ihrer Börse gehandelt wird, z. B. EUR an XETRA oder USD an der NASDAQ. Renditen werden in der Handelswährung verwendet und nicht in Euro umgerechnet; Wechselkursschwankungen werden nicht berücksichtigt (E16). | `currency` |
| **ISIN** | International Securities Identification Number: weltweit eindeutige, zwölfstellige Kennnummer eines Wertpapiers, z. B. IE00B4L5Y983. | `isin` |
| **Kennzahlen** | Werte, die das Ergebnis einer Simulation zusammenfassen, z. B. Median-Endwert, Value at Risk, maximaler Drawdown und Wahrscheinlichkeit, ein Sparziel zu erreichen. Die einzelnen Kennzahlen werden mit US-05 (Sprint 3) festgelegt und dann hier erklärt. | `metrics` |
| **Kurshistorie** | Folge der historischen Monatswerte einer Anlage. Für eine Simulation sind je Anlage mindestens 3 Jahre nötig, also mindestens 36 Monatswerte (PO-Entscheidung E8). | – |
| **Korrelation** | Maß dafür, wie stark sich die Kurse zweier Anlagen gemeinsam bewegen. Die Simulation berücksichtigt sie **nicht**: Jede Anlage wird unabhängig von den anderen simuliert (E18). Bei Anlagen, die sich meist gleichläufig bewegen, wird die Schwankung des Portfolios dadurch unterschätzt. | – |
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
| **Vergleich** | Gegenüberstellung von zwei oder mehr Anlagen oder Portfolios unter gleichen Eingaben. Alle Varianten werden mit denselben Zufallszahlen berechnet, damit Unterschiede auf die Zusammensetzung zurückgehen und nicht auf Zufall (Kern-Story US-07). | `comparisons` |
| **Verlaufspunkt** | Werte des 5., 50. und 95. Perzentils für einen Monat. Ein Fächerdiagramm besteht aus einem Verlaufspunkt je Monat, von Monat 0 bis zum Ende des Anlagehorizonts. | Eintrag in `timeline` |
| **Volatilität** | Maß dafür, wie stark der Kurs einer Anlage schwankt, als Jahreswert geschätzt aus der Kurshistorie im Schätzzeitraum. Hohe Volatilität bedeutet eine breitere Bandbreite möglicher Ergebnisse. | `annualVolatility` |

## Anwendung und Technik

| Begriff | Erklärung | Im Code / in der API |
|---|---|---|
| **Anmeldung** | Zeitraum, in dem ein registrierter Nutzer nach Eingabe von E-Mail-Adresse und Passwort geschützte Funktionen nutzen kann. Sie gilt 24 Stunden oder bis zum Abmelden (NF-16). Technisch wird sie durch einen Token nachgewiesen. | `POST /auth/login`, `/auth/logout` |
| **API-Key** | Geheimer Zugangsschlüssel für die Finanzdaten-API. Er liegt nur im Backend und gelangt nie in den Browser oder ins Repository (NF-06). | Umgebungsvariable |
| **Backend** | Serverseitiger Teil von LookUpStock: Prüfregeln, Simulation, Datenbank und Zugriff auf die Finanzdaten-API. | Repository `Look-Up-Stock-BackEnd` |
| **Beispielsimulation** | Simulation mit vorgegebenen Beispielportfolios, die ein Gast ohne Konto ausprobieren kann. Ergebnisse werden nicht gespeichert (US-09). | `/demo/simulations` |
| **Cache** | In der eigenen Datenbank gespeicherte Kursdaten und Suchergebnisse. Sie werden verwendet, solange sie jünger als 24 Stunden sind, auch wenn die Finanzdaten-API nicht erreichbar ist (NF-07). | Cache im `MarketDataProvider` |
| **Finanzdaten-API** | Externer Dienst, der Stammdaten und historische Kursdaten liefert. Im Projekt ist das Twelve Data (PO-Entscheidung E7). | Interface `MarketDataProvider` |
| **Frontend** | Teil von LookUpStock, der im Browser läuft: Suchfeld, Portfolio-Editor, Eingabemaske und Fächerdiagramm. | Repository `Look-Up-Stock-FrontEnd` |
| **Gast** | Person ohne Nutzerkonto oder ohne gültige Anmeldung, siehe Akteursbeschreibung (`akteure.md`). | – |
| **Geschützte Funktion** | Funktion, die nur mit gültiger Anmeldung nutzbar ist, z. B. Portfolio erstellen oder Simulation durchführen. Öffentlich sind nur Registrieren, Anmelden, Anlagen suchen und die Beispielsimulation. | Feld `token` im Request-Body |
| **Mock** | Testattrappe, die eine echte Komponente nachahmt. Der Mock der Finanzdaten-API liefert feste, künstliche Kursreihen, damit Tests ohne Internet und mit immer gleichen Ergebnissen laufen (NF-12). | `MockMarketDataProvider` |
| **Nutzerkonto** | Konto eines registrierten Nutzers, bestehend aus E-Mail-Adresse und Passwort. Die E-Mail-Adresse ist eindeutig, Groß- und Kleinschreibung spielen keine Rolle (E11). Das Passwort hat mindestens 8 Zeichen und wird nur als Hash gespeichert (NF-03). | `User`, Tabelle `app_user` |
| **Passwort-Hash** | Aus dem Passwort berechneter Wert, aus dem sich das Passwort nicht zurückrechnen lässt. Bei der Anmeldung wird das eingegebene Passwort erneut gehasht und verglichen (NF-03). | BCrypt, `password_hash` |
| **Registrierter Nutzer** | Person mit Nutzerkonto, siehe Akteursbeschreibung (`akteure.md`). Im Domänenmodell als „Nutzer“ bezeichnet. | `User` |
| **Token** | Nachweis der Anmeldung, den das System nach dem Login ausgibt. Er wird bei geschützten Anfragen mitgeschickt und ist 24 Stunden gültig (NF-16). | `token` |
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
