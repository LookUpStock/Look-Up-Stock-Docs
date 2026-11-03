# 5 Implementierung: MPA und SPA

*Docs #177 · Carl Stibbe*

## 5.1 Ausgangslage und Ziel

Das LookUpStock-Frontend wurde zunächst als Multi-Page Application (MPA) umgesetzt. Die Oberfläche verteilt sich auf die HTML-Seiten `index.html`, `login.html`, `portfolio.html` und `compare.html`. Bei einer Navigation zwischen diesen Seiten fordert der Browser jeweils ein neues HTML-Dokument an. Die Seiten verwenden gemeinsame JavaScript-Module für Anmeldung, API-Aufrufe, Simulationen und Diagramme.

Im Sprint 5 kam durch die Portfolio-Guidelines die Anforderung hinzu, neben dem MPA-Touchpoint auch einen Touchpoint nach dem SPA-Prinzip bereitzustellen. Das Sprint-Planning hielt fest, dass die SPA dieselbe REST- oder GraphQL-Schnittstelle wie die MPA nutzen und keine zusätzlichen Backend-Endpunkte benötigen sollte. Der Product Owner bestimmte den Umfang: eigene Portfolios ansehen und simulieren sowie gespeicherte Simulationen ansehen. Die bestehende MPA sollte als eigener Touchpoint erhalten bleiben (Sprint-Planning-Protokoll Sprint 5, Abschnitte 1 und 4; ADR-08).

Diese Abgrenzung war wichtig: Ziel war nicht, jede MPA-Funktion ein zweites Mal zu implementieren. Die SPA sollte einen klar vorführbaren Ablauf anbieten und zugleich zeigen, dass die Ansichten clientseitig gewechselt werden, ohne ein neues HTML-Dokument zu laden.

## 5.2 Entscheidung: eigener Einstieg und Hash-Router

ADR-08 vergleicht zwei Entscheidungen: Wo die SPA liegen soll und wie ihre Routen abgebildet werden. Wir entschieden uns für eine zusätzliche Einstiegsseite `app.html` neben den MPA-Seiten. Ein Umbau von `portfolio.html` hätte die Trennung der Touchpoints verwischt und eine bestehende, getestete MPA-Seite kurz vor der Abgabe verändert. Mit dem separaten Einstieg können beide Varianten unabhängig voneinander aufgerufen und verglichen werden. Zur Auffindbarkeit erhielt die MPA-Navigation einen Link „App (SPA)“. „MPA bleibt erhalten“ bedeutet daher, dass ihre bestehenden Seiten und Funktionen bestehen blieben; die Navigation wurde um diesen Einstieg ergänzt.

Für die Navigation verwendeten wir einen kleinen Router ohne Framework und ohne Build-Schritt. Ein Framework-Router hätte zusätzliche Abhängigkeiten und Einarbeitung bedeutet. Eine History-API-Lösung mit Pfaden wie `/app/portfolios/5` hätte außerdem eine Serverregel erfordert, die direkte Aufrufe und Neuladen auf `app.html` zurückleitet. Das wäre sowohl für den `serve`-Server auf Heroku als auch für die lokale Entwicklung zu konfigurieren gewesen.

Stattdessen verwendet die SPA Hash-Routen wie `app.html#/portfolios/5`. Der Browser sendet den Teil nach `#` nicht als Pfad an den Server. Dadurch funktionieren Direktaufrufe, Lesezeichen sowie Zurück und Vor, ohne dass die Deployment-Konfiguration geändert werden muss. Der Nachteil sind die sichtbaren `#` in den Adressen. Für diese angemeldete Anwendung ohne Suchmaschinen-Anforderung hielten wir das für vertretbar (ADR-08).

## 5.3 Aufbau und Navigation

`src/app.html` lädt das Grundgerüst mit Kopfzeile, Navigation und dem Inhaltsbereich `#route-view`. Das Modul `src/js/spa/router.js` übersetzt den Hash beim ersten Laden und bei `hashchange` in einen Routennamen und gegebenenfalls eine ID. `src/js/spa/app.js` rendert anschließend die passende Ansicht im Inhaltsbereich.

Der Router reagiert auf den aktuellen Hash, löst ihn auf und übergibt die Route an den Renderer (gekürzter Ausschnitt aus `src/js/spa/router.js`):

```js
const route = resolveRoute(hash);
browserWindow.document.title = `${ROUTE_TITLES[route.name]} – LookUpStock (SPA)`;
await render(route);
```

Damit bleibt die Navigation im selben HTML-Dokument; der Renderer ersetzt den Inhalt des Ansichtsbereichs.

| Hash-Route | Ansicht und Aktion | Backend-Aufruf |
|---|---|---|
| `#/portfolios` | Eigene Portfolios auflisten | `GET /portfolios` |
| `#/portfolios/:id` | Portfolio-Positionen ansehen und Simulation starten | `GET /portfolios/{id}`, `POST /simulations` |
| `#/simulations` | Gespeicherte Simulationen auflisten | `GET /simulations` |
| `#/simulations/:id` | Gespeicherte Simulation mit Diagramm öffnen | `GET /simulations/{id}` |
| leerer Hash | Auf `#/portfolios` weiterleiten | – |
| unbekannte Route | Hinweis mit Rückweg zu den Portfolios | – |

Die Navigation besteht aus normalen `a`-Elementen mit Hash-Zielen. Das erhält die erwarteten Browserfunktionen und vermeidet eine eigene Klickbehandlung für jeden Link. Je Ansicht wird der Titel des Browser-Tabs angepasst. Nach dem Rendern setzt der Router den Fokus auf die neue Überschrift. Dadurch ist der Ansichtswechsel auch für Tastaturbedienung erkennbar (NF-09, AT-ADR8-04).

## 5.4 Wiederverwendung statt doppelter Fachlogik

Die SPA nutzt dieselbe REST-Schnittstelle und dieselben geprüften Frontend-Module wie die MPA:

- `api.js` führt die geschützten Anfragen mit dem Bearer-Token aus und liefert API-Fehler als `ApiError`.
- `auth.js` liest die Anmeldung. Ohne gültige Sitzung zeigen die SPA-Ansichten einen Hinweis mit Link zur vorhandenen `login.html`.
- `simulation.js` prüft die Eingaben und fasst das Simulationsergebnis zusammen.
- `chart.js` rendert dasselbe Fächerdiagramm wie in der MPA.
- `saved-simulations.js` formatiert die Angaben zu gespeicherten Simulationen.

Die Eingabeprüfung und die Backend-Aufrufe werden daher nicht für die SPA ein zweites Mal implementiert. Das reduziert Abweichungen zwischen den Touchpoints. Fehler wie eine fehlende Anmeldung, ein nicht gefundenes fremdes Portfolio, ungültige Simulationsparameter oder nicht verfügbare Kursdaten werden über dieselbe API-Schicht verarbeitet. Bei asynchron geladenen Listen verhindert eine Versionsprüfung, dass eine Antwort für eine inzwischen verlassene Ansicht noch gerendert wird (gekürzt aus `src/js/spa/app.js`):

```js
const data = await listPortfolios(session.token);
if (!page.isCurrent()) return;
renderPortfolioList(page.section, data?.portfolios ?? []);
```

Die Versionsprüfung schützt die Leseansichten vor Antworten, die nach einem Routenwechsel eintreffen. Für den asynchronen Simulationsaufruf gibt es dagegen keine entsprechende Prüfung vor der weiteren Verarbeitung des Ergebnisses. Eine verspätete Antwort wird zwar in den vom DOM getrennten Ergebnisbereich der alten Ansicht geschrieben; insbesondere ein 401 kann aber noch den Sitzungszustand beeinflussen. Das ist eine Grenze gegenüber dem in ADR-08 formulierten Ziel, verspätete Antworten nach einem Ansichtswechsel zu verwerfen.

## 5.5 Funktionsvergleich der Touchpoints

Die SPA und MPA greifen auf dieselben Benutzerkonten und REST-Endpunkte zu, bieten aber bewusst nicht denselben Funktionsumfang. Die folgende Gegenüberstellung beschreibt den Stand nach Sprint 5:

| Funktion | MPA | SPA |
|---|---|---|
| Eigene Portfolios ansehen | Ja | Ja |
| Portfolio anlegen, bearbeiten und löschen | Ja, im Portfolio-Editor | Nicht vorgesehen |
| Portfolio simulieren und Fächerdiagramm ansehen | Ja | Ja |
| Simulationen speichern und verwalten | Ja | Gespeicherte Simulationen ansehen; in der SPA nicht anlegen oder löschen |
| Zwei Portfolios oder Anlagen vergleichen | Ja | Nicht vorgesehen |
| Anmeldedaten und geschützte API verwenden | Ja | Ja, über dieselbe Anmeldung und API-Schicht |
| Navigation zwischen Ansichten | Neue HTML-Seite | Hash-Wechsel innerhalb von `app.html` |

Das ist der vom Product Owner festgelegte Ausschnitt „Portfolios ansehen und simulieren, gespeicherte Simulationen ansehen“, keine vollständige Parität zur MPA. Für die Bewertung und Vorführung sollte dieser Unterschied ausdrücklich benannt werden: Die Funktionen der SPA lassen sich mit der MPA vergleichen, aber einige MPA-Funktionen liegen außerhalb des vereinbarten SPA-Umfangs.

## 5.6 Prüfung und Ergebnis

Die Routerlogik ist als eigenes ES-Modul implementiert. `tests/router.test.js` nutzt `node:test` und prüft die Portfolioliste als Standardroute, das Lesen von IDs, die Simulationsrouten sowie unbekannte und ungültige Routen. Damit lässt sich die zentrale Routenauflösung ohne Browser testen.

Die manuellen Abnahmetests prüften darüber hinaus, ob nach dem ersten Laden weitere Ansichten ohne neue Dokumentanfrage erscheinen, ob Browsernavigation, Neuladen und direkte Links funktionieren, ob unbekannte Routen und fehlende Anmeldung behandelt werden und ob der Fokus zur neuen Überschrift wechselt. Laut Sprint-Review 5 bestanden die sechs SPA-Abnahmetestfälle (AT-ADR8-01 bis AT-ADR8-06) alle. Im Netzwerktab wurde beim Routenwechsel keine weitere Anfrage vom Typ `document` erwartet; API-Aufrufe wie `GET /portfolios/{id}` und `POST /simulations` bleiben davon unberührt (Abnahmetestfälle Sprint 5, Abschnitt „SPA-Touchpoint“; Sprint-Review 5, Abschnitte 1, 2 und 5).

Die Abnahme belegt die vereinbarten Funktionsabläufe. Sie ist keine Messung der Ladezeit und kein Nachweis, dass die SPA in allen Funktionen gleichwertig zur MPA ist. Die bewusste Begrenzung des Umfangs und die in Abschnitt 5.4 genannte Behandlung verspäteter Antworten bleiben bei der Interpretation des Ergebnisses zu berücksichtigen.

## 5.7 Rückblick

Die Entscheidung für eine separate `app.html` erwies sich als zweckmäßig: Wir konnten den SPA-Touchpoint ergänzen, ohne die bestehenden MPA-Abläufe zu ersetzen, und beide Varianten im Abnahmetest nebeneinander betrachten. Hash-Routing passte zum statischen Deployment und ließ sich mit einem kleinen, unabhängig testbaren Modul umsetzen. Im Planning wurden für FE #53 sechs Stunden angesetzt. Die Wahl eines kleinen eigenen Routers passte zu diesem engen Zeitrahmen. Aus den vorliegenden Unterlagen geht jedoch nicht hervor, ob die tatsächliche Arbeitszeit dem Schätzwert entsprach (ADR-08; Sprint-Planning-Protokoll Sprint 5).

Der wichtigste Kompromiss war der kleinere Funktionsumfang. Das war durch die Product-Owner-Entscheidung gedeckt, muss aber in der Gegenüberstellung offen sichtbar bleiben. Aus dem Code- und Testabgleich ergibt sich außerdem ein konkreter Verbesserungsbedarf: Ein Abnahmetest sollte während einer laufenden Simulation die Route wechseln und prüfen, dass das verspätete Ergebnis verworfen wird. Die sechs bestehenden SPA-Fälle prüfen diesen Fall nicht gesondert. So ließe sich die in ADR-08 formulierte Anforderung an verspätete Antworten direkt nachweisen.

## Projektinterne Quellen

- [ADR-08: SPA-Touchpoint mit Hash-Routing](../adr/ADR-08.md)
- [Sprint-Planning-Protokoll Sprint 5](../sprints/sprint-5/Sprint-Planning-Protokoll-Sprint-5.docx)
- [Abnahmetestfälle Sprint 5, SPA-Touchpoint](../testkonzept/abnahmetestfaelle-sprint-5.md)
- Sprint-Review 5, insbesondere Abschnitte 1, 2 und 5: `sprints/sprint-5/Sprintreview5.docx`
- Frontend-Implementierung im Repository `Look-Up-Stock-FrontEnd`: MPA-Seiten `src/index.html`, `src/login.html`, `src/portfolio.html`, `src/compare.html`; SPA `src/app.html`, `src/js/spa/router.js`, `src/js/spa/app.js`; Routertest `tests/router.test.js`.