# Kurzformat-Tabellen – Sprint 2

| | |
|---|---|
| **Projekt** | LookUpStock |
| **Sprint** | 2 (05.10.2026 – 11.10.2026) |
| **Issue** | Docs #56 Kurzformat-Tabelle für US-01 |
| **Verantwortlich** | Mert Eminel (KujoBaraka), Product Owner |
| **Version** | 1 |
| **Stand** | 07.10.2026 |
| **Grundlagen** | Vorlage Use-Case-Kurzbeschreibung (Vorlesung, Folie 20), Akzeptanzkriterien aus dem Sprint-Planning-Protokoll Sprint 2 (Abschnitt 4), API-Design (Stand 02.10.2026, Abschnitt „Auth“), NFRs nach FURPS+ (NF-03, NF-04, NF-16), Akteursbeschreibung v1, Use-Case-Diagramm v1, Glossar |
| **KI-Nutzung** | Entwurf mit KI-Unterstützung erstellt und vom Product Owner geprüft, siehe KI-Log |

Diese Datei setzt `kurzformat_sprint-1.md` fort. Die Tabellen aus Sprint 1 bleiben als Stand von Sprint 1 unverändert; wie sich US-02 bis US-04 durch die Anmeldung ändern, steht im Abschnitt „Auswirkungen von US-01 auf US-02 bis US-04“.

US-01 wird in zwei Tabellen beschrieben, weil die Story zwei Anwendungsfälle mit unterschiedlichen primären Akteuren umfasst. So ist es auch im Use-Case-Diagramm dargestellt: „Registrieren“ löst der Gast aus, „An- und abmelden“ der registrierte Nutzer.

---

## US-01 (Teil 1) Registrieren

| **Name** | **Registrieren** |
|---|---|
| **Art** | User Story (US-01), Teil 1 von 2 |
| **Kurzbeschreibung** | Ein Gast legt mit seiner E-Mail-Adresse und einem Passwort ein Nutzerkonto an, damit er anschließend eigene Portfolios anlegen, simulieren und speichern kann. |
| **Auslöser** | Der Gast möchte eigene Portfolios anlegen und speichern. |
| **Akteure** | Gast (primär) |
| **Eingehende Information** | E-Mail-Adresse (gültiges Format), Passwort (mindestens 8 Zeichen) |
| **Vorbedingungen** | Die Anwendung ist im Browser geöffnet; der Gast ist nicht angemeldet. |
| **Nachbedingungen** | • Für die E-Mail-Adresse ist ein Nutzerkonto angelegt.<br>• Das Passwort ist nur als Hash gespeichert, nie im Klartext (NF-03).<br>• Der Gast kann sich mit E-Mail-Adresse und Passwort anmelden (Teil 2); er ist nach der Registrierung noch nicht angemeldet (E13).<br>• Ist die E-Mail-Adresse bereits registriert, wird kein weiteres Konto angelegt und der Hinweis „Diese E-Mail ist bereits registriert.“ erscheint. |
| **Essenzielle Schritte** | • E-Mail-Adresse und Passwort eingeben<br>• Eingaben prüfen<br>• Nutzerkonto anlegen<br>• Registrierung bestätigen |

---

## US-01 (Teil 2) An- und abmelden

| **Name** | **An- und abmelden** |
|---|---|
| **Art** | User Story (US-01), Teil 2 von 2 |
| **Kurzbeschreibung** | Ein registrierter Nutzer meldet sich mit E-Mail-Adresse und Passwort an, um mit seinen eigenen Portfolios zu arbeiten, und meldet sich danach wieder ab. |
| **Auslöser** | Der Nutzer möchte auf seine Portfolios zugreifen oder seine Anmeldung beenden. |
| **Akteure** | Registrierter Nutzer (primär) |
| **Eingehende Information** | Anmelden: E-Mail-Adresse und Passwort. Abmelden: keine Eingabe. |
| **Vorbedingungen** | Für den Nutzer besteht ein Nutzerkonto (Teil 1). Zum Abmelden ist der Nutzer angemeldet. |
| **Nachbedingungen** | • Nach erfolgreicher Anmeldung ist der Nutzer 24 Stunden angemeldet (NF-16) und sieht ausschließlich seine eigenen Portfolios (NF-04).<br>• Bei falschen Anmeldedaten erscheint „E-Mail oder Passwort ist falsch.“; der Hinweis verrät nicht, ob die E-Mail-Adresse registriert ist.<br>• Nach dem Abmelden oder nach Ablauf der 24 Stunden ist die Anmeldung ungültig; geschützte Funktionen verlangen eine neue Anmeldung (E14). |
| **Essenzielle Schritte** | • E-Mail-Adresse und Passwort eingeben<br>• Anmeldedaten prüfen<br>• Anmeldung bestätigen und eigene Portfolios bereitstellen<br>• Abmelden |

---

## Auswirkungen von US-01 auf US-02 bis US-04

Laut Sprint-Planning-Protokoll Sprint 2 gilt ab diesem Sprint die Vorbedingung „Nutzer ist angemeldet“, soweit der Endpunkt laut API-Design eine Anmeldung erfordert.

| Story | Feld | Sprint 1 | Ab Sprint 2 | Grundlage |
|---|---|---|---|---|
| US-02 Anlagen suchen | – | Ohne Anmeldung nutzbar | **Unverändert.** Die Suche bleibt auch für Gäste ohne Anmeldung nutzbar. | API-Design: öffentliche Abfragen bleiben GET; Use-Case-Diagramm v1: Gast – Anlagen suchen |
| US-03 Portfolio erstellen | Akteure | Nutzer | Registrierter Nutzer | Akteursbeschreibung v1 |
| | Vorbedingungen | Anlagen wurden über die Anlagensuche gefunden. | Zusätzlich: Der Nutzer ist angemeldet. | Sprint-Planning-Protokoll Sprint 2, Abschnitt 4 |
| | Nachbedingungen | Portfolio ist gespeichert. | Zusätzlich: Das Portfolio ist dem angemeldeten Nutzer zugeordnet und nur für ihn sichtbar. | NF-04; in Sprint 1 bewusst gestrichen, da es noch keine Anmeldung gab |
| US-04 Simulation durchführen | Akteure | Nutzer (primär), Twelve Data (sekundär) | Registrierter Nutzer (primär), Twelve Data (sekundär) | Akteursbeschreibung v1 |
| | Vorbedingungen | Es wurde mindestens ein Portfolio erstellt. | Der Nutzer ist angemeldet und hat mindestens ein eigenes Portfolio erstellt. | Sprint-Planning-Protokoll Sprint 2, Abschnitt 4 |
| | Nachbedingungen | – | Zusätzlich: Simuliert werden kann nur ein eigenes Portfolio; ein fremdes Portfolio wird wie ein nicht vorhandenes behandelt. | NF-04 (fremde Ressourcen liefern 404) |

---

## Festlegungen des Product Owners (07.10.2026)

Anlass: Abgleich der Akzeptanzkriterien von US-01 mit dem API-Design. Die Nummerierung setzt E1 bis E10 aus Sprint 1 fort.

| Nr. | Story | Festlegung | Begründung |
|---|---|---|---|
| E11 | US-01 | Die E-Mail-Adresse wird ohne Unterscheidung von Groß- und Kleinschreibung verglichen (`Anna@example.com` und `anna@example.com` sind dieselbe Adresse). | Verhindert zwei Konten für dieselbe Adresse und eine scheiternde Anmeldung wegen der Schreibweise. |
| E12 | US-01 | Eine E-Mail-Adresse ohne gültiges Format wird mit dem Hinweis „Bitte gib eine gültige E-Mail-Adresse ein.“ abgelehnt. | Das API-Design legt bisher nur den Hinweis zum Passwort fest; für Oberfläche und Abnahmetest wird ein einheitlicher Text benötigt. |
| E13 | US-01 | Nach der Registrierung ist der Nutzer noch nicht angemeldet; er meldet sich anschließend mit seinen Daten an. | `POST /auth/register` liefert laut API-Design keinen Token. |
| E14 | US-01 | Ist die Anmeldung abgelaufen, wird der Nutzer mit dem Hinweis „Deine Anmeldung ist abgelaufen. Bitte melde dich erneut an.“ zur Anmeldung geführt. Ist sie aus einem anderen Grund ungültig (z. B. nach dem Abmelden), lautet der Hinweis „Bitte melde dich an.“ | Der Nutzer versteht, warum er sich erneut anmelden muss. Das Frontend erkennt den Ablauf selbst über `expiresAt` aus der Login-Antwort; das Backend bleibt unverändert (401 mit „Bitte melde dich an.“). |

---

## Akzeptanzkriterien

Zur Übernahme in das Story-Issue US-01 (Docs #9). Grundlage sind die Kriterien aus dem Sprint-Planning-Protokoll Sprint 2; Präzisierungen durch die Festlegungen E11 bis E14 sind gekennzeichnet.

**US-01 Registrieren und anmelden**

- Die Registrierung erfolgt mit E-Mail-Adresse und Passwort. Ein Passwort mit weniger als 8 Zeichen wird mit „Das Passwort muss mindestens 8 Zeichen haben.“ abgelehnt.
- Eine E-Mail-Adresse ohne gültiges Format wird abgelehnt. *(E12)*
- Eine bereits registrierte E-Mail-Adresse wird mit dem Hinweis „Diese E-Mail ist bereits registriert.“ abgelehnt; Groß- und Kleinschreibung spielen dabei keine Rolle. *(E11)*
- Passwörter werden nur gehasht gespeichert (NF-03).
- Nach der Registrierung meldet sich der Nutzer mit seinen Daten an. *(E13)*
- Falsche Anmeldedaten führen zur Meldung „E-Mail oder Passwort ist falsch.“, ohne zu verraten, ob die E-Mail-Adresse existiert.
- Eine Anmeldung ist 24 Stunden gültig (NF-16).
- Ein Nutzer sieht nur seine eigenen Portfolios; der Zugriff auf fremde Portfolios liefert 404 (NF-04).
- Ein angemeldeter Nutzer kann sich abmelden; die Anmeldung ist danach ungültig.
- Ist die Anmeldung abgelaufen, wird der Nutzer mit „Deine Anmeldung ist abgelaufen. Bitte melde dich erneut an.“ zur Anmeldung geführt; bei sonst ungültiger Anmeldung mit „Bitte melde dich an.“ *(E14)*

---

## Änderungshistorie

| Version | Datum | Story | Änderung | Anlass |
|---|---|---|---|---|
| 1 | 07.10.2026 | US-01 | Neu: Kurzformat-Tabellen „Registrieren“ und „An- und abmelden“, Akzeptanzkriterien | Docs #56, Sprint-Planning-Protokoll Sprint 2 |
| 1 | 07.10.2026 | US-03, US-04 | Vorbedingung „Nutzer ist angemeldet“, Akteur „Registrierter Nutzer“, Zuordnung der Portfolios zum Nutzer | US-01, NF-04 |
| 1 | 07.10.2026 | US-02 | Ausdrücklich ohne Anmeldung nutzbar | API-Design (öffentliche GET-Abfrage) |
| 1 | 07.10.2026 | US-01 | Festlegungen E11 bis E14 | Abgleich mit dem API-Design |
