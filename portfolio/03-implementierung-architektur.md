# 3 Implementierung: Architektur, getrennte Dienste, GraphQL

*Docs #175 · Paul Kaufmann (Paul02112004)*

Geplant war LookUpStock als gewöhnliche Spring-Boot-Anwendung mit einem Frontend davor. Nach fünf Sprints besteht das Backend aus zwei Diensten, die sich eine Datenbank teilen und über eine Queue kommunizieren, und GraphQL bietet ungefähr dieselben Funktionen wie REST. Dieses Kapitel beschreibt den Weg dorthin chronologisch, weil fast jede Entscheidung auf einer früheren aufbaut. Grundlage sind ADR-01 bis ADR-07, die Pull Requests im Backend-Repository und das KI-Log. Twelve Data, Queue und Caches im Detail behandelt Kapitel 4, die Oberflächen Kapitel 5.

## 3.1 Ausgangslage: ein Backend mit drei Schichten

In Sprint 1 haben wir uns in ADR-01 für eine Drei-Schichten-Architektur entschieden: Controller, Services und Repositories in einem einzigen Spring-Boot-Backend. Microservices haben wir bewusst verworfen, weil mehrere Deployments und verteilte Fehlerbehandlung für vier Personen und acht Wochen zu viel schienen. Zwei kleine Entscheidungen aus dieser Zeit haben sich später ausgezahlt: Alle Kursdaten liefen über die Schnittstelle `MarketDataProvider`, ursprünglich nur, damit Tests ohne Internet laufen. Und unsere Use-Case-Controller koordinieren nur, statt selbst zu rechnen.

Der eigentliche Fehler lag woanders. Wir haben ADR-01 nie gegen die Vorgaben der Webprogrammierung geprüft, die monolithische Architekturen ausschließen. Aufgefallen ist uns das erst im Sprint-Planning 5, keine zwei Wochen vor der Abgabe. Heute würden wir die Bewertungskriterien beider Module gleich zu Projektbeginn als Anforderungen aufschreiben.

## 3.2 Vorarbeit: Anmeldung im Header (ADR-06)

In ADR-04 hatten wir das Token im JSON-Body übertragen. Weil Browser bei GET keinen Body senden, liefen lesende Abfragen als POST auf eigene Pfade wie `POST /portfolios/list`, was nicht REST-konform ist. Mit ADR-06 steht das Token im Header `Authorization: Bearer <token>`. Statt den Header in jeder Controller-Methode auszulesen, haben wir einen `HandlerMethodArgumentResolver` geschrieben; ein Controller verlangt einfach `@CurrentUser User user` (gekürzt):

```java
@Override
public User resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
        NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {
    String token = extractToken(webRequest.getHeader(HttpHeaders.AUTHORIZATION));
    if (token == null) {
        throw new UnauthorizedException(Messages.NOT_LOGGED_IN); // 401
    }
    return authService.getObject().authenticate(token);          // 401, wenn abgelaufen
}
```

Damit ist jeder neue Endpunkt automatisch geschützt, und der spätere GraphQL-Interceptor nutzt dieselbe Methode `extractToken`. Weil sich Pfade und Übertragung gleichzeitig änderten, mussten Backend und Frontend im selben Moment gemergt werden. Im Rückblick hätten wir uns den Umweg sparen können: Dass Browser bei GET keinen Body senden, wussten wir schon bei ADR-04, haben aber um das Problem herumgebaut statt es zu lösen.

## 3.3 Aufteilung in zwei Dienste (ADR-07)

Vor dem Umbau haben wir in ADR-07 die Optionen abgewogen. Ein Dienst pro Fachbereich hätte sechs Deployments und verteilte Transaktionen bedeutet. Die Simulation als eigener Dienst klang verlockend, braucht aber Portfolios, Kurse und das Speichern, sodass drei Dienste hätten zusammenarbeiten müssen. Gewählt haben wir zwei Dienste: den core-service mit allem Fachlichen und beiden Schnittstellen für das Frontend, und den marktdaten-service, der als einziger Twelve Data kennt.

![Abbildung: Architektur nach Sprint 5 · 2 Dienste, 1 Datenbank, 2 Schemas](abbildungen/architektur-dienste.png)

*Abbildung: Architektur nach Sprint 5 · 2 Dienste, 1 Datenbank, 2 Schemas*

Der Schnitt liegt genau an der Schnittstelle `MarketDataProvider`. Im core-service ersetzt ein `MarktdatenClient` den alten `TwelveDataProvider` und implementiert dieselbe Schnittstelle; für Simulation und Suche änderte sich nichts. Das war der Moment, in dem ein GRASP-Prinzip aus der Vorlesung (Protected Variations) tatsächlich Arbeit gespart hat.

Diesen Aufruf haben wir bewusst synchron gelassen. Die Guidelines erlauben begründete Ausnahmen, und eine Simulation kann erst rechnen, wenn die Kurse da sind; eine Queue hätte nur Wartezeit erzeugt. Der Client wartet höchstens fünf Sekunden, jeder Fehler wird wie früher zu `503`, sodass das Frontend von der Aufteilung nichts merkt. Weil der marktdaten-service als eigene Heroku-App im Internet liegt, lässt ein Filter nur Anfragen mit dem gemeinsamen Schlüssel im Header `X-Internal-Api-Key` durch. Die Datenbank haben wir nicht getrennt: Beide Dienste nutzen eine Datenbank mit zwei Schemas und eigener Flyway-Historie, und der marktdaten-service liest keine Tabelle des Core.

Die synchrone Ausnahme würden wir wieder so wählen, ihr Preis ist aber größer als in ADR-07 beschrieben. Fällt der zweite Dienst aus oder stimmt der Schlüssel nicht, liefern Suche und Simulation `503`, obwohl der Core fehlerfrei ist. Wie viel Zeit der zusätzliche Aufruf kostet, haben wir nicht gemessen.

## 3.4 Umsetzung in kleinen Schritten

Den Umbau haben wir in einzeln mergebare Schritte zerlegt. Zuerst kam ein reiner Verschiebe-PR: Der Code wanderte mit `git mv` nach `core-service/`, 140 von 141 Dateien waren reine Umbenennungen. Den marktdaten-service haben wir zuerst vom Feature-Branch auf Heroku deployt und erst gemergt, als der Health-Check „UP“ meldete; sonst hätten Suche und Simulation online sofort mit 503 geantwortet.

Die größten Probleme lagen nicht im Code. Beim Umzug fiel uns auf, dass in `.env.example` seit Sprint 1 ein echter Twelve-Data-Key stand. Wir haben ihn aus der Datei entfernt, er blieb aber in der Git-Historie, bis er in der Abnahme ersetzt wurde. Auf Heroku kosteten uns Buildpack, Datenbank-Anbindung und ein falsch benannter App-Name Zeit, auch weil nur ein Teammitglied Zugang hatte. Den Code für die Dienste und GraphQL haben wir mit KI-Unterstützung erstellt; die KI konnte ihn nicht selbst kompilieren, deshalb liefen vor jedem Push lokal alle Tests, am Ende 302 im Core und 33 im marktdaten-service.

## 3.5 Asynchronität zwischen den Diensten

Erst beim Abgleich mit den Guidelines haben wir gemerkt, dass die Queue nur Teile des Core verband: Nutzungsereignisse stellt der Core ein und verarbeitet sie selbst, zwischen den Diensten lief alles synchron. Die Lösung stand schon in ADR-07: ein zweites Thema `kursdaten.vorladen`. Legt jemand ein Portfolio an oder ändert es, stellt der Core Symbol und MIC-Code der Anlagen in die Queue. Der marktdaten-service holt alle 15 Sekunden höchstens drei Nachrichten ab und lädt fehlende Kurse in seinen Cache. Die Nachricht enthält bewusst keine IDs aus dem Core:

```json
{ "instruments": [ { "symbol": "URTH", "micCode": "ARCX" }, { "symbol": "IEMG", "micCode": "ARCX" } ] }
```

Dank der Schnittstelle `MessagePublisher` aus der Queue von Kapitel 4 war ein neues Thema nur ein neuer Absender und ein neuer Verbraucher. Die Lehre: Eine Vorgabe ist nicht erfüllt, nur weil die passende Technik im Projekt existiert. Hätten wir zu jeder Vorgabe die konkrete Interaktion notiert, die sie nachweist, wäre die Lücke schon beim Planen aufgefallen.

## 3.6 GraphQL mit vergleichbarem Umfang

Seit Sprint 4 war GraphQL nur lesend. In Sprint 5 haben wir sechs Mutations ergänzt, je eine pro schreibendem REST-Endpunkt: Portfolio anlegen, ändern und löschen, simulieren mit optionalem Speichern, gespeicherte Simulation löschen und vergleichen. „Vergleichbar“ haben wir wörtlich genommen: Der `MutationController` bindet die Eingaben an dieselben Klassen wie REST und ruft dieselben Services auf (Stand nach der Behebung von B8):

```java
@MutationMapping
public SimulationResponse simulate(@Argument SimulationRequest input,
        @ContextValue(name = AuthTokenInterceptor.TOKEN_KEY, required = false) String token) {
    User user = currentUser(token);                // UNAUTHORIZED wie 401 bei REST
    String name = input.validatedName();           // Name vor dem Rechnen prüfen
    SimulationParameters parameters = input.parameters() == null
            ? null : input.parameters().toSimulationParameters();
    SimulationResult result = input.shouldSave()   // beim Speichern kein zweites SIMULATION_RUN (B8)
            ? simulationService.simulateForSave(input.portfolioId(), parameters, user)
            : simulationService.simulate(input.portfolioId(), parameters, user);
    if (!input.shouldSave()) {
        return SimulationResponse.from(result);
    }
    SavedSimulation saved = savedSimulationService.save(result, name, user);
    return SimulationResponse.from(saved, savedSimulationService.readResult(saved));
}
```

Dass beide Schnittstellen dieselben Services nutzen, zeigte sich auch beim Fehler B8 aus der Abnahme: Die doppelte Zählung gespeicherter Simulationen steckte in REST und GraphQL zugleich, die Behebung im Service gilt für beide.

Drei Details haben wir bewusst entschieden. Die Eingabefelder sind im Schema optional, damit unsere eigenen Prüfregeln mit deutschen Meldungen greifen statt der englischen Standardmeldung von GraphQL. Der Seed ist als `Float` typisiert, weil `Int` nur 32 Bit kann; das ist bis 2^53 exakt, darüber würde stillschweigend gerundet. Praktisch tritt das nicht auf, weil das System nur Seeds unter einer Million erzeugt, sauber wäre aber ein eigener Datentyp. Und weil GraphQL auch bei Fehlern mit 200 antwortet, stehen Fehlercode und Einzelfehler im Feld `extensions`. Registrierung, Anmeldung, Gastmodus und Auswertungen gibt es nur über REST; das steht ausdrücklich im API-Design.

## 3.7 Fazit

Die Architektur hat sich in der letzten Woche grundlegend verändert, ohne dass wir Kernlogik oder Tests umbauen mussten. Das lag an früheren Entscheidungen: Schnittstellen wie `MarketDataProvider` und `MessagePublisher`, Controller, die nur koordinieren, und ADRs mit Alternativen vor dem Code. Diese würden wir wieder so treffen, ebenso kleine, einzeln gemergte Umbauschritte. Anders machen würden wir den Anfang: Bewertungskriterien von Beginn an als Anforderungen, Schlüssel nie in Beispieldateien, Branch-Schutz ab Sprint 1 und Heroku-Zugang für alle. Offen bleibt die gemeinsame Datenbank, eine bewusste Kopplung, die bei weiterem Wachstum als Nächstes getrennt würde.
