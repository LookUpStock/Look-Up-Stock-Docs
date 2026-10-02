# Kurzformat-Tabellen – Sprint 1

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026) |
| **Issue** | #14 Kurzformat-Tabellen für US-02, US-03, US-04 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 2 |
| **Stand** | 02.10.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 1, API-Design (Stand 02.10.2026), Entscheidungen des Product Owners E1 bis E10 (02.10.2026), NFRs nach FURPS+ (Version 2) |
| **KI-Nutzung** | Entwurf und Überarbeitung mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log (Nr. 14 und Eintrag zu Version 2) |

Die Tabellen folgen der Vorlage zur Use-Case-Kurzbeschreibung aus der Vorlesung; das dort gestrichene Feld „Ergebnis“ entfällt. Die essenziellen Schritte sind bewusst fachlich formuliert. Der technische Ablauf inklusive Alternativszenarien wird für die Kern-Story US-04 im Aktivitäts- und Sequenzdiagramm dargestellt.

In Sprint 1 ist noch keine Anmeldung erforderlich. Mit US-01 (Sprint 2) kommt bei allen drei Stories die Vorbedingung „Nutzer ist angemeldet“ hinzu, und Portfolios werden dem angemeldeten Nutzer zugeordnet.

---

## US-02 Anlagen suchen

| **Name** | **Anlagen suchen** |
|---|---|
| **Art** | User Story (US-02) |
| **Kurzbeschreibung** | Der Nutzer sucht über Ticker, ISIN oder Namen nach Aktien und ETFs, um sie anschließend einem Portfolio hinzuzufügen. |
| **Auslöser** | Der Nutzer möchte eine bestimmte Anlage für sein Portfolio finden. |
| **Akteure** | Nutzer (primär), Finanzdaten-API Twelve Data (sekundär) |
| **Eingehende Information** | Suchbegriff (Ticker, ISIN oder Name, mindestens zwei Zeichen) |
| **Vorbedingungen** | Die Anwendung ist im Browser geöffnet. |
| **Nachbedingungen** | • Die Trefferliste mit Name, Ticker, Börse und Währung wird angezeigt; bei keinem Treffer erscheint der Hinweis „Keine passende Anlage gefunden.“<br>• Der Nutzer hat eine Anlage ausgewählt.<br>• Die ausgewählte Anlage kann in ein Portfolio übernommen werden. |
| **Essenzielle Schritte** | • Suchbegriff eingeben<br>• Passende Anlagen ermitteln<br>• Trefferliste anzeigen<br>• Anlage auswählen |

---

## US-03 Portfolio mit Gewichtungen erstellen

| **Name** | **Portfolio mit Gewichtungen erstellen** |
|---|---|
| **Art** | User Story (US-03) |
| **Kurzbeschreibung** | Der Nutzer stellt aus einer oder mehreren Anlagen ein Portfolio zusammen, vergibt einen Namen und legt für jede Anlage eine Gewichtung in Prozent fest. |
| **Auslöser** | Der Nutzer möchte eine Anlagestrategie abbilden, um sie anschließend zu simulieren und zu vergleichen. |
| **Akteure** | Nutzer |
| **Eingehende Information** | Portfolioname (Pflicht, höchstens 100 Zeichen), ausgewählte Anlagen (aus der Anlagensuche, US-02), Gewichtung je Anlage in Prozent (größer als 0, höchstens 2 Nachkommastellen) |
| **Vorbedingungen** | Die aufzunehmenden Anlagen wurden über die Anlagensuche (US-02) gefunden. |
| **Nachbedingungen** | • Das Portfolio ist unter seinem Namen gespeichert.<br>• Es enthält mindestens eine Anlage, jede Anlage höchstens einmal.<br>• Alle Gewichtungen sind größer als 0 %, haben höchstens 2 Nachkommastellen und ergeben zusammen 100 %.<br>• Das Portfolio kann für eine Simulation (US-04) ausgewählt werden. |
| **Essenzielle Schritte** | • Portfolio benennen<br>• Anlagen hinzufügen<br>• Gewichtungen festlegen<br>• Eingaben prüfen<br>• Portfolio speichern |

---

## US-04 Monte-Carlo-Simulation durchführen (Kern-Story)

| **Name** | **Monte-Carlo-Simulation durchführen** |
|---|---|
| **Art** | User Story (US-04), Kern-Story |
| **Kurzbeschreibung** | Der Nutzer gibt für ein gespeichertes Portfolio Startkapital, monatliche Sparrate und Anlagehorizont ein und kann optional die Anzahl der Simulationspfade und einen Seed festlegen. Das System schätzt aus historischen Kursdaten Rendite und Volatilität der enthaltenen Anlagen, simuliert die möglichen Wertverläufe und stellt deren Bandbreite als Fächerdiagramm dar. |
| **Auslöser** | Der Nutzer möchte wissen, wie sich sein Portfolio über einen gewählten Zeitraum entwickeln könnte. |
| **Akteure** | Nutzer (primär), Finanzdaten-API Twelve Data (sekundär) |
| **Eingehende Information** | Ausgewähltes Portfolio, Startkapital in Euro, monatliche Sparrate in Euro, Anlagehorizont in ganzen Jahren, optional Anzahl der Simulationspfade (Standard 10.000) und Seed, historische Kursdaten der enthaltenen Anlagen als Monatswerte (aus dem Cache oder der Finanzdaten-API) |
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
| 7 | US-04 | Für jede Anlage sind mindestens 3 Jahre Kurshistorie erforderlich, sonst erscheint ein Hinweis mit der betroffenen Anlage. *(präzisiert durch E6 und E8)* | Mindestdatenmenge für eine belastbare Schätzung von Rendite und Volatilität. |

## Festlegungen des Product Owners (02.10.2026)

Anlass: Hinweise aus dem Testkonzept (Grenzwerte) und Abgleich mit dem API-Design. Die Nummern E1 bis E7 entsprechen der Nummerierung im API-Design und im Testkonzept.

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| E1 | US-03 | Der Portfolioname ist Pflicht und hat höchstens 100 Zeichen. | Eindeutige Grenze für Eingabe, Speicherung und Test |
| E2 | US-03 | Gewichtungen haben höchstens 2 Nachkommastellen. Das Backend rechnet mit exakten Dezimalzahlen. | Verhindert, dass gültige Portfolios (z. B. 69,99 % + 30,01 %) an Rechenungenauigkeiten scheitern |
| E3 | US-03 | Es gibt keine Höchstzahl an Anlagen pro Portfolio. Die 5 Anlagen in NF-01 sind nur der Messfall des Performancetests. | Keine fachliche Notwendigkeit für eine Grenze |
| E6 | US-04 | Haben mehrere Anlagen zu wenig Kurshistorie, nennt der Hinweis alle betroffenen Anlagen. | Der Nutzer kann alle Probleme auf einmal beheben. |
| E7 | US-02, US-04 | Finanzdaten-API ist Twelve Data. | Auswahl aus dem API-Vergleich |
| E8 | US-04 | „Mindestens 3 Jahre Kurshistorie“ bedeutet mindestens 36 Monatswerte je Anlage. | Die Simulation rechnet in Monatsschritten mit Monatsdaten von Twelve Data; eine Anzahl ist eindeutig prüfbar. |
| E9 | US-03 | Ein Name, der nur aus Leerzeichen besteht, gilt als leer und wird abgelehnt. | Ein solcher Name ist in der Portfolioliste nicht erkennbar. |
| E10 | US-02 | Bei keinem Treffer lautet der Hinweis „Keine passende Anlage gefunden.“ | Einheitlicher Text für Oberfläche und Abnahmetest |

E4 (Suchparameter `limit` 1 bis 25, Standard 10) und E5 (unbekannte `assetId` wird abgelehnt) sind technische Regeln der Schnittstelle. Der Nutzer kann sie über die Oberfläche nicht auslösen. Sie stehen deshalb im API-Design und im Testkonzept, nicht in den Akzeptanzkriterien.

---

## Akzeptanzkriterien

Zur Übernahme in die Story-Issues (US-02: Docs #8, US-03: Docs #1, US-04: Docs #7). Änderungen gegenüber Version 1 (30.09.2026) sind mit *(v2)* gekennzeichnet; die vollständige Historie steht im letzten Abschnitt.

**US-02 Anlagen suchen**

- Die Suche nach Ticker, ISIN oder Name liefert eine Trefferliste mit Name, Ticker, Börse und Währung.
- Die Suche startet erst ab zwei eingegebenen Zeichen.
- Bei keinem Treffer erscheint der Hinweis „Keine passende Anlage gefunden.“ *(v2: Text festgelegt, E10)*
- Fällt die Finanzdaten-API aus, erscheint eine verständliche Fehlermeldung statt eines Absturzes.

**US-03 Portfolio erstellen**

- Ein Portfolio hat einen Namen mit höchstens 100 Zeichen und mindestens eine Anlage. Ein Name nur aus Leerzeichen gilt als leer. *(v2: E1, E9)*
- Die Gewichtungen müssen zusammen 100 % ergeben (Rundungstoleranz ± 0,01 Prozentpunkte), sonst ist Speichern nicht möglich.
- Jede Gewichtung muss größer als 0 % sein.
- Gewichtungen haben höchstens 2 Nachkommastellen. *(v2: neu, E2)*
- Doppelte Anlagen werden abgelehnt.

**US-04 Monte-Carlo-Simulation durchführen**

- Startkapital, Sparrate und Anlagehorizont sind Pflichtfelder; negative oder leere Werte werden abgelehnt.
- Der Anlagehorizont liegt zwischen 1 und 40 ganzen Jahren.
- Startkapital und Sparrate dürfen nicht beide 0 sein.
- Die Anzahl der Simulationspfade kann optional zwischen 1.000 und 50.000 gewählt werden; ohne Angabe werden 10.000 Pfade berechnet.
- Ein Seed kann optional als ganze Zahl eingegeben werden; ohne Angabe erzeugt das System einen Seed und zeigt ihn nach der Simulation an.
- Bei gleichem Seed und gleichen Eingaben liefert die Simulation identische Ergebnisse.
- Rendite und Volatilität werden aus historischen Kursdaten der Anlagen geschätzt.
- Liegen für eine oder mehrere Anlagen weniger als 3 Jahre Kurshistorie (36 Monatswerte) vor, erscheint ein Hinweis, der alle betroffenen Anlagen nennt. *(v2: E6, E8)*
- Sind die Kursdaten nicht abrufbar, erscheint eine verständliche Fehlermeldung statt eines Absturzes.
- Ein Fächerdiagramm zeigt das 5., 50. und 95. Perzentil über den Anlagehorizont.

---

## Änderungshistorie

| Version | Datum | Story | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 30.09.2026 | US-03 | Rundungstoleranz ± 0,01 Prozentpunkte ergänzt; „negative Gewichtungen werden abgelehnt“ durch „jede Gewichtung größer als 0 %“ ersetzt | Abgleich mit dem API-Design (Festlegungen 1, 2) |
| 1 | 30.09.2026 | US-04 | Neu: Anlagehorizont 1 bis 40 Jahre, Startkapital und Sparrate nicht beide 0, optionaler Seed, Hinweis bei weniger als 3 Jahren Kurshistorie, Fehlermeldung bei nicht abrufbaren Kursdaten. Geändert: Pfadanzahl optional 1.000 bis 50.000 statt mindestens 10.000. Präzisiert: identische Ergebnisse bei gleichem Seed | Abgleich mit dem API-Design (Festlegungen 3 bis 7) |
| 2 | 02.10.2026 | US-02 | Hinweistext bei keinem Treffer festgelegt; Twelve Data als Akteur genannt | E7, E10 |
| 2 | 02.10.2026 | US-03 | Name höchstens 100 Zeichen, Name nur aus Leerzeichen gilt als leer, Gewichtungen höchstens 2 Nachkommastellen (Kriterien, eingehende Information, Nachbedingungen) | E1, E2, E9; Hinweise aus dem Testkonzept |
| 2 | 02.10.2026 | US-04 | Hinweis nennt alle betroffenen Anlagen; 3 Jahre Kurshistorie als 36 Monatswerte definiert; Kursdaten aus Cache oder Finanzdaten-API; Twelve Data als Akteur genannt | E6, E7, E8; NFRs Version 2 (Begriff „Cache“) |
