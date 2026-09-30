# Kurzformat-Tabellen – Sprint 1

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026) |
| **Issue** | #14 Kurzformat-Tabellen für US-02, US-03, US-04 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 30.09.2026 |
| **Grundlagen** | Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 1, API-Design v2 (29.09.2026) |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log Nr. 14 |

Die Tabellen folgen der Vorlage zur Use-Case-Kurzbeschreibung aus der Lehrveranstaltung. Die essenziellen Schritte sind bewusst fachlich formuliert. Der technische Ablauf inklusive Alternativszenarien wird für die Kern-Story US-04 im Aktivitäts- und Sequenzdiagramm dargestellt.

In Sprint 1 ist noch keine Anmeldung erforderlich. Mit US-01 (Sprint 2) kommt bei allen drei Stories die Vorbedingung „Nutzer ist angemeldet“ hinzu, und Portfolios werden dem angemeldeten Nutzer zugeordnet.

---

## US-02 Anlagen suchen

| Feld | Inhalt |
|---|---|
| **Name** | Anlagen suchen |
| **Art** | User Story (US-02) |
| **Kurzbeschreibung** | Der Nutzer sucht über Ticker, ISIN oder Namen nach Aktien und ETFs, um sie anschließend einem Portfolio hinzuzufügen. |
| **Auslöser** | Der Nutzer möchte eine bestimmte Anlage für sein Portfolio finden. |
| **Ergebnis** | Der Nutzer sieht eine Trefferliste passender Anlagen mit Name, Ticker, Börse und Währung und kann eine Anlage auswählen. |
| **Akteure** | Nutzer (primär), Finanzdaten-API (sekundär) |
| **Eingehende Information** | Suchbegriff (Ticker, ISIN oder Name, mindestens zwei Zeichen) |
| **Vorbedingungen** | Die Anwendung ist im Browser geöffnet. |
| **Nachbedingungen** | • Die Trefferliste wird angezeigt, bei keinem Treffer erscheint ein Hinweis.<br>• Gefundene Anlagen sind im System hinterlegt und können in Portfolios verwendet werden.<br>• Die ausgewählte Anlage steht zur Übernahme in ein Portfolio bereit. |
| **Essenzielle Schritte** | 1. Suchbegriff eingeben<br>2. Passende Anlagen ermitteln<br>3. Trefferliste anzeigen<br>4. Anlage auswählen |

---

## US-03 Portfolio mit Gewichtungen erstellen

| Feld | Inhalt |
|---|---|
| **Name** | Portfolio mit Gewichtungen erstellen |
| **Art** | User Story (US-03) |
| **Kurzbeschreibung** | Der Nutzer stellt aus einer oder mehreren Anlagen ein Portfolio zusammen, vergibt einen Namen und legt für jede Anlage eine Gewichtung in Prozent fest. |
| **Auslöser** | Der Nutzer möchte eine Anlagestrategie abbilden, um sie anschließend zu simulieren und zu vergleichen. |
| **Ergebnis** | Ein benanntes Portfolio mit gültigen Gewichtungen ist gespeichert und für Simulationen verfügbar. |
| **Akteure** | Nutzer |
| **Eingehende Information** | Portfolioname, ausgewählte Anlagen (aus der Anlagensuche, US-02), Gewichtung je Anlage in Prozent |
| **Vorbedingungen** | Die aufzunehmenden Anlagen wurden über die Anlagensuche (US-02) gefunden. |
| **Nachbedingungen** | • Das Portfolio ist gespeichert.<br>• Es enthält mindestens eine Anlage, jede Anlage höchstens einmal.<br>• Alle Gewichtungen sind größer als 0 % und ergeben zusammen 100 %.<br>• Das Portfolio kann für eine Simulation (US-04) ausgewählt werden. |
| **Essenzielle Schritte** | 1. Portfolio benennen<br>2. Anlagen hinzufügen<br>3. Gewichtungen festlegen<br>4. Eingaben prüfen<br>5. Portfolio speichern |

---

## US-04 Monte-Carlo-Simulation durchführen (Kern-Story)

| Feld | Inhalt |
|---|---|
| **Name** | Monte-Carlo-Simulation durchführen |
| **Art** | User Story (US-04), Kern-Story |
| **Kurzbeschreibung** | Der Nutzer gibt für ein gespeichertes Portfolio Startkapital, monatliche Sparrate und Anlagehorizont ein. Das System schätzt aus historischen Kursdaten Rendite und Volatilität der enthaltenen Anlagen, simuliert 10.000 mögliche Wertverläufe und stellt deren Bandbreite als Fächerdiagramm dar. |
| **Auslöser** | Der Nutzer möchte wissen, wie sich sein Portfolio über einen gewählten Zeitraum entwickeln könnte. |
| **Ergebnis** | Ein Fächerdiagramm zeigt den pessimistischen (5. Perzentil), mittleren (50. Perzentil) und optimistischen (95. Perzentil) Verlauf des Portfoliowerts über den Anlagehorizont. |
| **Akteure** | Nutzer (primär), Finanzdaten-API (sekundär) |
| **Eingehende Information** | Ausgewähltes Portfolio, Startkapital in Euro, monatliche Sparrate in Euro, Anlagehorizont in ganzen Jahren, historische Kursdaten der enthaltenen Anlagen (aus der Finanzdaten-API) |
| **Vorbedingungen** | Es wurde mindestens ein Portfolio erstellt (US-03). |
| **Nachbedingungen** | • Die Simulation wurde mit 10.000 Pfaden berechnet.<br>• Das Fächerdiagramm wird angezeigt.<br>• Die verwendeten Parameter einschließlich des Seeds sind bekannt, sodass das Ergebnis reproduzierbar ist.<br>• Das Ergebnis wird in Sprint 1 nicht gespeichert (Speichern folgt mit US-08). |
| **Essenzielle Schritte** | 1. Portfolio auswählen<br>2. Startkapital, Sparrate und Anlagehorizont eingeben<br>3. Eingaben prüfen<br>4. Historische Kursdaten beschaffen<br>5. Rendite und Volatilität schätzen<br>6. Mögliche Wertverläufe simulieren<br>7. Ergebnis als Fächerdiagramm anzeigen |

---

## Festlegungen des Product Owners (30.09.2026)

Beim Abgleich der Akzeptanzkriterien mit dem API-Design v2 wurden folgende Punkte festgelegt:

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| 1 | US-03 | Jede Gewichtung muss größer als 0 % sein. | Eine Position mit 0 % hat keinen Einfluss auf die Simulation. |
| 2 | US-04 | Der Anlagehorizont liegt zwischen 1 und 40 Jahren. | Kürzere Zeiträume sind auf Monatsbasis kaum aussagekräftig, längere nicht seriös aus der Kurshistorie ableitbar; begrenzt zudem die Rechenzeit. |
| 3 | US-04 | Startkapital und Sparrate dürfen nicht beide 0 sein. | Ohne eingesetztes Kapital ist das Ergebnis immer 0. |
| 4 | US-04 | Die Simulation berechnet 10.000 Pfade; die Anzahl ist kein Eingabefeld. | Für Nutzer ohne Statistikkenntnisse nicht sinnvoll einstellbar; feste Größe für den Performanz-Nachweis. |
| 5 | US-04 | Für jede Anlage sind mindestens 3 Jahre Kurshistorie erforderlich, sonst erscheint ein Hinweis mit der betroffenen Anlage. | Mindestdatenmenge für eine belastbare Schätzung von Rendite und Volatilität. |
| 6 | US-04 | Der Seed ist kein Eingabefeld für den Nutzer. | Technisches Detail für Reproduzierbarkeit und Tests. |

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

- Startkapital, Sparrate und Anlagehorizont können eingegeben werden; negative oder leere Werte werden abgelehnt.
- Der Anlagehorizont liegt zwischen 1 und 40 ganzen Jahren. *(neu)*
- Startkapital und Sparrate dürfen nicht beide 0 sein. *(neu)*
- Rendite und Volatilität werden aus historischen Kursdaten der Anlagen geschätzt.
- Liegen für eine Anlage weniger als 3 Jahre Kurshistorie vor, erscheint ein Hinweis, der die betroffene Anlage nennt. *(neu)*
- Fällt die Finanzdaten-API aus und liegen keine zwischengespeicherten Kursdaten vor, erscheint eine verständliche Fehlermeldung statt eines Absturzes. *(neu)*
- Die Simulation berechnet 10.000 Pfade. *(geändert, vorher: mindestens 10.000 Pfade)*
- Ein Fächerdiagramm zeigt das 5., 50. und 95. Perzentil über den Anlagehorizont.
- Bei gleichem Seed liefert die Simulation identische Ergebnisse. Der Seed ist kein Eingabefeld für den Nutzer. *(ergänzt)*
