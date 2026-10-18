# Kurzformat-Tabellen – Sprint 3

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 3 (12.10.2026 – 18.10.2026) |
| **Issue** | Docs #83 Kurzformat-Tabelle US-05 und PO-Festlegungen zu den Kennzahlen; Docs #86 Kurzformat-Tabelle US-06; Docs #89 Kurzformat-Tabelle US-07; Docs #95 Kurzformat-Tabelle US-08 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 1 |
| **Stand** | 18.10.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 3 (Abschnitt 4), API-Design (Abschnitte Portfolios, Simulationen, Vergleiche; Stand 17.10.2026), ADR-05, Akteursbeschreibung v2, Kurzformat-Tabellen Sprint 1 und Sprint 2 (E1 bis E23), NFRs (NF-01, NF-04, NF-08), Umsetzung im Backend: BE #68 (Simulationen speichern), BE #69 (Vergleich), BE #70 (Portfolio ändern und löschen), BE #71 (Kennzahlen) |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt `kurzformat_sprint-2.md` fort. Damit liegen für alle Stories bis US-08 Kurzformat-Tabellen vor; US-09 und US-10 folgen mit ihrer Einplanung.

In diesem Sprint hat das Team die Kennzahlen, den Vergleich, das Ändern und Löschen von Portfolios und das Speichern von Simulationen im Backend umgesetzt, bevor die PO-Festlegungen schriftlich vorlagen. Der Product Owner hat mit dem Team vereinbart, dass die Entwickler fachliche Detailfragen mit dem besseren Code-Kontext selbst entscheiden und der Product Owner die Regeln anschließend prüft und hier festhält. Die Festlegungen E24 bis E34 übernehmen deshalb die umgesetzten Regeln; wo sie von den Vorschlägen im Sprint-Planning-Protokoll abweichen, ist das in der Spalte „Begründung“ vermerkt.

Alle vier Stories setzen eine gültige Anmeldung voraus (geschützte Anwendungsfälle laut Akteursbeschreibung v2). Ohne gültige Anmeldung antwortet das Backend mit 401 und die Oberfläche führt zur Anmeldung (E14).

---

## US-05 Kennzahlen anzeigen

| **Name** | **Kennzahlen anzeigen** |
|---|---|
| **Art** | User Story (US-05), erweitert US-04 |
| **Kurzbeschreibung** | Der Nutzer sieht unter dem Fächerdiagramm einer Simulation wenige zusammenfassende Kennzahlen, um das Ergebnis schnell einschätzen zu können, ohne den gesamten Verlauf lesen zu müssen. |
| **Auslöser** | Eine Simulation wurde berechnet (US-04) oder eine gespeicherte Simulation geöffnet (US-08). |
| **Akteure** | Registrierter Nutzer (primär) |
| **Eingehende Information** | Keine zusätzliche Pflichteingabe. Optional bei der Simulation: Zielbetrag in Euro (größer als 0), z. B. 100.000 € (E27). |
| **Vorbedingungen** | • Der Nutzer ist angemeldet.<br>• Für ein eigenes Portfolio liegt ein Simulationsergebnis vor (US-04) oder eine gespeicherte Simulation ist geöffnet (US-08). |
| **Nachbedingungen** | • Unter dem Fächerdiagramm stehen die Summe der Einzahlungen und der Endwert im ungünstigen (p5), mittleren (p50) und günstigen Verlauf (p95) (E24).<br>• Value at Risk (95 %) und maximaler Drawdown im Median werden angezeigt (E25, E26) und im Glossar erklärt.<br>• Mit Zielbetrag wird die Wahrscheinlichkeit angezeigt, ihn am Ende zu erreichen; ohne Zielbetrag entfällt die Angabe (E27).<br>• Ein Zielbetrag von 0 oder kleiner wird mit „Der Zielbetrag muss größer als 0 sein.“ abgelehnt; es wird nicht gerechnet.<br>• Gleiche Eingaben und gleicher Seed ergeben identische Kennzahlen (NF-08). |
| **Essenzielle Schritte** | • Simulationsergebnis bereitstellen (US-04)<br>• Kennzahlen aus den simulierten Verläufen ermitteln<br>• Kennzahlen unter dem Fächerdiagramm anzeigen |

---

## US-06 Portfolio bearbeiten und löschen

| **Name** | **Portfolio bearbeiten und löschen** |
|---|---|
| **Art** | User Story (US-06) |
| **Kurzbeschreibung** | Der Nutzer ändert Name, Anlagen und Gewichtungen eines eigenen Portfolios oder löscht es, um seine Anlagestrategie anzupassen. |
| **Auslöser** | Der Nutzer möchte die Zusammensetzung eines Portfolios ändern oder ein Portfolio nicht mehr verwenden. |
| **Akteure** | Registrierter Nutzer (primär) |
| **Eingehende Information** | Bearbeiten: Name (höchstens 100 Zeichen), Anlagen und Gewichtungen in Prozent (wie US-03). Löschen: Bestätigung des Nutzers. |
| **Vorbedingungen** | • Der Nutzer ist angemeldet.<br>• Der Nutzer hat mindestens ein eigenes Portfolio (US-03). |
| **Nachbedingungen** | • Bearbeiten: Die Änderungen sind gespeichert und werden in der Portfolioliste angezeigt; der Zeitpunkt der letzten Änderung ist aktualisiert.<br>• Bearbeiten: Es gelten dieselben Prüfregeln und Meldungen wie beim Erstellen (E1, E2, E9). Bei einer ungültigen Änderung bleibt das Portfolio unverändert.<br>• Löschen: Erst nach der Bestätigung wird gelöscht. Das Portfolio erscheint nicht mehr in der Liste und kann nicht mehr simuliert werden. Seine gespeicherten Simulationen werden mitgelöscht (E28).<br>• Ein fremdes oder nicht vorhandenes Portfolio wird weder geändert noch gelöscht und wird wie ein nicht vorhandenes behandelt (404, NF-04). |
| **Essenzielle Schritte** | • Eigenes Portfolio auswählen<br>• Bearbeiten: Name, Anlagen oder Gewichtungen ändern, Änderungen prüfen und speichern<br>• Löschen: Löschen bestätigen und Portfolio entfernen |

---

## US-07 Anlagen und Portfolios vergleichen (Kern-Story)

| **Name** | **Anlagen und Portfolios vergleichen** |
|---|---|
| **Art** | User Story (US-07), Kern-Story |
| **Kurzbeschreibung** | Der Nutzer vergleicht zwei Einträge, jeweils ein eigenes Portfolio oder eine einzelne Anlage, unter denselben Annahmen, um sich zwischen ihnen zu entscheiden. Beide Einträge werden mit denselben Eingaben, demselben Schätzzeitraum und denselben Zufallszahlen simuliert, sodass Unterschiede nur auf die Zusammensetzung zurückgehen. |
| **Auslöser** | Der Nutzer schwankt zwischen zwei Portfolios oder Anlagen und möchte wissen, welche Variante unter gleichen Annahmen besser zu ihm passt. |
| **Akteure** | Registrierter Nutzer (primär), Finanzdaten-API Twelve Data (sekundär) |
| **Eingehende Information** | • Zwei Einträge, jeweils ein eigenes Portfolio oder eine Anlage aus der Anlagensuche (US-02)<br>• Optional je Eintrag eine Bezeichnung<br>• Gemeinsame Parameter wie bei US-04: Startkapital, Sparrate (in Euro, nicht beide 0), Anlagehorizont (1 bis 40 Jahre), Anzahl der Pfade (1.000 bis 50.000, Standard 10.000), optional Seed und Zielbetrag |
| **Vorbedingungen** | • Der Nutzer ist angemeldet.<br>• Für jeden Eintrag ist ein eigenes Portfolio vorhanden (US-03) oder eine Anlage über die Anlagensuche gefunden (US-02). |
| **Nachbedingungen** | • Beide Verläufe (Median und Bandbreite p5 bis p95) stehen in einem Diagramm; die Kennzahlen aus US-05 stehen nebeneinander in einer Tabelle.<br>• Jeder Eintrag trägt seine Bezeichnung; ohne Eingabe den Namen des Portfolios bzw. der Anlage.<br>• Der verwendete Seed und der gemeinsame Schätzzeitraum werden angezeigt (E29).<br>• Die Wahrscheinlichkeit, dass der erste Eintrag am Ende mehr wert ist als der zweite, wird angezeigt (E31).<br>• Gleiche Eingaben und gleicher Seed ergeben identische Ergebnisse (NF-08).<br>• Ungültige Parameter werden mit denselben Meldungen wie bei US-04 abgelehnt (400).<br>• Liegen für eine oder mehrere Anlagen weniger als 36 Monatswerte vor, nennt der Hinweis alle betroffenen Anlagen (422, E6, E8); sind die Kursdaten nicht abrufbar, erscheint „Kursdaten sind gerade nicht verfügbar.“ (503).<br>• Ein fremdes oder nicht vorhandenes Portfolio wird wie ein nicht vorhandenes behandelt (404, NF-04). |
| **Essenzielle Schritte** | • Zwei Einträge auswählen und optional bezeichnen<br>• Gemeinsame Parameter eingeben<br>• Eingaben prüfen<br>• Kursdaten bereitstellen und Rendite und Volatilität für alle Anlagen in einem gemeinsamen Schätzzeitraum schätzen<br>• Beide Einträge mit denselben Zufallszahlen simulieren<br>• Verläufe und Kennzahlen gegenüberstellen |

Der Ablauf mit Alternativszenarien ist im Aktivitätsdiagramm US-07 (Docs #90) dargestellt, die beteiligten Klassen im Sequenzdiagramm US-07 (Docs #91).

---

## US-08 Simulation speichern und aufrufen

| **Name** | **Simulation speichern und aufrufen** |
|---|---|
| **Art** | User Story (US-08) |
| **Kurzbeschreibung** | Der Nutzer speichert eine Simulation unter einem Namen und ruft sie später wieder auf, um das Ergebnis nicht erneut berechnen zu müssen. Nicht mehr benötigte Simulationen löscht er. |
| **Auslöser** | Der Nutzer möchte ein Simulationsergebnis aufbewahren, später wieder ansehen oder nicht mehr benötigte Ergebnisse entfernen. |
| **Akteure** | Registrierter Nutzer (primär) |
| **Eingehende Information** | Speichern: Name (Pflicht, höchstens 100 Zeichen) zusätzlich zu den Eingaben der Simulation. Aufrufen und Löschen: Auswahl einer gespeicherten Simulation; beim Löschen zusätzlich die Bestätigung. |
| **Vorbedingungen** | • Der Nutzer ist angemeldet.<br>• Speichern: Für ein eigenes Portfolio liegen gültige Eingaben für eine Simulation vor (US-04).<br>• Aufrufen und Löschen: Der Nutzer hat mindestens eine Simulation gespeichert. |
| **Nachbedingungen** | • Die Simulation ist unter dem Namen mit Parametern, Schätzungen, Verlauf und Kennzahlen gespeichert (E32). Ohne Namen oder mit mehr als 100 Zeichen wird nicht gespeichert und die Meldung „Bitte einen Namen mit höchstens 100 Zeichen eingeben.“ erscheint.<br>• Die Liste zeigt nur eigene gespeicherte Simulationen mit Name, Portfolio und Speicherzeitpunkt, die neueste zuerst (NF-04).<br>• Eine geöffnete Simulation zeigt dieselben Parameter, Schätzungen, Verläufe und Kennzahlen wie beim Speichern, ohne Neuberechnung, auch wenn sich Kursdaten oder das Portfolio inzwischen geändert haben.<br>• Eine gelöschte Simulation erscheint nicht mehr in der Liste.<br>• Eine fremde oder nicht vorhandene Simulation wird wie eine nicht vorhandene behandelt (404, NF-04).<br>• Vergleiche werden in Sprint 3 noch nicht gespeichert (E34). |
| **Essenzielle Schritte** | • Speichern: Namen eingeben, Simulation berechnen und mit Ergebnis speichern<br>• Aufrufen: Liste der gespeicherten Simulationen anzeigen, eine Simulation auswählen und anzeigen<br>• Löschen: Simulation auswählen, Löschen bestätigen und entfernen |

---

## Festlegungen des Product Owners (18.10.2026)

Anlass: Abgleich der Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 3 mit der Umsetzung im Backend (BE #68 bis #71) und dem API-Design. Die Nummerierung setzt E1 bis E23 fort. Die Festlegungen übernehmen die umgesetzten Regeln (siehe Einleitung).

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| E24 | US-05 | Angezeigt werden sieben **Kennzahlen zum Ende des Anlagehorizonts**: Summe der Einzahlungen (E19), Endwert im ungünstigen (p5), mittleren (p50) und günstigen Verlauf (p95), Value at Risk (95 %), maximaler Drawdown im Median und, falls ein Zielbetrag angegeben ist, die Wahrscheinlichkeit, ihn zu erreichen. Die Endwerte werden mit derselben Methode wie das Fächerdiagramm ermittelt; sie stimmen daher mit dem letzten Verlaufspunkt überein. | Wenige Werte, die ein Privatanleger ohne Vorwissen einordnen kann. Gleiche Rechenmethode verhindert Abweichungen zwischen Diagramm und Tabelle. Umgesetzt in BE #71. |
| E25 | US-05 | Der **Value at Risk (95 %)** ist der Betrag, um den der Endwert im ungünstigen Verlauf (p5) unter der Summe der Einzahlungen liegt. Liegt der ungünstige Endwert über den Einzahlungen, beträgt er 0 €. | Bezugsgröße sind die eingezahlten Euro, weil ein Sparer wissen will, ob er im schlechten Fall weniger zurückbekommt, als er eingezahlt hat. Bewusste Vereinfachung gegenüber dem Value at Risk aus dem Risikomanagement, der sich auf einen festen Zeitraum und den Anfangswert bezieht. |
| E26 | US-05 | Der **maximale Drawdown** ist je Simulationspfad der größte Rückgang des Portfoliowerts von einem bisherigen Höchststand, als Prozentwert (Monatswerte). Angezeigt wird der Median über alle Pfade. Ist der Höchststand 0 € (Startkapital 0 € vor der ersten Sparrate), zählt kein Rückgang. | Zeigt, mit welchem zwischenzeitlichen Wertverlust der Nutzer im typischen Verlauf rechnen muss. Bekannte Vereinfachung: Der Wert enthält die laufenden Einzahlungen, die Rückgänge abmildern; er ist daher kleiner als der Drawdown der reinen Wertentwicklung. |
| E27 | US-04, US-05 | Der **Zielbetrag** ist optional und muss größer als 0 sein. Die **Zielwahrscheinlichkeit** ist der Anteil der Pfade, deren Endwert mindestens den Zielbetrag erreicht. Ohne Zielbetrag wird sie nicht berechnet und nicht angezeigt (in der API `null`). | Ein Sparziel ist für viele Nutzer die wichtigste Frage, aber nicht für alle; deshalb freiwillig. `null` statt 0 %, damit „kein Ziel“ nicht mit „Ziel unerreichbar“ verwechselt wird. |
| E28 | US-06, US-08 | Wird ein Portfolio gelöscht, werden **seine gespeicherten Simulationen mitgelöscht**. | Beantwortet die offene PO-Frage aus dem Sprint-Planning-Protokoll Sprint 3. Eine gespeicherte Simulation ohne Portfolio ließe sich in der Liste keinem Portfolio mehr zuordnen. Umgesetzt in BE #68 (Migration V7, Löschweitergabe). Die Bestätigungsabfrage vor dem Löschen schützt vor versehentlichem Datenverlust. |
| E29 | US-07 | Für den Vergleich gilt **ein gemeinsamer Schätzzeitraum für alle Anlagen beider Einträge** (E17 auf den Vergleich übertragen). Hat eine Anlage eine kurze Historie, verkürzt sie den Zeitraum für beide Einträge; der Zeitraum wird im Ergebnis angezeigt. Rendite und Volatilität einer Anlage können deshalb im Vergleich von einer Einzelsimulation abweichen. | Nur mit gleichem Zeitraum gehen Unterschiede auf die Zusammensetzung zurück (ADR-05). |
| E30 | US-07 | Eine **einzelne Anlage** wird wie ein Portfolio mit 100 % in dieser Anlage simuliert. | Portfolios und Anlagen werden gleich behandelt; es ist kein eigener Rechenweg nötig (ADR-05). |
| E31 | US-07 | Zusätzlich zu den Kennzahlen aus US-05 wird die **Wahrscheinlichkeit** angezeigt, dass der erste Eintrag am Ende mehr wert ist als der zweite (Anteil der Pfade). | Beantwortet die eigentliche Vergleichsfrage in einer Zahl. Aussagekräftig, weil beide Einträge dieselben Zufallszahlen verwenden. Im API-Design als `probabilityAOutperformsB` vorgesehen. |
| E32 | US-08 | Gespeichert wird **beim Berechnen**: Der Nutzer gibt einen Namen an, das System rechnet mit den Eingaben und dem Seed der angezeigten Simulation und speichert Parameter, Schätzungen, Verlauf und Kennzahlen. Beim Aufrufen wird das gespeicherte Ergebnis unverändert angezeigt. | Entspricht dem API-Design (`save: true` mit `name`, 201). Mit demselben Seed entsteht dasselbe Ergebnis (NF-08). Das gespeicherte Ergebnis bleibt erhalten, auch wenn sich Kursdaten oder Portfolio später ändern. Umgesetzt in BE #68. |
| E33 | US-07 | Der Vergleich von zwei Einträgen mit je 10.000 Pfaden ist mit je 5 Anlagen und 40 Jahren Anlagehorizont in **höchstens 6 Sekunden** berechnet; gemessen wird `runtimeMs` wie bei NF-01. | Ersetzt den Platzhalter „[PO: Sekunden]“ im Sprint-Planning-Protokoll. Zwei Einträge verdoppeln etwa den Rechenaufwand gegenüber NF-01 (3 Sekunden); die Schätzung erfolgt nur einmal (ADR-05). |
| E34 | US-08 | **Vergleiche speichern** (`POST /comparisons` mit `save`) folgt in Sprint 4 zusammen mit US-10. | Laut Sprint-Planning-Protokoll Sprint 3 bei Engpass nach Sprint 4; im Backend noch nicht umgesetzt (`save` wird noch nicht ausgewertet). |

---

## Auswirkungen auf bestehende Stories

| Story | Feld | Bisher | Ab Sprint 3 | Grundlage |
|---|---|---|---|---|
| US-04 Simulation durchführen | Eingehende Information | Startkapital, Sparrate, Anlagehorizont, Pfade, optional Seed | Zusätzlich optional: Zielbetrag in Euro (größer als 0) | E27 |
| | Nachbedingungen | Fächerdiagramm mit Schätzungen und Hinweisen | Zusätzlich: Kennzahlen unter dem Fächerdiagramm (US-05) | E24 |
| US-03 Portfolio erstellen | – | – | **Unverändert.** Bearbeiten und Löschen sind als eigene Story US-06 beschrieben; die Prüfregeln gelten dort ebenso. | US-06 |

---

## Akzeptanzkriterien

Zur Übernahme in die Story-Issues US-05 (Docs #6), US-06 (Docs #5), US-07 (Docs #3) und US-08 (Docs #4). Grundlage sind die Kriterien aus dem Sprint-Planning-Protokoll Sprint 3; Präzisierungen durch die Festlegungen E24 bis E34 sind gekennzeichnet. Für alle vier Stories gilt: Ohne gültige Anmeldung 401.

**US-05 Kennzahlen anzeigen**

- Unter dem Fächerdiagramm stehen die Summe der Einzahlungen und der Endwert im ungünstigen (p5), mittleren (p50) und günstigen Verlauf (p95). *(E24)*
- Value at Risk (95 %) und maximaler Drawdown im Median werden angezeigt und im Glossar erklärt. *(E25, E26)*
- Ist ein Zielbetrag angegeben, erscheint die Wahrscheinlichkeit, ihn zu erreichen; ohne Zielbetrag entfällt die Angabe. Ein Zielbetrag von 0 oder kleiner wird mit „Der Zielbetrag muss größer als 0 sein.“ abgelehnt. *(E27)*
- Gleicher Seed führt zu identischen Kennzahlen (NF-08).

**US-06 Portfolio bearbeiten und löschen**

- Name, Anlagen und Gewichtungen eines eigenen Portfolios lassen sich ändern; es gelten dieselben Prüfregeln und Meldungen wie beim Erstellen (US-03).
- Nach dem Speichern zeigt die Portfolioliste die geänderten Werte; der Zeitpunkt der letzten Änderung ist aktualisiert.
- Ungültige Änderungen werden abgelehnt; das Portfolio bleibt unverändert.
- Vor dem Löschen erscheint eine Bestätigung; danach ist das Portfolio nicht mehr in der Liste, kann nicht mehr simuliert werden und seine gespeicherten Simulationen sind gelöscht. *(E28)*
- Ändern oder Löschen eines fremden oder nicht vorhandenen Portfolios liefert 404; die Daten des anderen Nutzers bleiben unverändert (NF-04).

**US-07 Anlagen und Portfolios vergleichen**

- Der Nutzer wählt genau zwei Einträge, jeweils ein eigenes Portfolio oder eine einzelne Anlage aus der Suche. *(E30)*
- Jeder Eintrag bekommt eine Bezeichnung; ohne Eingabe wird der Portfolio- bzw. Anlagename verwendet.
- Beide Einträge werden mit denselben Parametern, demselben Seed und einem gemeinsamen Schätzzeitraum berechnet; Seed und Schätzzeitraum werden angezeigt. *(E29)*
- Gleiche Eingaben und gleicher Seed führen zu identischen Ergebnissen (NF-08).
- Das Ergebnis zeigt beide Verläufe (Median und Bandbreite p5 bis p95) in einem Diagramm, die Kennzahlen aus US-05 nebeneinander in einer Tabelle und die Wahrscheinlichkeit, dass der erste Eintrag am Ende mehr wert ist als der zweite. *(E31)*
- Ungültige Parameter werden wie bei US-04 mit derselben Meldung abgelehnt (400).
- Fehlen für einen Eintrag Kursdaten (weniger als 36 Monatswerte), werden alle betroffenen Anlagen genannt (422, E6); sind die Kursdaten nicht abrufbar, erscheint „Kursdaten sind gerade nicht verfügbar.“ (503).
- Ein fremdes Portfolio wird wie ein nicht vorhandenes behandelt (404, NF-04).
- Der Vergleich von zwei Einträgen mit je 10.000 Pfaden ist nach höchstens 6 Sekunden fertig. *(E33)*

**US-08 Simulation speichern und aufrufen**

- Der Nutzer kann eine Simulation unter einem Namen speichern; ohne Namen oder mit mehr als 100 Zeichen wird nicht gespeichert und die Meldung nennt das Feld (400). *(E32)*
- Die Liste der gespeicherten Simulationen zeigt Name, Portfolio und Speicherzeitpunkt, neueste zuerst, und nur eigene Einträge (NF-04).
- Eine gespeicherte Simulation lässt sich öffnen und zeigt dieselben Parameter, Schätzungen, Verläufe und Kennzahlen wie beim Speichern, ohne Neuberechnung, auch wenn sich Kursdaten oder das Portfolio inzwischen geändert haben.
- Der Nutzer kann eine gespeicherte Simulation löschen; danach erscheint sie nicht mehr in der Liste (204).
- Abrufen oder Löschen einer fremden oder nicht vorhandenen Simulation liefert 404 (NF-04).
- Vergleiche speichern folgt in Sprint 4. *(E34)*

---

## Änderungshistorie

| Version | Datum | Story | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 18.10.2026 | US-05 bis US-08 | Neu: Kurzformat-Tabellen, Festlegungen E24 bis E34, Auswirkungen auf US-03 und US-04, Akzeptanzkriterien zur Übernahme in die Story-Issues | Docs #83, #86, #89, #95; Sprint-Planning-Protokoll Sprint 3; Abgleich mit BE #68 bis #71 |
