# KI-Reflexion

| | |
|---|---|
| **Projekt** | LookUpStock, Kurs WWI25B |
| **Issue** | Docs #216 |
| **Stand** | 20.11.2026 |
| **Grundlagen** | Aufgabenbeschreibung Kapitel III.2, KI-Logs aller vier Teammitglieder (`ki-log/`), `CONTRIBUTING.md` (Abschnitt KI-Nutzung) |
| **KI-Nutzung** | Gliederung und Entwurf mit KI-Unterstützung aus den KI-Logs erstellt; Beispiele, Bewertungen und Schlussfolgerungen vom Team geprüft und angepasst (siehe KI-Log) |

## 1. Wie wir KI eingesetzt haben

Wir haben KI in allen Arbeitsschritten genutzt, von der Projektidee über Anforderungen, Diagramme und ADRs bis zu Code, Tests, Protokollen und Präsentationen. Die vier KI-Logs enthalten zusammen 96 Einträge: 81 mit Claude (Opus 5.5) als Chat-Assistent, 2 mit Claude Code in VS Code und 13 mit OpenAI Codex. 84 Einträge haben das Ergebnis „angepasst“; nur sieben Ergebnisse haben wir nach Prüfung oder unverändert übernommen.

Für die Zusammenarbeit galten zwei Regeln aus `CONTRIBUTING.md`: Jede KI-Nutzung, deren Ergebnis in ein abgegebenes Artefakt einfließt, kommt ins persönliche KI-Log, und keine personenbezogenen oder vertraulichen Daten gehen in KI-Werkzeuge. Für Claude Code mit Zugriff auf das Repository haben wir zusätzlich Grenzen gesetzt: kein Push auf `main`, Bestätigung vor Push, Merge und dem Schließen von Issues (KI-Log PKA-03).

## 2. Wo KI die Arbeit beschleunigt oder verbessert hat

**Gerüste und neue Technik.** Für Spring Boot, Spring for GraphQL, Flyway oder eine Datenbank-Queue haben wir uns Konzepte erklären und einen ersten Entwurf geben lassen (PIO-01, PIO-18, PIO-23). Die Entscheidungen haben wir selbst getroffen, zum Beispiel die Queue als Tabelle in PostgreSQL mit `FOR UPDATE SKIP LOCKED` statt eines externen Brokers. So konnten wir in einwöchigen Sprints Techniken einsetzen, die für uns neu waren.

**Abgleich von Dokumenten.** Am meisten Zeit hat KI beim Abgleich gespart. Beim Erstellen des Testkonzepts meldete sie 14 Widersprüche zwischen Kurzformat, API-Design und ADRs, die der Product Owner anschließend als Festlegungen E1 bis E7 entschied (PKA-06). Klassen- und ER-Diagramme haben wir mehrfach gegen Code und Migrationen abgleichen lassen (PKA-16). In den Sprint-Protokollen fand die KI sachliche Fehler wie „Java Bootstrap“ statt Spring Boot (PKA-15).

**Lücken in Tests.** Die KI hat Testlücken gefunden, die wir übersehen hatten, etwa den Zugriff auf ein fremdes Portfolio und das Anlegen ohne Token (PKA-11). Beide Tests haben wir ergänzt.

**Verständnis statt Kopie.** Vor der Zwischenpräsentation hat der Product Owner so lange zu Multiplizitäten und Datentypen nachgefragt, bis er die eigenen Folien selbst erklären konnte (ME-16). Diese Nutzung war für uns genauso wertvoll wie generierter Code.

## 3. Fehlerhafte, unvollständige oder inkonsistente Ergebnisse

Die folgenden Beispiele zeigen, dass KI-Ergebnisse ohne Prüfung nicht in die Abgabe hätten gehen dürfen.

**Beispiel 1: ADR widerspricht dem GRASP-Controller (PKA-05).** Im KI-Entwurf von ADR-02 empfing der REST-Controller das Systemereignis, koordiniert wurde aber im Service. Das passt nicht zum Controller-Pattern, wie es Kapitel III.4 verlangt: Das Objekt unterhalb der UI-Schicht soll das Ereignis empfangen und die Systemoperation koordinieren. **Entdeckt** haben wir das beim gezielten Abgleich mit der Aufgabenbeschreibung („Passen die ADRs wirklich zu dieser Aufgabenstellung?“). Nach der Korrektur ist der `SimulationService` der Controller; das Sequenzdiagramm US-04 wurde in Sprint 2 daran angepasst (Docs #59).

**Beispiel 2: Erfundene Bezüge in Anforderungen und Diagrammen (ME-04, ME-05, ME-11).** In einem KI-Entwurf der NFRs standen Testfall-Nummern, die es nicht gab. Ein Aktivitätsdiagramm enthielt Ja/Nein-Verzweigungen statt Bedingungen und eine nicht belegte „10 Jahre Kurshistorie“. Ein vorzeitiger Diagrammentwurf verwendete ausgedachte Methodennamen. **Entdeckt** hat das der Product Owner durch den Abgleich mit den Vorlesungsfolien und mit dem tatsächlichen Code. Daraus entstand eine feste Regel für die weitere Arbeit: Fehlt der KI eine Information, soll sie das melden und warten, statt etwas zu ergänzen.

**Beispiel 3: Unvollständiges API-Design (PKA-04).** Der erste Entwurf des API-Designs trennte Ein- und Ausgaben nicht, fasste drei Vergleichs-Endpunkte zusammen und ließ den Logout weg, obwohl US-01 ihn verlangt. **Entdeckt** wurde das im Team-Review gegen die Akzeptanzkriterien.

**Beispiel 4: Falsche Aussage über die eigene Arbeit (PKA-06).** Beim Kürzen der Abnahmetestfälle von 78 auf 32 behauptete die KI „gleiche Prüftiefe“. Tatsächlich fielen Grenzwerte weg, die Kapitel III.3 ausdrücklich verlangt. **Entdeckt** haben wir das im Review der gekürzten Liste. Die Abnahmetestfälle enthalten die Grenzwerte deshalb ausdrücklich, zum Beispiel AT-04-10 bis AT-04-12 für Anlagehorizont, Beträge und Pfadanzahl.

**Beispiel 5: Code auf veraltetem Stand (ME-19).** Ein KI-Vorschlag für die Befunde B7 und B1 beruhte auf einem alten Stand des Frontends. Nach dem Merge waren Änderungen eines anderen Teammitglieds überschrieben. **Entdeckt** haben wir das nach dem Merge; ein Reparatur-Pull-Request stellte die Änderungen wieder her (119 Tests grün). Zusätzlich hatte die KI angenommen, sie dürfe selbst pushen. Als Folge haben wir große Sammellieferungen abgelehnt, einzelne Pull Requests verlangt und das Review des Diffs in die Definition of Done aufgenommen.

**Beispiel 6: Falsche Bewertung bei der Abnahme (ME-12, ME-24).** Bei der Abnahme von Sprint 2 übersah die KI offene Pull Requests und hielt US-02 für abnehmbar, obwohl das laut Testkonzept nicht zutraf. Bei der Abnahme von Sprint 5 nannte sie ein falsches Testkonto. **Entdeckt** wurde beides, weil der Product Owner alle Abnahmetests selbst auf Heroku durchgeführt hat.

Die Beispiele haben ein gemeinsames Muster: Die KI liefert in sich schlüssige Ergebnisse, die nicht zum Rest des Projekts passen. Gefunden haben wir die Fehler fast immer durch einen Abgleich mit einer zweiten Quelle, also mit der Aufgabenbeschreibung, dem Code, den Akzeptanzkriterien oder einem eigenen Test. Wo die KI selbst nichts prüfen konnte, mussten wir es tun: Sie konnte die Twelve-Data-API nicht aufrufen (PKA-08) und den Backend-Code nicht kompilieren (PKA-22). Die ISIN-Suche haben wir deshalb von Hand im Browser getestet, die Mutations lokal.

## 4. Risiken

**Datenschutz.** Wir haben keine personenbezogenen Daten in KI-Werkzeuge eingegeben, auch keine Daten aus den Ausbildungsbetrieben. Die Nutzungskennzahlen aus dem Change Request werden ohne Nutzerbezug erfasst (E36). Beim Bedienbarkeitstest NF-09 werden die Testpersonen nur als P1 bis P3 geführt.

**Code Leakage und Zugangsdaten.** Code und Dokumente unseres Projekts sind in die Chats der Anbieter geflossen. Für ein Studienprojekt ohne Betriebsgeheimnisse halten wir das für vertretbar; mit Code aus einem Unternehmen wäre das nicht erlaubt. Ernster war ein anderer Fund: Die KI entdeckte, dass in `.env.example` ein echter API-Schlüssel von Twelve Data stand (PKA-21). Wir haben ihn ersetzt; seitdem liegt er nur noch als Config Var auf Heroku. Der Fall zeigt, dass KI ein Risiko aufdecken kann, aber auch, dass wir Secrets beim Arbeiten mit KI besonders sorgfältig trennen müssen, weil ganze Dateien schnell in einen Prompt kopiert sind.

**Urheberrecht.** KI-generierter Code kann geschützten Code aus dem Training ähneln, ohne dass das erkennbar ist. Wir haben deshalb Entwürfe nicht ungelesen übernommen, sondern an unsere Struktur angepasst, und für Bibliotheken nur gängige Open-Source-Abhängigkeiten über Maven und npm verwendet. Die Verantwortung für alle Ergebnisse liegt bei uns als Team, auch für KI-generierte Teile.

**Abhängigkeit und Nachvollziehbarkeit.** Bei Einträgen mit Code-Assistenten ist im KI-Log nicht immer festgehalten, welche zusätzlichen Tests oder Reviews es zu einem Vorschlag gab (CSI-01 bis CSI-03). Hier hätten wir genauer dokumentieren sollen. Außerdem waren Chatverläufe für frühe Einträge nicht mehr vollständig verfügbar; diese Prompts sind sinngemäß wiedergegeben (PKA-28).

## 5. Fazit

KI hat uns vor allem beim Einarbeiten, beim Entwerfen und beim Abgleichen von Dokumenten schneller gemacht. Den größten Wert hatte sie dort, wo wir sie als Prüfer eingesetzt haben, nicht als Autor. Ihre Fehler waren selten offensichtlich, sondern plausibel und deshalb gefährlich. Für ein nächstes Projekt würden wir drei Dinge beibehalten: jede KI-Ausgabe gegen eine zweite Quelle prüfen, der KI das Ergänzen fehlender Informationen verbieten und nur kleine, einzeln geprüfte Änderungen übernehmen. Ändern würden wir die Dokumentation: Die Prüfung eines KI-Ergebnisses halten wir künftig für jeden Eintrag im KI-Log fest.
