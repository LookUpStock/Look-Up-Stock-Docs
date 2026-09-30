# Kurzformat-Tabellen – Sprint 1

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026) |
| **Issue** | #14 Kurzformat-Tabellen für US-02, US-03, US-04 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 30.09.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 1, API-Design (Stand 30.09.2026) |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log Nr. 14 |

Die Tabellen folgen der Vorlage zur Use-Case-Kurzbeschreibung aus der Vorlesung; das dort gestrichene Feld „Ergebnis“ entfällt. Die essenziellen Schritte sind bewusst fachlich formuliert. Der technische Ablauf inklusive Alternativszenarien wird für die Kern-Story US-04 im Aktivitäts- und Sequenzdiagramm dargestellt.

In Sprint 1 ist noch keine Anmeldung erforderlich. Mit US-01 (Sprint 2) kommt bei allen drei Stories die Vorbedingung „Nutzer ist angemeldet“ hinzu, und Portfolios werden dem angemeldeten Nutzer zugeordnet.

---

## US-02 Anlagen suchen

| **Name** | **Anlagen suchen** |
|---|---|
| **Art** | User Story (US-02) |
| **Kurzbeschreibung** | Der Nutzer sucht über Ticker, ISIN oder Namen nach Aktien und ETFs, um sie anschließend einem Portfolio hinzuzufügen. |
| **Auslöser** | Der Nutzer möchte eine bestimmte Anlage für sein Portfolio finden. |
| **Akteure** | Nutzer (primär), Finanzdaten-API (sekundär) |
| **Eingehende Information** | Suchbegriff (Ticker, ISIN oder Name, mindestens zwei Zeichen) |
| **Vorbedingungen** | Die Anwendung ist im Browser geöffnet. |
| **Nachbedingungen** | • Die Trefferliste mit Name, Ticker, Börse und Währung wird angezeigt; bei keinem Treffer erscheint ein Hinweis.<br>• Der Nutzer hat eine Anlage ausgewählt.<br>• Die ausgewählte Anlage kann in ein Portfolio übernommen werden. |
| **Essenzielle Schritte** | • Suchbegriff eingeben<br>• Passende Anlagen ermitteln<br>• Trefferliste anzeigen<br>• Anlage auswählen |

---

## US-03 Portfolio mit Gewichtungen erstellen

| **Name** | **Portfolio mit Gewichtungen erstellen** |
|---|---|
| **Art** | User Story (US-03) |
| **Kurzbeschreibung** | Der Nutzer stellt aus einer oder mehreren Anlagen ein Portfolio zusammen, vergibt einen Namen und legt für jede Anlage eine Gewichtung in Prozent fest. |
| **Auslöser** | Der Nutzer möchte eine Anlagestrategie abbilden, um sie anschließend zu simulieren und zu vergleichen. |
| **Akteure** | Nutzer |
| **Eingehende Information** | Portfolioname, ausgewählte Anlagen (aus der Anlagensuche, US-02), Gewichtung je Anlage in Prozent |
| **Vorbedingungen** | Die aufzunehmenden Anlagen wurden über die Anlagensuche (US-02) gefunden. |
| **Nachbedingungen** | • Das Portfolio ist unter seinem Namen gespeichert.<br>• Es enthält mindestens eine Anlage, jede Anlage höchstens einmal.<br>• Alle Gewichtungen sind größer als 0 % und ergeben zusammen 100 %.<br>• Das Portfolio kann für eine Simulation (US-04) ausgewählt werden. |
| **Essenzielle Schritte** | • Portfolio benennen<br>• Anlagen hinzufügen<br>• Gewichtungen festlegen<br>• Eingaben prüfen<br>• Portfolio speichern |

---

## US-04 Monte-Carlo-Simulation durchführen (Kern-Story)

| **Name** | **Monte-Carlo-Simulation durchführen** |
|---|---|
| **Art** | User Story (US-04), Kern-Story |
| **Kurzbeschreibung** | Der Nutzer gibt für ein gespeichertes Portfolio Startkapital, monatliche Sparrate und Anlagehorizont ein und kann optional die Anzahl der Simulationspfade und einen Seed festlegen. Das System schätzt aus historischen Kursdaten Rendite und Volatilität der enthaltenen Anlagen, simuliert die möglichen Wertverläufe und stellt deren Bandbreite als Fächerdiagramm dar. |
| **Auslöser** | Der Nutzer möchte wissen, wie sich sein Portfolio über einen gewählten Zeitraum entwickeln könnte. |
| **Akteure** | Nutzer (primär), Finanzdaten-API (sekundär) |
| **Eingehende Information** | Ausgewähltes Portfolio, Startkapital in Euro, monatliche Sparrate in Euro, Anlagehorizont in ganzen Jahren, optional Anzahl der Simulationspfade (Standard 10.000) und Seed, historische Kursdaten der enthaltenen Anlagen (aus der Finanzdaten-API) |
| **Vorbedingungen** | Es wurde mindestens ein Portfolio erstellt (US-03). |
| **Nachbedingungen** | • Die Simulation wurde mit der gewählten Anzahl an Pfaden berechnet.<br>• Ein Fächerdiagramm zeigt den pessimistischen (5. Perzentil), mittleren (50. Perzentil) und optimistischen (95. Perzentil) Verlauf des Portfoliowerts über den Anlagehorizont.<br>• Der verwendete Seed wird angezeigt, sodass die Simulation mit denselben Eingaben reproduziert werden kann.<br>• Das Ergebnis wird in Sprint 1 nicht gespeichert (Speichern folgt mit US-08). |
| **Essenzielle Schritte** | • Portfolio auswählen<br>• Startkapital, Sparrate und Anlagehorizont eingeben, optional Pfadanzahl und Seed festlegen<br>• Eingaben prüfen<br>• Historische Kursdaten beschaffen<br>• Rendite und Volatilität schätzen<br>• Mögliche Wertverläufe simulieren<br>• Ergebnis als Fächerdiagramm anzeigen |

---

## Festlegungen des Product Owners (30.09.2026)

Beim Abgleich der Akzeptanzkriterien mit dem API-Design wurden folgende Punkte festgelegt:

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| 1 | US-03 | Jede Gewichtung muss größer als 0 % sein. | Eine Position mit 0 % hat keinen Einfluss auf die Simulation. |
| 2 | US-03 | Die Summe der Gewichtungen muss 100 % ergeben, mit einer Rundungstoleranz von ± 0,01 Prozentpunkten. | Verhindert, dass Eingaben mit Nachkommastellen an Rundungsdifferenzen scheitern. |
| 3 | US-04 | Der Anlagehorizont liegt zwischen 1 und 40 ganzen Jahren. | Kürzere Zeiträume sind auf Monatsbasis kaum aussagekräftig, längere nicht seriös aus der Kurshistorie ableitbar; begrenzt zudem die Rechenzeit. |
| 4 | US-04 | Startkapital und Sparrate dürfen nicht beide 0 sein. | Ohne eingesetztes Kapital ist das Ergebnis immer 0. |
| 5 | US-04 | Die Anzahl der Simulationspfade ist ein optionales Eingabefeld: Standard 10.000, wählbar von 1.000 bis 50.000. | Nutzer können zwischen schneller Berechnung und höherer Genauigkeit wählen; die Obergrenze begrenzt die Rechenzeit. |
| 6 | US-04 | Der Seed ist ein optionales Eingabefeld (ganze Zahl). Ohne Angabe erzeugt das System einen Seed und zeigt ihn an. | Ermöglicht es dem Nutzer, eine Simulation exakt zu wiederholen; dient zugleich der Testbarkeit. |
| 7 | US-04 | Für jede Anlage sind mindestens 3 Jahre Kurshistorie erforderlich, sonst erscheint ein Hinweis mit der betroffenen Anlage. | Mindestdatenmenge für eine belastbare Schätzung von Rendite und Volatilität. |

---

## Aktualisierte Akzeptanzkriterien

Zur Übernahme in die Story-Issues und das Sprint-Planning-Protokoll. Änderungen gegenüber dem Stand vom 27.09.2026 sind gekennzeichnet.

**US-02 Anlagen suchen** (unverändert)

- Die Suche nach Ticker, ISIN oder Name liefert eine Trefferliste mit Name, Ticker, Börse und Währung.
- Die Suche startet erst ab zwei eingegebenen Zeichen.
- Bei keinem Treffer erscheint ein Hinweis.
- Fällt die Finanzdaten-API aus, erscheint eine verständliche Fehlermeldung statt eines Absturzes.

**US-03 Portfolio erstellen**

- Ein Portfolio hat einen Namen und mindestens eine Anlage.
- Die Gewichtungen müssen zusammen 100 % ergeben (Rundungstoleranz ± 0,01 Prozentpunkte), sonst ist Speichern nicht möglich. *(ergänzt: Toleranz)*
- Jede Gewichtung muss größer als 0 % sein. *(geändert, vorher: negative Gewichtungen werden abgelehnt)*
- Doppelte Anlagen werden abgelehnt.

**US-04 Monte-Carlo-Simulation durchführen**

- Startkapital, Sparrate und Anlagehorizont sind Pflichtfelder; negative oder leere Werte werden abgelehnt.
- Der Anlagehorizont liegt zwischen 1 und 40 ganzen Jahren. *(neu)*
- Startkapital und Sparrate dürfen nicht beide 0 sein. *(neu)*
- Die Anzahl der Simulationspfade kann optional zwischen 1.000 und 50.000 gewählt werden; ohne Angabe werden 10.000 Pfade berechnet. *(geändert, vorher: mindestens 10.000 Pfade)*
- Ein Seed kann optional als ganze Zahl eingegeben werden; ohne Angabe erzeugt das System einen Seed und zeigt ihn nach der Simulation an. *(neu)*
- Bei gleichem Seed und gleichen Eingaben liefert die Simulation identische Ergebnisse. *(präzisiert)*
- Rendite und Volatilität werden aus historischen Kursdaten der Anlagen geschätzt.
- Liegen für eine Anlage weniger als 3 Jahre Kurshistorie vor, erscheint ein Hinweis, der die betroffene Anlage nennt. *(neu)*
- Sind die Kursdaten nicht abrufbar, erscheint eine verständliche Fehlermeldung statt eines Absturzes. *(neu)*
- Ein Fächerdiagramm zeigt das 5., 50. und 95. Perzentil über den Anlagehorizont.
