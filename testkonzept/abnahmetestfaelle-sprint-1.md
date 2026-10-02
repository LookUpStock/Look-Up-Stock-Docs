# Abnahmetestfälle – Sprint 1

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 1 (28.09.2026 – 04.10.2026) |
| **Issue** | Docs #23 Testkonzept v1 und Abnahmetestfälle |
| **Verantwortlich** | Paul Kaufmann (Paul02112004) |
| **Stand** | 02.10.2026 |
| **Stories** | US-02 Anlagen suchen (Docs #8), US-03 Portfolio mit Gewichtungen erstellen (Docs #1), US-04 Monte-Carlo-Simulation durchführen (Docs #7, Kern-Story) |
| **Grundlagen** | Akzeptanzkriterien und Festlegungen des Product Owners in `anforderungen/kurzformat_sprint-1.md` (Stand 30.09.2026), Entscheidungen des Product Owners E1 bis E7 (02.10.2026), API-Design (Stand 02.10.2026), Aktivitätsdiagramm US-04 v1, [Testkonzept](testkonzept.md) |


---

## Vorbemerkungen

- **Durchführung:** Der Product Owner (KujoBaraka) führt die Testfälle mit Unterstützung der Entwickler in der Abnahmeumgebung durch (Testkonzept, Abschnitt 7). Das Backend läuft mit dem `MockMarketDataProvider` und den Testdaten aus Abschnitt 6 des Testkonzepts.
- **Zwei Prüfebenen:** Die Spalte „Erwartetes Ergebnis“ nennt zuerst, was die Oberfläche zeigt, und danach die erwartete API-Antwort. Viele ungültige Eingaben fängt bereits das Frontend ab, sodass keine Anfrage gesendet wird. Damit auch die Prüfung im Backend abgenommen wird, wird bei diesen Testfällen derselbe Wert zusätzlich direkt an die API geschickt (z. B. mit einem HTTP-Client).
- **Endpunkte:** Alle Pfade beziehen sich auf die Basis-URL `/api/v1`. Das Feld `token` wird in Sprint 1 ignoriert und in den Eingaben weggelassen.
- **Fehlerformat:** Fehlerantworten haben laut API-Design immer das Format `{ "error": { "code", "message", "details" } }`. Wo das API-Design einen Meldungstext vorgibt, ist er im erwarteten Ergebnis zitiert. Wo kein Text vorgegeben ist, wird geprüft, dass eine verständliche deutsche Meldung erscheint, die das betroffene Feld bzw. Problem nennt. Die vollständigen Fehlerfälle von `POST /portfolios` und `POST /simulations` werden in Docs #34 in das API-Design übernommen; danach werden die Meldungstexte hier zitiert. [OFFEN: Hinweistext bei keinem Suchtreffer.]
- **Basiseingaben US-04:** Wenn nicht anders angegeben, gilt das Portfolio „Mein Weltportfolio“ (EUNL 70 %, IS3N 30 %) mit `startCapital` 10000, `monthlyContribution` 200, `horizonYears` 20, ohne `paths` und ohne `seed`. In den Schritten wird nur der abweichende Wert genannt.
- **Art:** `Normalfall`, `Alternativszenario`, `Grenzwert` oder `NFR` (Nachweis einer nicht-funktionalen Anforderung).
- **Status:** zunächst `offen`; nach der Durchführung `bestanden` oder `fehlgeschlagen`. Bei `fehlgeschlagen` wird ein Bug-Issue angelegt und hier verlinkt (Testkonzept, Abschnitt 11).
- **Stabile IDs:** Einmal vergebene Testfall-IDs werden nicht umnummeriert. Nachträglich ergänzte Testfälle (aus den PO-Entscheidungen vom 02.10.2026) erhalten die nächste freie Nummer und stehen in der Tabelle ihrer Art.

**Anzahl der Testfälle**

| Story | Normalfall | Alternativszenario | Grenzwert | NFR | Summe |
|---|---|---|---|---|---|
| US-02 | 4 | 3 | 8 | 3 | 18 |
| US-03 | 2 | 5 | 16 | 0 | 23 |
| US-04 | 4 | 7 | 21 | 5 | 37 |
| **Gesamt** | 10 | 15 | 45 | 8 | 78 |

---

## US-02 Anlagen suchen

| ID | Bezug | Art | Vorbedingung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-02-01 | AK: Suche nach Ticker liefert Trefferliste mit Name, Ticker, Börse und Währung | Normalfall | Anwendung im Browser geöffnet | Suchbegriff `EUNL` eingeben | **UI:** Trefferliste mit „iShares Core MSCI World UCITS ETF“, Ticker EUNL, Börse XETRA, Währung EUR.<br>**API:** `GET /assets/search?q=EUNL` → `200 OK`; `results` enthält einen Eintrag mit `symbol` „EUNL“ und den Feldern `name`, `exchange`, `currency`. | offen |
| AT-02-02 | AK: Suche nach ISIN | Normalfall | wie AT-02-01 | Suchbegriff `IE00B4L5Y983` eingeben | **UI:** Trefferliste enthält EUNL mit Name, Ticker, Börse und Währung.<br>**API:** `200 OK`; `results` enthält den Eintrag mit `isin` „IE00B4L5Y983“. | offen |
| AT-02-03 | AK: Suche nach Name | Normalfall | wie AT-02-01 | Suchbegriff `msci` eingeben | **UI:** Trefferliste mit passenden Anlagen, jeweils mit Name, Ticker, Börse und Währung.<br>**API:** `200 OK`; `query` ist „msci“, `results` enthält mindestens zwei Einträge. | offen |
| AT-02-04 | Nachbedingung Kurzformat: ausgewählte Anlage kann in ein Portfolio übernommen werden | Normalfall | Trefferliste aus AT-02-03 wird angezeigt | Eintrag EUNL in der Trefferliste auswählen | **UI:** EUNL erscheint im Portfolio-Editor als neue Position.<br>**API:** keine weitere Anfrage notwendig; die `id` 17 aus dem Suchergebnis wird als `assetId` verwendet. | offen |
| AT-02-05 | AK: Suche startet erst ab zwei Zeichen | Grenzwert | wie AT-02-01 | 1 Zeichen eingeben: `M` | **UI:** Es wird keine Suche gestartet; es erscheint keine Trefferliste bzw. ein Hinweis, mindestens 2 Zeichen einzugeben.<br>**API (direkter Aufruf):** `GET /assets/search?q=M` → `400 VALIDATION_ERROR`, „Bitte mindestens 2 Zeichen eingeben.“ | offen |
| AT-02-06 | AK: Suche startet erst ab zwei Zeichen | Grenzwert | wie AT-02-01 | 2 Zeichen eingeben: `EU` | **UI:** Die Suche startet; eine Trefferliste (ggf. leer mit Hinweis) wird angezeigt.<br>**API:** `GET /assets/search?q=EU` → `200 OK`. | offen |
| AT-02-07 | AK: Suche startet erst ab zwei Zeichen | Grenzwert | wie AT-02-01 | Suchfeld leer lassen bzw. Inhalt löschen | **UI:** Keine Suche, keine Fehlermeldung, keine Trefferliste.<br>**API (direkter Aufruf):** `GET /assets/search?q=` → `400 VALIDATION_ERROR`. | offen |
| AT-02-14 | E4: `limit` erlaubt 1 bis 25 (knapp unterhalb) | Grenzwert | Testdaten mit mehr als 25 Treffern für `msci` | Direkter API-Aufruf `GET /assets/search?q=msci&limit=0` | **API:** `400 VALIDATION_ERROR`; `details` nennt das Feld `limit`.<br>Reiner API-Testfall. | offen |
| AT-02-15 | E4: `limit` erlaubt 1 bis 25 (untere Grenze) | Grenzwert | wie AT-02-14 | `GET /assets/search?q=msci&limit=1` | **API:** `200 OK`; `results` enthält genau 1 Eintrag. | offen |
| AT-02-16 | E4: `limit` erlaubt 1 bis 25 (obere Grenze) | Grenzwert | wie AT-02-14 | `GET /assets/search?q=msci&limit=25` | **API:** `200 OK`; `results` enthält genau 25 Einträge. | offen |
| AT-02-17 | E4: `limit` erlaubt 1 bis 25 (knapp oberhalb) | Grenzwert | wie AT-02-14 | `GET /assets/search?q=msci&limit=26` | **API:** `400 VALIDATION_ERROR`; `details` nennt das Feld `limit`. | offen |
| AT-02-18 | E4: `limit` ohne Angabe, Standard 10 | Grenzwert | wie AT-02-14 | `GET /assets/search?q=msci` ohne `limit` | **API:** `200 OK`; `results` enthält genau 10 Einträge. | offen |
| AT-02-08 | AK: Bei keinem Treffer erscheint ein Hinweis | Alternativszenario | wie AT-02-01 | Suchbegriff `xyzabc` eingeben | **UI:** Hinweis, dass keine passende Anlage gefunden wurde; keine leere Tabelle ohne Erklärung.<br>**API:** `200 OK` mit `"results": []`. | offen |
| AT-02-09 | AK: Bei API-Ausfall verständliche Fehlermeldung statt Absturz; NF-07 | Alternativszenario | Finanzdaten-API im Mock auf „nicht erreichbar“ gestellt; Suchbegriff wurde noch nicht gesucht (kein Cache) | Suchbegriff `EUNL` eingeben | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“; die Seite bleibt bedienbar, eine neue Suche ist möglich.<br>**API:** `503 UPSTREAM_UNAVAILABLE`, „Kursdaten sind gerade nicht verfügbar.“, Antwort innerhalb von 5 s. | offen |
| AT-02-10 | AK: Bei API-Ausfall verständliche Fehlermeldung statt Absturz; NF-07 (Cache) | Alternativszenario | `EUNL` wurde vor kurzem erfolgreich gesucht (Ergebnis im Cache); danach Finanzdaten-API im Mock auf „nicht erreichbar“ gestellt | Suchbegriff `EUNL` erneut eingeben | **UI:** Trefferliste wie in AT-02-01, keine Fehlermeldung.<br>**API:** `200 OK` mit dem Ergebnis aus dem Cache.<br>[OFFEN: Ob Suchergebnisse bei einem API-Ausfall aus dem Cache beantwortet werden und wie lange sie gültig sind, ist nicht festgelegt. NF-07 nennt nur Kursdaten jünger als 24 h.] | offen |
| AT-02-11 | NF-02: Antwortzeit ≤ 1 s bei Cache-Treffer | NFR | Suchbegriff `msci` wurde unmittelbar zuvor gesucht (Cache-Treffer) | Suche `msci` 5-mal wiederholen und die Antwortzeit messen (Netzwerk-Tab der Entwicklertools oder HTTP-Client) | Jede der 5 Antworten liegt bei ≤ 1 s.<br>**API:** `200 OK`. | offen |
| AT-02-12 | NF-02: Antwortzeit ≤ 3 s ohne Cache | NFR | Cache geleert; Finanzdaten-API erreichbar | 5 verschiedene, bisher nicht gesuchte Suchbegriffe eingeben und jeweils die Antwortzeit messen | Jede der 5 Antworten liegt bei ≤ 3 s.<br>**API:** `200 OK`. | offen |
| AT-02-13 | NF-07: Antwort ≤ 5 s auch bei ausbleibender Antwort der Finanzdaten-API | NFR | Finanzdaten-API im Mock auf „antwortet nicht“ gestellt (Verbindung bleibt hängen statt abgelehnt zu werden); kein Cache | Suchbegriff `EUNL` eingeben und Zeit bis zur Meldung messen | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“ spätestens nach 5 s; die Seite bleibt bedienbar.<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |

---

## US-03 Portfolio mit Gewichtungen erstellen

Vorbedingung aller Testfälle: Die Anlagen EUNL (`assetId` 17) und IS3N (`assetId` 23) wurden über die Suche (US-02) gefunden. Gewichtungen werden in Prozent angegeben.

| ID | Bezug | Art | Vorbedingung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-03-01 | AK: Portfolio hat Namen und mindestens eine Anlage; Summe 100 % | Normalfall | Portfolio-Editor geöffnet | Name `Mein Weltportfolio`; EUNL 70 %, IS3N 30 %; speichern | **UI:** Bestätigung, dass das Portfolio gespeichert ist; es erscheint in der Portfolioliste mit beiden Positionen.<br>**API:** `POST /portfolios` → `201 Created` mit `id`, `name`, `createdAt`, `updatedAt` und beiden Positionen inkl. `weight` 70 und 30. `POST /portfolios/list` enthält das Portfolio. | offen |
| AT-03-02 | AK: mindestens eine Anlage (Grenze: 1 Anlage) | Grenzwert | Portfolio-Editor geöffnet | Name `Nur Welt`; nur EUNL 100 %; speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created` mit einer Position. | offen |
| AT-03-03 | Nachbedingung Kurzformat: Portfolio kann für eine Simulation ausgewählt werden | Normalfall | AT-03-01 bestanden | Simulationsseite öffnen und das Portfolio auswählen | **UI:** „Mein Weltportfolio“ ist auswählbar und zeigt die Positionen.<br>**API:** `POST /portfolios/{id}/details` → `200 OK` mit beiden Positionen. | offen |
| AT-03-04 | AK: Portfolio hat einen Namen | Grenzwert | Portfolio-Editor geöffnet | Name leer lassen; EUNL 70 %, IS3N 30 %; speichern | **UI:** Speichern nicht möglich; Meldung am Namensfeld.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt das Feld `name`. | offen |
| AT-03-05 | AK: Portfolio hat einen Namen (Äquivalenzklasse: nur Leerzeichen) | Alternativszenario | Portfolio-Editor geöffnet | Name aus drei Leerzeichen; EUNL 70 %, IS3N 30 %; speichern | [OFFEN: Ob ein Name nur aus Leerzeichen als leer gilt, ist nicht festgelegt. Vorschlag: wie AT-03-04 ablehnen.] | offen |
| AT-03-06 | E1: Name Pflicht, höchstens 100 Zeichen (Grenze unten) | Grenzwert | Portfolio-Editor geöffnet | Name mit 1 Zeichen: `A`; EUNL 70 %, IS3N 30 %; speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created`, `name` ist „A“. | offen |
| AT-03-07 | E1: Name höchstens 100 Zeichen (Grenze oben) | Grenzwert | Portfolio-Editor geöffnet | Name mit genau 100 Zeichen; EUNL 70 %, IS3N 30 %; speichern | **UI:** Portfolio wird gespeichert; der Name wird vollständig angezeigt.<br>**API:** `201 Created`, `name` hat 100 Zeichen. | offen |
| AT-03-08 | E1: Name höchstens 100 Zeichen (knapp darüber) | Grenzwert | Portfolio-Editor geöffnet | Name mit 101 Zeichen; EUNL 70 %, IS3N 30 %; speichern | **UI:** Speichern nicht möglich bzw. Eingabe auf 100 Zeichen begrenzt; Hinweis am Namensfeld.<br>**API (direkter Aufruf mit 101 Zeichen):** `400 VALIDATION_ERROR`, `details` nennt das Feld `name`. | offen |
| AT-03-09 | AK: Summe 100 % ± 0,01 Prozentpunkte (knapp unterhalb der Toleranz) | Grenzwert | Portfolio-Editor geöffnet | EUNL 70 %, IS3N 29,98 % (Summe 99,98 %); speichern | **UI:** Speichern nicht möglich; Meldung „Die Gewichtungen müssen zusammen 100 % ergeben.“<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, „Die Gewichtungen müssen zusammen 100 % ergeben.“, `details` mit `field` „positions“ nennt die Summe 99,98 %. | offen |
| AT-03-10 | AK: Summe 100 % ± 0,01 Prozentpunkte (untere Toleranzgrenze) | Grenzwert | Portfolio-Editor geöffnet | EUNL 70 %, IS3N 29,99 % (Summe 99,99 %); speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created`. | offen |
| AT-03-11 | AK: Summe 100 % (exakt); E2: 2 Nachkommastellen werden akzeptiert | Grenzwert | Portfolio-Editor geöffnet | EUNL 33,33 %, IS3N 33,33 %, dritte Anlage aus der Suche 33,34 % (Summe 100,00 %); speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created` mit drei Positionen. | offen |
| AT-03-12 | AK: Summe 100 % ± 0,01 Prozentpunkte (obere Toleranzgrenze) | Grenzwert | Portfolio-Editor geöffnet | EUNL 70 %, IS3N 30,01 % (Summe 100,01 %); speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created`. | offen |
| AT-03-13 | AK: Summe 100 % ± 0,01 Prozentpunkte (knapp oberhalb der Toleranz) | Grenzwert | Portfolio-Editor geöffnet | EUNL 70 %, IS3N 30,02 % (Summe 100,02 %); speichern | **UI:** Speichern nicht möglich; Meldung „Die Gewichtungen müssen zusammen 100 % ergeben.“<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt die Summe 100,02 %. | offen |
| AT-03-14 | AK: Summe ungleich 100 %, sonst ist Speichern nicht möglich | Alternativszenario | Portfolio-Editor geöffnet | EUNL 70 %, IS3N 25 % (Summe 95 %); speichern | **UI:** Speichern nicht möglich; Meldung zur Gewichtungssumme; aktuelle Summe ist erkennbar.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, „Die Gewichtungen müssen zusammen 100 % ergeben.“, `details`: „Summe der Gewichtungen ist 95 %.“ | offen |
| AT-03-15 | AK: Jede Gewichtung > 0 % (Grenze: 0 %) | Grenzwert | Portfolio-Editor geöffnet | EUNL 100 %, IS3N 0 %; speichern | **UI:** Speichern nicht möglich; Meldung an der Position IS3N.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`. | offen |
| AT-03-16 | AK: Jede Gewichtung > 0 % (knapp darüber) | Grenzwert | Portfolio-Editor geöffnet | EUNL 99,99 %, IS3N 0,01 %; speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created`, IS3N mit `weight` 0.01. | offen |
| AT-03-17 | AK: Jede Gewichtung > 0 % (negative Gewichtung) | Grenzwert | Portfolio-Editor geöffnet | EUNL 110 %, IS3N −10 % (Summe 100 %); speichern | **UI:** Speichern nicht möglich; Meldung an der Position IS3N.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`. | offen |
| AT-03-18 | AK: mindestens eine Anlage (Grenze: 0 Anlagen) | Grenzwert | Portfolio-Editor geöffnet | Name `Leer`; keine Anlage hinzufügen; speichern | **UI:** Speichern nicht möglich; Hinweis, mindestens eine Anlage hinzuzufügen.<br>**API (direkter Aufruf mit `"positions": []`):** `400 VALIDATION_ERROR`, `details` nennt `positions`. | offen |
| AT-03-19 | AK: Doppelte Anlagen werden abgelehnt | Alternativszenario | Portfolio-Editor geöffnet, EUNL bereits als Position enthalten | EUNL über die Suche ein zweites Mal hinzufügen; Gewichtungen 50 % und 50 %; speichern | **UI:** Das zweite Hinzufügen wird verhindert oder Speichern ist nicht möglich; Hinweis, dass die Anlage bereits enthalten ist.<br>**API (direkter Aufruf mit zweimal `assetId` 17):** `400 VALIDATION_ERROR`. | offen |
| AT-03-20 | AK: Jede Gewichtung > 0 % (Äquivalenzklasse: leeres Feld) | Alternativszenario | Portfolio-Editor geöffnet | EUNL 100 %, Gewichtung von IS3N leer lassen; speichern | **UI:** Speichern nicht möglich; Meldung an der Position IS3N.<br>**API (direkter Aufruf ohne `weight` bzw. mit `null`):** `400 VALIDATION_ERROR`. | offen |
| AT-03-21 | E2: höchstens 2 Nachkommastellen (3 Nachkommastellen) | Grenzwert | Portfolio-Editor geöffnet | EUNL 33,333 %, IS3N 33,333 %, dritte Anlage aus der Suche 33,334 % (Summe 100,000 %); speichern | **UI:** Speichern nicht möglich bzw. Eingabe auf 2 Nachkommastellen begrenzt; Hinweis an den betroffenen Positionen.<br>**API (direkter Aufruf mit `weight` 33.333, 33.333, 33.334):** `400 VALIDATION_ERROR`, `details` mit `field` „positions“. | offen |
| AT-03-22 | E2: exakte Dezimalrechnung im Backend (Gleitkomma-Testfall) | Grenzwert | Portfolio-Editor geöffnet | EUNL 69,99 %, IS3N 30,01 % (Summe 100,00 %); speichern | **UI:** Portfolio wird gespeichert.<br>**API:** `201 Created`; die Summe wird als genau 100,00 % erkannt, `weight` 69.99 und 30.01 werden unverändert zurückgegeben. | offen |
| AT-03-23 | E5: unbekannte `assetId` wird abgelehnt | Alternativszenario | Es existiert keine Anlage mit `assetId` 99999 | Direkter API-Aufruf `POST /portfolios` mit Name `Test`, Positionen `assetId` 17 mit 50 % und `assetId` 99999 mit 50 % | **API:** `400 VALIDATION_ERROR`; `details` enthält einen Eintrag mit `field` „positions“, dessen Meldung die ID 99999 nennt.<br>**UI:** nicht über die Oberfläche erreichbar, da Anlagen nur aus der Suche übernommen werden. | offen |

---

## US-04 Monte-Carlo-Simulation durchführen (Kern-Story)

Vorbedingung aller Testfälle, wenn nicht anders angegeben: „Mein Weltportfolio“ aus AT-03-01 ist gespeichert und auf der Simulationsseite ausgewählt; die Finanzdaten-API (Mock) ist erreichbar. Eingaben siehe „Basiseingaben US-04“ in den Vorbemerkungen.

### Normalfälle und Alternativszenarien

| ID | Bezug | Art | Vorbedingung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-04-01 | AK: Pflichtfelder; Standard 10.000 Pfade; Seed wird ohne Angabe erzeugt und angezeigt; Aktivitätsdiagramm: Hauptpfad, Zweig „ohne Seed“ | Normalfall | siehe oben | Basiseingaben ohne Pfadanzahl und ohne Seed; Simulation starten | **UI:** Fächerdiagramm über 20 Jahre; der vom System erzeugte Seed wird angezeigt; als Pfadanzahl ist 10.000 erkennbar.<br>**API:** `POST /simulations` → `200 OK`; `parameters.paths` = 10000; `parameters.seed` ist eine ganze Zahl; `timeline` hat 241 Punkte (Monat 0 bis 240); `id` ist `null` (keine Speicherung in Sprint 1). | offen |
| AT-04-02 | AK: Pfadanzahl und Seed optional wählbar; Aktivitätsdiagramm: Zweig „mit Seed“ | Normalfall | siehe oben | Basiseingaben, zusätzlich Pfadanzahl 20000 und Seed 42; Simulation starten | **UI:** Fächerdiagramm; angezeigter Seed ist 42.<br>**API:** `200 OK`; `parameters.paths` = 20000, `parameters.seed` = 42. | offen |
| AT-04-03 | AK: Rendite und Volatilität werden aus historischen Kursdaten geschätzt | Normalfall | siehe oben | Basiseingaben; Simulation starten; Antwort der API ansehen | **API:** `200 OK`; `estimates` enthält `historyFrom`, `historyTo` und je Anlage des Portfolios (`assetId` 17 und 23) einen Eintrag mit `annualReturn` und `annualVolatility`. Die Werte passen zu den synthetischen Kursreihen des Mocks (genaue Prüfung im Unit-Test, siehe unten). | offen |
| AT-04-04 | AK: Fächerdiagramm zeigt 5., 50. und 95. Perzentil über den Anlagehorizont | Normalfall | siehe oben | Basiseingaben; Simulation starten; Diagramm prüfen | **UI:** Diagramm zeigt drei beschriftete Verläufe bzw. ein Band für das 5., 50. und 95. Perzentil; die Zeitachse reicht von Monat 0 bis Jahr 20; zum Start liegen alle drei Werte beim Startkapital von 10.000 €.<br>**API:** In jedem Punkt von `timeline` gilt `p5` ≤ `p50` ≤ `p95`; in Monat 0 sind alle drei Werte 10000. | offen |
| AT-04-05 | Aktivitätsdiagramm: Rücksprung „Eingaben ungültig“ → Eingabe korrigieren | Alternativszenario | siehe oben | Anlagehorizont 0 eingeben, Simulation starten; danach Anlagehorizont auf 20 korrigieren und erneut starten | **UI:** Zuerst Fehlermeldung am Feld Anlagehorizont, kein Diagramm; nach der Korrektur wird die Simulation ohne erneute Portfolioauswahl berechnet und angezeigt.<br>**API:** erste Anfrage `400 VALIDATION_ERROR`, zweite Anfrage `200 OK`. | offen |
| AT-04-06 | Aktivitätsdiagramm: Portfolio nicht vorhanden | Alternativszenario | Es existiert kein Portfolio mit der `portfolioId` 9999 | Direkter API-Aufruf `POST /simulations` mit `portfolioId` 9999 und gültigen Basiseingaben | **API:** `404 NOT_FOUND`, „Portfolio nicht gefunden.“<br>**UI:** in Sprint 1 nicht über die Oberfläche erreichbar, da Portfolios noch nicht gelöscht werden können. | offen |
| AT-04-07 | AK: Kursdaten nicht abrufbar → verständliche Fehlermeldung statt Absturz; Aktivitätsdiagramm: „nicht verfügbar“; NF-07 | Alternativszenario | Finanzdaten-API im Mock auf „nicht erreichbar“ gestellt; keine Kursdaten des Portfolios im Cache | Basiseingaben; Simulation starten | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“; kein Diagramm; die Eingaben bleiben erhalten, ein erneuter Start ist möglich.<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |
| AT-04-08 | NF-07: Kursdaten jünger als 24 h werden weiterverwendet | Alternativszenario | Kursdaten beider Anlagen liegen seit 23 h im Cache; danach Finanzdaten-API im Mock auf „nicht erreichbar“ gestellt | Basiseingaben mit Seed 42; Simulation starten | **UI:** Fächerdiagramm wie im Normalfall, keine Fehlermeldung.<br>**API:** `200 OK`; Ergebnis identisch mit einer Simulation mit Seed 42 bei erreichbarer API und denselben Kursdaten. | offen |
| AT-04-09 | NF-07: Kursdaten älter als 24 h werden nicht weiterverwendet | Alternativszenario | Kursdaten beider Anlagen liegen seit 25 h im Cache; Finanzdaten-API im Mock auf „nicht erreichbar“ gestellt | Basiseingaben; Simulation starten | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |
| AT-04-10 | AK: weniger als 3 Jahre Kurshistorie → Hinweis nennt die betroffene Anlage; E6 (eine Anlage betroffen); Aktivitätsdiagramm: „< 3 Jahre“ | Alternativszenario | Portfolio aus EUNL 50 % und einer Testanlage mit kurzer Historie 50 % ist gespeichert und ausgewählt | Basiseingaben; Simulation starten | **UI:** Hinweis, dass weniger als 3 Jahre Kursdaten vorliegen, mit dem Namen der Testanlage (nicht EUNL); kein Diagramm.<br>**API:** `422 INSUFFICIENT_DATA`; `message` nennt den Namen der Testanlage; `details` enthält genau einen Eintrag mit `assetId` und `name` der Testanlage. | offen |
| AT-04-37 | AK: weniger als 3 Jahre Kurshistorie; E6 (zwei Anlagen betroffen) | Alternativszenario | Portfolio aus EUNL 50 % und den beiden Testanlagen mit kurzer Historie je 25 % ist gespeichert und ausgewählt | Basiseingaben; Simulation starten | **UI:** Hinweis mit den Namen beider Testanlagen (nicht EUNL); kein Diagramm.<br>**API:** `422 INSUFFICIENT_DATA`; `message` nennt die Namen beider Testanlagen; `details` enthält genau zwei Einträge, je Testanlage einer mit `assetId` und `name`. | offen |

### Grenzwerte

| ID | Bezug | Art | Vorbedingung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-04-11 | AK: mindestens 3 Jahre Kurshistorie (knapp darunter) | Grenzwert | Portfolio nur aus der Testanlage mit einer Historie knapp unter 3 Jahren (100 %) | Basiseingaben; Simulation starten | **UI:** Hinweis mit dem Namen der Testanlage; kein Diagramm.<br>**API:** `422 INSUFFICIENT_DATA`; `message` nennt den Namen der Testanlage; `details` enthält einen Eintrag mit `assetId` und `name` der Testanlage.<br>[OFFEN: Definition von „3 Jahre Kurshistorie“ (Kalenderzeitraum, Anzahl Monats- oder Handelstagswerte) und damit der genaue Wert „knapp darunter“.] | offen |
| AT-04-12 | AK: mindestens 3 Jahre Kurshistorie (genau auf der Grenze) | Grenzwert | Portfolio nur aus der Testanlage mit genau 3 Jahren Historie (100 %) | Basiseingaben; Simulation starten | **UI:** Fächerdiagramm, kein Hinweis.<br>**API:** `200 OK`. | offen |
| AT-04-13 | AK: Anlagehorizont 1 bis 40 ganze Jahre (knapp unterhalb) | Grenzwert | siehe oben | Anlagehorizont 0 | **UI:** Fehlermeldung am Feld Anlagehorizont; keine Simulation.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `horizonYears`. | offen |
| AT-04-14 | AK: Anlagehorizont 1 bis 40 (untere Grenze) | Grenzwert | siehe oben | Anlagehorizont 1 | **UI:** Fächerdiagramm über 12 Monate.<br>**API:** `200 OK`; `timeline` hat 13 Punkte (Monat 0 bis 12). | offen |
| AT-04-15 | AK: Anlagehorizont 1 bis 40 (obere Grenze) | Grenzwert | siehe oben | Anlagehorizont 40 | **UI:** Fächerdiagramm über 40 Jahre.<br>**API:** `200 OK`; `timeline` hat 481 Punkte (Monat 0 bis 480). | offen |
| AT-04-16 | AK: Anlagehorizont 1 bis 40 (knapp oberhalb) | Grenzwert | siehe oben | Anlagehorizont 41 | **UI:** Fehlermeldung am Feld Anlagehorizont.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `horizonYears`. | offen |
| AT-04-17 | AK: Anlagehorizont in ganzen Jahren (Kommazahl) | Grenzwert | siehe oben | Anlagehorizont 2,5 | **UI:** Fehlermeldung am Feld Anlagehorizont.<br>**API (direkter Aufruf mit `horizonYears` 2.5):** `400 VALIDATION_ERROR`. | offen |
| AT-04-18 | AK: negative Werte werden abgelehnt (Anlagehorizont) | Grenzwert | siehe oben | Anlagehorizont −5 | **UI:** Fehlermeldung am Feld Anlagehorizont.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`. | offen |
| AT-04-19 | AK: Pflichtfeld, leere Werte werden abgelehnt (Anlagehorizont) | Grenzwert | siehe oben | Feld Anlagehorizont leer lassen | **UI:** Fehlermeldung am Feld Anlagehorizont.<br>**API (direkter Aufruf ohne `horizonYears`):** `400 VALIDATION_ERROR`. | offen |
| AT-04-20 | AK: Startkapital und Sparrate nicht beide 0 | Grenzwert | siehe oben | Startkapital 0, Sparrate 0 | **UI:** Fehlermeldung, dass Startkapital und Sparrate nicht beide 0 sein dürfen.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`. | offen |
| AT-04-21 | AK: Startkapital und Sparrate nicht beide 0 (nur Startkapital 0) | Grenzwert | siehe oben | Startkapital 0, Sparrate 200 | **UI:** Fächerdiagramm; Verlauf beginnt bei 0 €.<br>**API:** `200 OK`; in Monat 0 sind `p5`, `p50`, `p95` gleich 0. | offen |
| AT-04-22 | AK: Startkapital und Sparrate nicht beide 0 (nur Sparrate 0) | Grenzwert | siehe oben | Startkapital 10000, Sparrate 0 | **UI:** Fächerdiagramm.<br>**API:** `200 OK`. | offen |
| AT-04-23 | AK: negative Werte werden abgelehnt (Startkapital) | Grenzwert | siehe oben | Startkapital −1 | **UI:** Fehlermeldung am Feld Startkapital.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `startCapital`. | offen |
| AT-04-24 | AK: negative Werte werden abgelehnt (Sparrate) | Grenzwert | siehe oben | Sparrate −1 | **UI:** Fehlermeldung am Feld Sparrate.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `monthlyContribution`. | offen |
| AT-04-25 | AK: Pflichtfeld, leere Werte werden abgelehnt (Startkapital) | Grenzwert | siehe oben | Feld Startkapital leer lassen | **UI:** Fehlermeldung am Feld Startkapital.<br>**API (direkter Aufruf ohne `startCapital`):** `400 VALIDATION_ERROR`. | offen |
| AT-04-26 | AK: Pflichtfeld, leere Werte werden abgelehnt (Sparrate) | Grenzwert | siehe oben | Feld Sparrate leer lassen | **UI:** Fehlermeldung am Feld Sparrate.<br>**API (direkter Aufruf ohne `monthlyContribution`):** `400 VALIDATION_ERROR`. | offen |
| AT-04-27 | AK: Pfadanzahl 1.000 bis 50.000 (knapp unterhalb) | Grenzwert | siehe oben | Pfadanzahl 999 | **UI:** Fehlermeldung am Feld Pfadanzahl.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `paths`. | offen |
| AT-04-28 | AK: Pfadanzahl 1.000 bis 50.000 (untere Grenze) | Grenzwert | siehe oben | Pfadanzahl 1000 | **UI:** Fächerdiagramm.<br>**API:** `200 OK`; `parameters.paths` = 1000. | offen |
| AT-04-29 | AK: Pfadanzahl 1.000 bis 50.000 (obere Grenze) | Grenzwert | siehe oben | Pfadanzahl 50000 | **UI:** Fächerdiagramm.<br>**API:** `200 OK`; `parameters.paths` = 50000. | offen |
| AT-04-30 | AK: Pfadanzahl 1.000 bis 50.000 (knapp oberhalb) | Grenzwert | siehe oben | Pfadanzahl 50001 | **UI:** Fehlermeldung am Feld Pfadanzahl.<br>**API (direkter Aufruf):** `400 VALIDATION_ERROR`, `details` nennt `paths`. | offen |
| AT-04-31 | AK: Seed als ganze Zahl | Grenzwert | siehe oben | Seed 4,2 | **UI:** Fehlermeldung am Feld Seed.<br>**API (direkter Aufruf mit `seed` 4.2):** `400 VALIDATION_ERROR`. | offen |

### Nicht-funktionale Anforderungen

| ID | Bezug | Art | Vorbedingung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-04-32 | AK: gleicher Seed und gleiche Eingaben → identische Ergebnisse; NF-08 | NFR | siehe oben | Basiseingaben mit Seed 42 zweimal nacheinander simulieren; die beiden Antworten vergleichen | **UI:** Beide Fächerdiagramme sind identisch.<br>**API:** Beide Antworten `200 OK`; `timeline` (alle `p5`, `p50`, `p95`) und `estimates` sind in beiden Antworten identisch. | offen |
| AT-04-33 | AK: vom System erzeugter Seed ermöglicht Wiederholung; NF-08 | NFR | AT-04-01 durchgeführt, angezeigten Seed notiert | Basiseingaben erneut simulieren, diesmal mit dem notierten Seed | **UI:** Fächerdiagramm identisch mit dem aus AT-04-01.<br>**API:** `timeline` identisch mit der Antwort aus AT-04-01. | offen |
| AT-04-34 | NF-01: 10.000 Pfade, mit 5 Anlagen (Messfall), Horizont bis 30 Jahre, ≤ 3 s in 95 % von 20 Messläufen | NFR | Portfolio aus 5 Testanlagen à 20 % gespeichert; Kursdaten im Cache (Messung ohne Netzwerkzeit zur Finanzdaten-API) [OFFEN: Referenzumgebung für die Messung, z. B. Server oder CI-Runner] | `POST /simulations` mit `startCapital` 10000, `monthlyContribution` 200, `horizonYears` 30, `paths` 10000, Seed 42; 20-mal ausführen und jeweils `runtimeMs` notieren | Mindestens 19 von 20 Läufen haben `runtimeMs` ≤ 3000.<br>Die 20 Messwerte werden als Nachweis im Sprint Review dokumentiert. | offen |
| AT-04-35 | NF-07: Antwort ≤ 5 s auch bei ausbleibender Antwort der Finanzdaten-API | NFR | Finanzdaten-API im Mock auf „antwortet nicht“ gestellt (Verbindung bleibt hängen); keine Kursdaten im Cache | Basiseingaben; Simulation starten und Zeit bis zur Meldung messen | **UI:** Meldung „Kursdaten sind gerade nicht verfügbar.“ spätestens nach 5 s; die Seite bleibt bedienbar.<br>**API:** `503 UPSTREAM_UNAVAILABLE` innerhalb von 5 s. | offen |
| AT-04-36 | NF-14: Hinweis „Keine Anlageberatung“ auf jeder Ergebnisseite | NFR | siehe oben | Basiseingaben; Simulation starten; Ergebnisseite prüfen | **UI:** Der Hinweis „Keine Anlageberatung“ ist auf der Ergebnisseite ohne weiteres Klicken sichtbar. | offen |

---

## Rückverfolgbarkeitsmatrix

### US-02 Anlagen suchen

| Akzeptanzkriterium bzw. Bezug | Testfälle |
|---|---|
| Suche nach Ticker, ISIN oder Name liefert Trefferliste mit Name, Ticker, Börse und Währung | AT-02-01, AT-02-02, AT-02-03 |
| Suche startet erst ab zwei eingegebenen Zeichen | AT-02-05, AT-02-06, AT-02-07 |
| Bei keinem Treffer erscheint ein Hinweis | AT-02-08 |
| E4: Suchparameter `limit` 1 bis 25, Standard 10, sonst Ablehnung | AT-02-14, AT-02-15, AT-02-16, AT-02-17, AT-02-18 |
| Bei Ausfall der Finanzdaten-API verständliche Fehlermeldung statt Absturz | AT-02-09, AT-02-10, AT-02-13 |
| Nachbedingung: ausgewählte Anlage kann in ein Portfolio übernommen werden | AT-02-04 |

### US-03 Portfolio mit Gewichtungen erstellen

| Akzeptanzkriterium bzw. Bezug | Testfälle |
|---|---|
| Ein Portfolio hat einen Namen und mindestens eine Anlage | AT-03-01, AT-03-02, AT-03-04, AT-03-05, AT-03-06, AT-03-18 |
| E1: Name Pflicht, höchstens 100 Zeichen | AT-03-04, AT-03-06, AT-03-07, AT-03-08 |
| Gewichtungen ergeben zusammen 100 % (± 0,01 Prozentpunkte), sonst kein Speichern | AT-03-09, AT-03-10, AT-03-11, AT-03-12, AT-03-13, AT-03-14, AT-03-22 |
| E2: höchstens 2 Nachkommastellen; exakte Dezimalrechnung im Backend | AT-03-11, AT-03-16, AT-03-21, AT-03-22 |
| Jede Gewichtung größer als 0 % | AT-03-15, AT-03-16, AT-03-17, AT-03-20 |
| Doppelte Anlagen werden abgelehnt | AT-03-19 |
| E5: unbekannte `assetId` wird abgelehnt | AT-03-23 |
| Nachbedingung: Portfolio kann für eine Simulation ausgewählt werden | AT-03-03 |

### US-04 Monte-Carlo-Simulation durchführen

| Akzeptanzkriterium bzw. Bezug | Testfälle |
|---|---|
| Startkapital, Sparrate und Anlagehorizont sind Pflichtfelder; negative oder leere Werte werden abgelehnt | AT-04-01, AT-04-18, AT-04-19, AT-04-23, AT-04-24, AT-04-25, AT-04-26 |
| Anlagehorizont zwischen 1 und 40 ganzen Jahren | AT-04-13, AT-04-14, AT-04-15, AT-04-16, AT-04-17 |
| Startkapital und Sparrate nicht beide 0 | AT-04-20, AT-04-21, AT-04-22 |
| Pfadanzahl optional 1.000 bis 50.000, Standard 10.000 | AT-04-01, AT-04-02, AT-04-27, AT-04-28, AT-04-29, AT-04-30 |
| Seed optional als ganze Zahl; ohne Angabe erzeugt und angezeigt | AT-04-01, AT-04-02, AT-04-31 |
| Gleicher Seed und gleiche Eingaben → identische Ergebnisse | AT-04-32, AT-04-33 |
| Rendite und Volatilität aus historischen Kursdaten geschätzt | AT-04-03 |
| Weniger als 3 Jahre Kurshistorie → Hinweis mit betroffener Anlage; E6: alle betroffenen Anlagen mit Namen | AT-04-10, AT-04-11, AT-04-12, AT-04-37 |
| Kursdaten nicht abrufbar → verständliche Fehlermeldung statt Absturz | AT-04-07, AT-04-08, AT-04-09, AT-04-35 |
| Fächerdiagramm mit 5., 50. und 95. Perzentil über den Anlagehorizont | AT-04-04, AT-04-14, AT-04-15 |
| Aktivitätsdiagramm: Rücksprung bei ungültigen Eingaben | AT-04-05 |
| Aktivitätsdiagramm: Portfolio nicht vorhanden | AT-04-06 |

### Nicht-funktionale Anforderungen

| NFR | Testfälle |
|---|---|
| NF-01 Performance der Simulation | AT-04-34 |
| NF-02 Performance der Suche | AT-02-11, AT-02-12 |
| NF-07 Ausfallsicherheit | AT-02-09, AT-02-10, AT-02-13, AT-04-07, AT-04-08, AT-04-09, AT-04-35 |
| NF-08 Reproduzierbarkeit | AT-04-32, AT-04-33 |
| NF-14 Rechtliche Abgrenzung | AT-04-36 |

Die übrigen NFRs werden nicht über Abnahmetestfälle, sondern über die Prüfmethoden in Abschnitt 10 des Testkonzepts nachgewiesen.

---

## Zusätzliche Umsetzung als automatisierte Tests

Die folgenden Testfälle werden zusätzlich als automatisierte Tests im Backend umgesetzt, damit sie bei jedem Pull Request in der CI laufen. Grundlage sind die Unit-Test-Tasks BE #32, BE #38 und BE #41.

| Komponente | Teststufe | Abgedeckte Testfälle | Inhalt |
|---|---|---|---|
| `Portfolio.validate()` | Unit-Test (JUnit 5) | AT-03-02, AT-03-04 bis AT-03-22 | Alle Grenzwerte zu Name, Anzahl Anlagen, Einzelgewichtung, Nachkommastellen, Gewichtungssumme (99,98 / 99,99 / 100,00 / 100,01 / 100,02 %, 69,99 + 30,01 %) und doppelten Anlagen als parametrisierte Tests mit `BigDecimal` |
| `PortfolioService` | Unit-Test mit Mockito | AT-03-23 | Unbekannte `assetId` wird vor dem Speichern abgelehnt; die ID steht in `details` |
| `Portfolio.getWeights()` | Unit-Test | AT-03-01 | 70 % / 30 % ergibt die Anteile 0,70 und 0,30 in fester Reihenfolge (ADR-03) |
| `ParameterEstimator` | Unit-Test | AT-04-03, AT-04-10 bis AT-04-12, AT-04-37 | Kursreihe mit konstantem Wachstum ergibt bekannte Rendite und Volatilität 0; Historie knapp unter bzw. genau 3 Jahre |
| `MonteCarloEngine` | Unit-Test | AT-04-01, AT-04-14, AT-04-15, AT-04-21, AT-04-22, AT-04-32, AT-04-33 | Anzahl der Pfade und Monatsschritte; bei Volatilität 0 exakt berechenbarer Endwert aus Startkapital, Sparrate und Rendite; gleicher Seed ergibt identische Ergebnisse (NF-08) |
| `MetricsCalculator` bzw. Perzentilberechnung | Unit-Test | AT-04-04 | Perzentile einer bekannten Werteliste; `p5` ≤ `p50` ≤ `p95` |
| `SimulationService` | Unit-Test mit Mockito | AT-04-01, AT-04-06 bis AT-04-10, AT-04-37 | Aufrufreihenfolge laut ADR-02; Seed-Erzeugung ohne Angabe; Weitergabe der Fehlerfälle Portfolio fehlt, Kursdaten nicht verfügbar, Historie zu kurz (alle betroffenen Anlagen) |
| `MarketDataProvider` mit Cache | Unit-Test | AT-04-08, AT-04-09 | Kursdaten jünger als 24 h werden bei Ausfall verwendet, ältere nicht (NF-07) |
| `SimulationController` | Integrationstest (`@WebMvcTest`, `MockMvc`) | AT-04-13 bis AT-04-31, AT-04-06, AT-04-07, AT-04-10, AT-04-37 | HTTP-Status und Fehlercodes `400 VALIDATION_ERROR`, `404 NOT_FOUND`, `422 INSUFFICIENT_DATA`, `503 UPSTREAM_UNAVAILABLE` im Fehlerformat des API-Designs |
| Portfolio-Endpunkt | Integrationstest | AT-03-01, AT-03-08, AT-03-09, AT-03-14, AT-03-18, AT-03-19, AT-03-21, AT-03-23 | `201 Created` und `400 VALIDATION_ERROR` mit `details` |
| Such-Endpunkt | Integrationstest | AT-02-01, AT-02-05, AT-02-07, AT-02-08, AT-02-09, AT-02-14 bis AT-02-18 | `200 OK`, leere Trefferliste, Grenzwerte von `limit`, `400 VALIDATION_ERROR`, `503 UPSTREAM_UNAVAILABLE` |
| Frontend-Prüflogik | Unit-Test (Jest) | AT-02-05, AT-02-06, AT-03-09 bis AT-03-13, AT-03-21, AT-03-22 | Mindestlänge des Suchbegriffs; Nachkommastellen; Berechnung und Prüfung der Gewichtungssumme |

Alle automatisierten Tests verwenden den `MockMarketDataProvider` und laufen ohne Internetzugang (NF-12). Die Performancemessung AT-04-34 wird nicht in der CI ausgeführt, weil die Laufzeit auf geteilten CI-Runnern schwankt; sie wird vor dem Sprint Review in der Referenzumgebung durchgeführt.
