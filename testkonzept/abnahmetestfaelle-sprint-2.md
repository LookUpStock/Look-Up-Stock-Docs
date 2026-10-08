# Abnahmetestfälle – Sprint 2

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #61 Abnahmetestfälle Sprint 2 und Fortschreibung des Testkonzepts (US-01, NF-03, NF-04) |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Stand** | 08.10.2026 |
| **Stories** | US-01 Registrieren und anmelden (Docs #9, neu); US-02 (Docs #8), US-03 (Docs #1) und US-04 (Docs #7, Kern-Story) aus Sprint 1 übernommen |
| **Grundlagen** | Akzeptanzkriterien und Festlegungen E11 bis E14 in `anforderungen/kurzformat_sprint-2.md`, Sprint-Planning-Protokoll Sprint 2, API-Design (Stand 02.10.2026, Abschnitt „Auth“), NF-03, NF-04, NF-14, NF-16, Backend-Umsetzung BE #30 (Stand 08.10.2026), [Testkonzept](testkonzept.md) Version 2 |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt [abnahmetestfaelle-sprint-1.md](abnahmetestfaelle-sprint-1.md) fort. Sie enthält die neuen Testfälle für US-01, die Ergänzungen für US-02 bis US-04 durch die Anmeldung und die Korrekturen an den aus Sprint 1 übernommenen Testfällen.

---

## Vorbemerkungen

Es gelten die Vorbemerkungen aus Sprint 1 (Bezug, Durchführung, Testdaten, Darstellung von UI und API, Status). Zusätzlich gilt:

- **Anmeldung:** Ab Sprint 2 wird bei geschützten Endpunkten das Feld `token` im JSON-Body geprüft (API-Design). Manuelle Testfälle zu US-03 und US-04 werden mit angemeldetem Testnutzer A durchgeführt, wenn nicht anders angegeben.
- **Testnutzer** (Testkonzept, Abschnitt 6):
  - **Nutzer A:** `anna@example.com`, Passwort `geheim123`, besitzt „Mein Weltportfolio“ (EUNL 70 %, IS3N 30 %)
  - **Nutzer B:** `ben@example.com`, Passwort `sicher456`, ohne Portfolio
- **Meldungstexte:** Es gelten die Texte aus dem API-Design und aus `kurzformat_sprint-2.md` (PO-Entscheidung vom 08.10.2026). Abweichende Texte im Code gelten als Fehler.
- **Zeitabhängige Fälle:** Der Ablauf einer Anmeldung nach 24 Stunden wird nicht abgewartet, sondern automatisiert mit einer festgelegten Uhrzeit geprüft. Das Backend bekommt die Uhr über `ClockConfig` und kann sie im Test vorgeben.

**Anzahl der neuen Testfälle**

| Story | Normalfall | Alternativszenario | Grenzwert | NFR | Summe | davon manuell | davon automatisiert |
|---|---|---|---|---|---|---|---|
| US-01 | 3 | 4 | 3 | 4 | 14 | 8 | 6 |
| US-02 | 1 | 0 | 0 | 0 | 1 | 1 | 0 |
| US-03 | 1 | 1 | 0 | 0 | 2 | 1 | 1 |
| US-04 | 0 | 1 | 0 | 0 | 1 | 0 | 1 |
| **Gesamt** | 5 | 6 | 3 | 4 | 18 | 10 | 8 |

---

## US-01 Registrieren und anmelden

Vorbedingung: Die Datenbank enthält noch keinen Nutzer `anna@example.com` (gilt für AT-01-01; die übrigen Testfälle bauen darauf auf).

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-01-01 | AK: Registrierung mit E-Mail und Passwort; E13; Nachbedingung: Konto angelegt | Normalfall | manuell | Als Gast „Registrieren“ öffnen; `anna@example.com` und `geheim123` eingeben; absenden | **UI:** Bestätigung der Registrierung und Aufforderung, sich anzumelden; der Nutzer ist noch **nicht** angemeldet (E13).<br>**API:** `POST /auth/register` → `201 Created` mit `id` und `email`; die Antwort enthält weder Passwort noch Hash. | offen |
| AT-01-02 | AK: Anmeldung 24 h gültig (NF-16); E11 beim Anmelden | Normalfall | manuell | 1. Mit `anna@example.com` / `geheim123` anmelden<br>2. Abmelden, dann mit `ANNA@Example.com` / `geheim123` anmelden | **UI:** 1. und 2. Anmeldung erfolgreich; die eigene (noch leere) Portfolioliste wird angezeigt.<br>**API:** jeweils `POST /auth/login` → `200 OK` mit `token` und `expiresAt`; `expiresAt` liegt 24 h nach dem Login-Zeitpunkt. | offen |
| AT-01-03 | AK: Abmelden, Anmeldung danach ungültig; E14 (ungültige Anmeldung); NF-16 | Normalfall | manuell | Als Nutzer A angemeldet „Abmelden“ wählen; danach über die Navigation die Portfolioseite aufrufen. Zusätzlich per HTTP-Client `POST /portfolios/list` mit dem alten Token | **UI:** Nach dem Abmelden ist der Nutzer abgemeldet. Die Portfolioseite führt zur Anmeldung mit dem Hinweis „Bitte melde dich an.“<br>**API:** `POST /auth/logout` → `204 No Content`; danach `POST /portfolios/list` mit demselben Token → `401 UNAUTHORIZED`, „Bitte melde dich an.“ | offen |
| AT-01-04 | AK: bereits registrierte E-Mail wird abgelehnt; E11 | Alternativszenario | manuell | Nutzer A existiert. Registrieren mit `Anna@Example.com` und einem gültigen Passwort | **UI:** Hinweis „Diese E-Mail ist bereits registriert.“; die Eingaben bleiben erhalten.<br>**API:** `409 CONFLICT` mit dieser Meldung; es wird kein zweites Konto angelegt. | offen |
| AT-01-05 | AK: falsche Anmeldedaten verraten nicht, ob die E-Mail existiert | Alternativszenario | manuell | 1. `anna@example.com` mit Passwort `falsch123`<br>2. `niemand@example.com` mit Passwort `geheim123` | **UI:** In beiden Fällen dieselbe Meldung „E-Mail oder Passwort ist falsch.“<br>**API:** beide `401 UNAUTHORIZED` mit identischem Body. | offen |
| AT-01-06 | E14: abgelaufene Anmeldung wird erkannt und erklärt | Alternativszenario | automatisiert (Frontend-Test `auth.test.js`) | Gespeichertes `expiresAt` liegt in der Vergangenheit; Nutzer ruft eine geschützte Funktion auf | **UI:** Hinweis „Deine Anmeldung ist abgelaufen. Bitte melde dich erneut an.“ und Weiterleitung zur Anmeldung; es wird keine geschützte Anfrage an das Backend gesendet. | offen |
| AT-01-07 | Abmelden mit ungültigem oder abgelaufenem Token | Alternativszenario | automatisiert (AuthServiceTest, AuthControllerTest) | `POST /auth/logout` mit unbekanntem, mit abgelaufenem und ohne Token | **API:** jeweils `204 No Content`; kein Fehler. Das Abmelden ist wiederholbar (PO-Entscheidung vom 08.10.2026, siehe Hinweis unten). | offen |
| AT-01-08 | AK: Passwort mindestens 8 Zeichen; technische Obergrenze 72 Byte (BCrypt) | Grenzwert | automatisiert (AuthServiceTest, Frontend-Test `auth.test.js`) | Passwort mit 7, 8, 72 und 73 Byte (nur ASCII-Zeichen) | **API:** 8 und 72 → `201 Created`; 7 → `400 VALIDATION_ERROR`, „Das Passwort muss mindestens 8 Zeichen haben.“, `details` nennt `password`; 73 → `400 VALIDATION_ERROR`, `details` nennt `password`.<br>**UI:** Bei 7 Zeichen wird die Meldung schon vor dem Absenden angezeigt. | offen |
| AT-01-09 | E12: gültiges E-Mail-Format | Grenzwert | automatisiert (AuthServiceTest, Frontend-Test `auth.test.js`) | Ungültig: leer, `anna`, `anna@`, `anna@example`, `anna @example.com`, 256 Zeichen. Gültig: `a@b.de`, 255 Zeichen | **API:** gültige Werte → `201 Created`; ungültige → `400 VALIDATION_ERROR`, „Bitte gib eine gültige E-Mail-Adresse ein.“, `details` nennt `email`. | offen |
| AT-01-10 | NF-16: Anmeldung genau 24 h gültig | Grenzwert | automatisiert (AuthServiceTest mit vorgegebener Uhr) | Login um 10:00:00; Prüfung des Tokens um 09:59:59 des Folgetags, um genau 10:00:00 und um 10:00:01 | 09:59:59 → gültig; 10:00:00 und 10:00:01 → `401 UNAUTHORIZED`, „Bitte melde dich an.“ | offen |
| AT-01-11 | NF-03: Passwörter nur als gesalzener Hash | NFR | manuell (Datenbank) und automatisiert (AuthServiceTest `register_speichertNurDenHash_NF03`) | Nutzer A und Nutzer B registrieren; Nutzer C mit demselben Passwort wie A registrieren; Tabelle `app_user` ansehen | Spalte `password_hash` beginnt bei allen mit `$2a$`; kein Passwort im Klartext. A und C haben trotz gleichem Passwort **unterschiedliche** Hashes (Salz). | offen |
| AT-01-12 | NF-14 (b): nur nötige personenbezogene Daten | NFR | manuell (Datenbank und Migration V3) | Tabelle `app_user` und Migration `V3__nutzer_und_token.sql` ansehen | `app_user` enthält nur `id`, `email`, `password_hash` und `created_at`; keine weiteren personenbezogenen Daten. | offen |
| AT-01-13 | AK: Nutzer sieht nur eigene Portfolios; NF-04 | NFR | manuell | 1. Als Nutzer B anmelden und die Portfolioliste öffnen<br>2. Per HTTP-Client mit dem Token von B: `POST /portfolios/{id}/details` mit der ID von „Mein Weltportfolio“ (Nutzer A)<br>3. Als Nutzer A anmelden | **UI:** 1. Die Liste von B ist leer; „Mein Weltportfolio“ erscheint nicht. 3. Bei A ist „Mein Weltportfolio“ unverändert vorhanden.<br>**API:** 2. `404 NOT_FOUND`, „Portfolio nicht gefunden.“ (nicht `403`, damit nicht erkennbar ist, dass die ID existiert). | offen |
| AT-01-14 | NF-04: fremde Ressourcen liefern 404 | NFR | automatisiert (PortfolioControllerTest, PortfolioServiceTest, SimulationServiceTest) | Mit dem Token von B: `POST /portfolios/list`, `POST /portfolios/{idA}/details`, `POST /simulations` mit `portfolioId` von A | `list` enthält nur Portfolios von B; `details` und `simulations` → `404 NOT_FOUND`. Die Daten von A bleiben unverändert. `PUT` und `DELETE` (US-06) folgen in Sprint 3. | offen |

**Hinweis zu AT-01-07:** Das API-Design sieht für `POST /auth/logout` mit ungültigem Token bisher `401` vor. Umgesetzt (BE #30) ist ein wiederholbares Abmelden, das immer `204` liefert. Das ist für den Nutzer sinnvoller: Wer sich mit einer bereits abgelaufenen Anmeldung abmeldet, soll keine Fehlermeldung sehen. Das API-Design wird entsprechend angepasst.

---

## Ergänzungen zu US-02 bis US-04

| ID | Bezug | Art | Durchführung | Schritte / Eingabe | Erwartetes Ergebnis | Status |
|---|---|---|---|---|---|---|
| AT-02-09 | US-02 bleibt öffentlich (Kurzformat Sprint 2, Use-Case-Diagramm v2) | Normalfall | manuell | Abgemeldet (als Gast) `EUNL` suchen | **UI:** Trefferliste wie in AT-02-01, keine Aufforderung zur Anmeldung.<br>**API:** `GET /assets/search?q=EUNL` ohne Token → `200 OK`. | offen |
| AT-03-10 | US-01 „um meine Portfolios zu speichern“; Nachbedingung US-03: Portfolio ist dem Nutzer zugeordnet | Normalfall | manuell | Als Nutzer A „Mein Weltportfolio“ anlegen (wie AT-03-01); abmelden; Browser schließen; erneut als A anmelden | **UI:** „Mein Weltportfolio“ ist nach der erneuten Anmeldung in der Liste und für die Simulation auswählbar.<br>**API:** `POST /portfolios/list` mit neuem Token enthält das Portfolio. | offen |
| AT-03-11 | Vorbedingung US-03: Nutzer ist angemeldet | Alternativszenario | automatisiert (PortfolioControllerTest) | `POST /portfolios`, `POST /portfolios/list` und `POST /portfolios/{id}/details` jeweils ohne Token, mit unbekanntem und mit abgelaufenem Token | **API:** jeweils `401 UNAUTHORIZED`, „Bitte melde dich an.“; es wird nichts gespeichert. | offen |
| AT-04-16 | Vorbedingung US-04: Nutzer ist angemeldet | Alternativszenario | automatisiert (SimulationControllerTest) | `POST /simulations` mit Basiseingaben ohne Token, mit unbekanntem und mit abgelaufenem Token | **API:** jeweils `401 UNAUTHORIZED`, „Bitte melde dich an.“; es wird keine Simulation berechnet. | offen |

---

## Übernommene Testfälle aus Sprint 1

In Sprint 1 wurde keine Story abgenommen. Alle Testfälle aus `abnahmetestfaelle-sprint-1.md` werden in Sprint 2 durchgeführt. Seit Sprint 1 haben sich folgende Vorgaben geändert; bei der Durchführung gilt die rechte Spalte:

| Testfall | Bisher (Sprint 1) | Ab Sprint 2 | Grund |
|---|---|---|---|
| alle manuellen Testfälle zu US-03 und US-04 | ohne Anmeldung, Feld `token` wird ignoriert | als Nutzer A angemeldet, `token` im Body | US-01; Sprint-Planning-Protokoll Sprint 2 |
| AT-02-06 | „Hinweis, dass keine passende Anlage gefunden wurde“ | genauer Text „Keine passende Anlage gefunden.“ | PO-Entscheidung E10 |
| AT-02-08 | 1. ≤ 1 s mit Cache (eine Messung); 2. ≤ 3 s ohne Cache | 20 Suchbegriffe, deren Ergebnisse im Cache liegen; mindestens 19 Antworten ≤ 1 s. Teil 2 ohne Cache entfällt | NF-02 Version 2 |
| AT-04-09 | [OFFEN: Definition von „3 Jahre Kurshistorie“] | Testanlagen mit 35 und mit 36 Monatswerten: 35 → `422`, 36 → gültig | PO-Entscheidung E8 |
| AT-04-14 | Horizont 30 Jahre; Referenzumgebung offen | Horizont 40 Jahre; Messung auf dem Projektserver (Heroku, Docs #64) | NF-01 Version 2 |
| Frontend-Tests | Jest | Node-Test-Runner (`node:test`), wie im Frontend-Repository eingerichtet | FE #6, README Frontend |

---

## Automatisierte Tests

| Testklasse | Repository | Teststufe | Abgedeckte Testfälle | Stand 08.10.2026 |
|---|---|---|---|---|
| `AuthServiceTest` | BE | Unit-Test mit Mockito und vorgegebener Uhr | AT-01-07 bis AT-01-11 | vorhanden (BE #30); Grenzwerte 72/73 Byte, 24 h und Meldungstexte nach PO-Entscheidung anzupassen |
| `AuthControllerTest` | BE | Integrationstest (`MockMvc`) | AT-01-07 | vorhanden (BE #30) |
| `PortfolioControllerTest` | BE | Integrationstest (`MockMvc`) | AT-01-14, AT-03-11 | zu ergänzen mit BE #54 |
| `PortfolioServiceTest` | BE | Unit-Test mit Mockito | AT-01-14 | zu ergänzen mit BE #54 |
| `SimulationServiceTest` | BE | Unit-Test mit Mockito | AT-01-14 | zu ergänzen mit BE #50 |
| `SimulationControllerTest` | BE | Integrationstest (`MockMvc`) | AT-04-16 | zu ergänzen mit BE #50 |
| `auth.test.js` | FE | Unit-Test (`node:test`) | AT-01-06, AT-01-08, AT-01-09 | zu ergänzen mit FE #4 |

Die Testfälle AT-01-11 und AT-01-12 prüfen zusätzlich die Datenbankinhalte. Dafür wird in der Abnahmeumgebung lesend auf die Datenbank zugegriffen (lokal mit `psql` bzw. über die Heroku-Postgres-Konsole).
