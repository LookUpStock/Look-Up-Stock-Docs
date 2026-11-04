# 4 Implementierung: Integration von Twelve Data, Queue und Caches

*Docs #176 · Peter Orban (Pixxels02), Mert Eminel (KujoBaraka)*

Dieses Kapitel beschreibt, wie LookUpStock den Fremdservice Twelve Data einbindet, mit dessen Grenzen umgeht und die Infrastruktur durch Caches und eine Queue entlastet – mit Fokus auf unsere Entscheidungen und die Fehler, aus denen sie entstanden. Den Schnitt in zwei Dienste beschreibt Kapitel 3.

## 4.1 Twelve Data als Fremdservice

### 4.1.1 Ausgangslage und Auswahl

Für die Monte-Carlo-Simulation braucht LookUpStock historische Monatskurse jeder Anlage, für den Portfolio-Editor eine Suche nach Aktien und ETFs. Nach einem Anbietervergleich in Sprint 1 (BE #16) hat der Product Owner Twelve Data festgelegt (E7). Genutzt werden die Symbolsuche (`symbol_search`) und die Kursreihe (`time_series` mit `interval=1month`). Über einen einfachen GET-Aufruf hinaus werden Antworten in eigene Datentypen übersetzt, Fehlercodes auf verständliche Meldungen abgebildet und Ergebnisse zwischengespeichert und vorgeladen.

### 4.1.2 Kapselung hinter einer Schnittstelle

Von Anfang an greift die Fachlogik nur über die Schnittstelle `MarketDataProvider` auf Kursdaten zu (NF-12). So laufen alle Unit-Tests ohne Internet und API-Key mit der Attrappe `MockMarketDataProvider` (BE #53, BE #89), und ein Anbieterwechsel betrifft nur eine Klasse. Der API-Key liegt ausschließlich als Umgebungsvariable auf dem Server (NF-06).

Das zahlte sich in Sprint 5 aus: Als die Anbindung mit ADR-07 in den marktdaten-service wanderte, implementierte im Core einfach der `MarktdatenClient` dieselbe Schnittstelle; Kernlogik und Tests blieben unverändert (Kapitel 3).

### 4.1.3 Grenzen des Anbieters und was wir daraus gelernt haben

Die meisten Probleme zeigten sich nicht in den Unit-Tests, sondern erst in den Abnahmetests gegen den echten Dienst:

- **Tarif statt Fehler (F3, Sprint 2):** Eine Simulation mit einem Xetra-ETF meldete „weniger als 3 Jahre Kursdaten“, mit einer US-Aktie funktionierte sie. Der kostenlose Tarif liefert keine Xetra-Kurse, und unsere Anbindung unterschied die Fehlerantwort nicht von einer kurzen Kursreihe. Twelve Data meldet solche Fehler meist mit HTTP 200 und einem Fehlercode im JSON-Body. Wir ordnen diese Codes seitdem ausdrücklich ein (gekürzt):

  ```java
  // marktdaten-service, TwelveDataError: Einordnung der Fehlercodes von Twelve Data
  return switch (code) {
      case 401 -> INVALID_KEY;      // API-Key fehlt oder ist ungültig
      case 403 -> PLAN_RESTRICTED;  // Börse nicht im Tarif, z. B. Xetra
      case 429 -> RATE_LIMIT;       // Kontingent pro Minute bzw. Tag ausgeschöpft
      case 400, 404 -> NOT_FOUND;   // Symbol oder Börse unbekannt
      default -> OTHER;
  };

  // TwelveDataProvider: dauerhaft keine Kurse → leere Liste, alles andere → 503
  if (error == TwelveDataError.NOT_FOUND || error == TwelveDataError.PLAN_RESTRICTED) {
      return List.of();
  }
  throw unavailable(operation, code, message);
  ```

  Der `ParameterEstimator` im core-service unterscheidet dann eine leere Kursreihe (Hinweis, die Anlage an einer anderen Börse auszuwählen) von einer zu kurzen (weniger als 36 Monatswerte), beides als `422`. Beispielportfolios und Testdaten nutzen seitdem Anlagen an US-Börsen (E45).
- **Reproduzierbarkeit (F4, Sprint 2):** Derselbe Seed lieferte beim ersten Lauf ein anderes Ergebnis als danach. Der erste Lauf rechnete mit ungerundeten Kursen vom Anbieter, die späteren mit den auf sechs Nachkommastellen gerundeten aus dem Cache. Seit dem Fix werden frische Kurse ebenso gerundet (NF-08, siehe 4.2).
- **Datenqualität (B5, B6, Sprint 3 und 4):** Der laufende, noch nicht abgeschlossene Monat wird verworfen (E22). Die Suche nach „ETHA“ endete mit `500`, weil die Spalte für den Börsencode zu kurz war (Migration V8).
- **Anfragelimit:** Der kostenlose Tarif erlaubt nur wenige Anfragen pro Minute; jede vermiedene Anfrage hält die Anwendung nutzbar. Das begründet die Caches (4.2) und die Drosselung des Vorladens (4.3).
- **Bereinigte Kurse (beim Schreiben dieses Kapitels aufgefallen):** Das Simulationsmodell geht von um Splits und Dividenden bereinigten Kursen aus (E20). Unser Aufruf setzt den Parameter `adjust` aber nicht. Laut Twelve Data sind Monatskurse um Splits bereinigt; ob Dividenden eingerechnet werden, bleibt offen (Twelve Data 2026). Ob E20 im Betrieb erfüllt ist, haben wir nicht geprüft.

**Erkenntnis:** Eine Schnittstelle mit Attrappe macht die Fachlogik testbar, schützt aber nicht vor falschen Annahmen über den echten Dienst. F3 hätten wir schon in Sprint 1 bemerken können: Das API-Design verwendete von Beginn an Xetra-ETFs (EUNL, IS3N), und ein einziger Abruf mit dem kostenlosen Schlüssel hätte die Tarifgrenze gezeigt. Der Anbietervergleich prüfte nur die grundsätzliche Verfügbarkeit, nicht unseren Tarif. Künftig würden wir zuerst Probeabrufe gegen den echten Dienst festhalten: eigene Beispielanlagen, je eine Börse, ein unbekanntes Symbol, Anfragelimit und Kursbereinigung.

## 4.2 Caches

LookUpStock nutzt zwei Caches mit unterschiedlicher Technik, weil die Daten unterschiedlich lange gebraucht werden. Beide liegen seit ADR-07 im marktdaten-service.

| | Such-Cache | Kurs-Cache |
|---|---|---|
| Inhalt | Treffer je Suchbegriff | Monatskurse je Anlage (Symbol, Börse, Datum) |
| Speicherort | Arbeitsspeicher (`ConcurrentHashMap`) | PostgreSQL, Schema `marktdaten`, Tabelle `price_cache` |
| Gültigkeit | 24 Stunden | 24 Stunden; genau 24 Stunden alt gilt als veraltet |
| Bei Ausfall von Twelve Data | Treffer jünger als 24 Stunden | Kurse jünger als 24 Stunden, sonst `503` |
| Anforderung | NF-02 (Antwortzeit der Suche) | NF-01 (Simulationsdauer), NF-07 (Ausfallsicherheit) |

**Such-Cache.** In Sprint 2 fehlte er ganz (F5, hohe Priorität). Wir haben ihn bewusst im Arbeitsspeicher umgesetzt, weil Suchergebnisse schnell neu zu beschaffen sind und ein Verlust beim Neustart unkritisch ist; `ConcurrentHashMap`, weil mehrere Anfragen gleichzeitig suchen. In der Abnahme Sprint 3 antworteten 20 Suchen aus dem Cache in höchstens 58 ms (AT-02-08); NF-02 verlangt höchstens 1 s.

**Kurs-Cache.** Kurse werden für jede Simulation gebraucht und überstehen Neustarts nur in der Datenbank. Der Ablauf im `PriceService` (gekürzt):

```java
Optional<CachedPrices> cached = priceCache.load(symbol, micCode);
if (cached.isPresent() && isFresh(cached.get().fetchedAt(), now)) {
    return cached.get().prices();                       // kein Aufruf von Twelve Data
}
List<PricePoint> fresh = marketDataProvider.getHistoricalPrices(symbol, micCode, "1month", outputSize);
List<PricePoint> sorted = fresh.stream()
        .map(p -> new PricePoint(p.date(), p.close().setScale(6, RoundingMode.HALF_UP)))  // F4, NF-08
        .sorted(Comparator.comparing(PricePoint::date))
        .toList();
priceCache.replace(symbol, micCode, sorted, now);      // Fehler beim Speichern nur protokolliert
return sorted;
```

Die Zeile mit `setScale(6, …)` ist die Lehre aus F4: Frische Kurse werden genauso gerundet wie die in `NUMERIC(19,6)` gespeicherten, sonst hängt das Ergebnis eines Seeds von der Herkunft der Kurse ab. Mit Kursen im Cache erfüllte die Simulation NF-01 in der Abnahme Sprint 3 in 20 von 20 Läufen (1.497 bis 1.602 ms, gefordert unter drei Sekunden).

Objektiv betrachtet bringt der Cache bei einem Ausfall weniger, als wir anfangs dachten. NF-07 verlangt, Kurse jünger als 24 Stunden weiterzuverwenden – genau diese werden aber ohnehin ohne Aufruf von Twelve Data geliefert. Ein eigenes Ausfallverhalten gibt es damit nicht: Ältere Kurse werden nicht verwendet, obwohl sie für eine Simulation über 20 Jahre kaum einen Unterschied machen würden. Eine längere Frist für den Ausfallfall, etwa einige Tage mit Hinweis an den Nutzer, hätten wir schon bei der Formulierung von NF-07 abwägen sollen.

Seit ADR-07 wird der Kurs-Cache über Symbol und Börsencode statt über die Anlagen-ID des Core angesprochen; die alte Tabelle `price_history` entfiel mit V11.

**Erkenntnis:** Ein Cache ist nicht nur eine Optimierung. Er verändert Ergebnisse (F4) und muss wie Fachlogik getestet werden; die Grenzfälle „genau 24 Stunden alt“ und „Cache veraltet, Anbieter nicht erreichbar“ sind deshalb mit vorgegebener Uhr getestet (`PriceServiceTest`).

## 4.3 Queue und asynchrone Kommunikation

### 4.3.1 Anforderung und Entscheidung

Die Portfolio-Guidelines verlangen asynchrone Kommunikation zwischen den Komponenten, begründete Ausnahmen sind zugelassen. Gleichzeitig kam mit CR-01 die Erfassung von Nutzungsereignissen für ein Dashboard der Geschäftsführung (US-11), die eine Simulation weder verlangsamen noch scheitern lassen darf.

ADR-07 hält drei Varianten fest: einen Message Broker als Heroku-Add-on, eine Queue-Tabelle in der vorhandenen PostgreSQL-Datenbank und Spring-Events. Spring-Events schieden aus, weil sie innerhalb eines Dienstes bleiben und Nachrichten bei einem Neustart verloren gehen; gegen einen Broker sprachen Kosten, zusätzliche Zugangsdaten und die Einarbeitung in einer Woche. Wir haben uns für die **Queue-Tabelle** entschieden: kostenlos, neustartfest und mit der Test-Datenbank der CI prüfbar. Die Verzögerung durch periodisches Abfragen ist für Nutzungszahlen unkritisch.

Die Kursabfrage vom Core zum marktdaten-service bleibt als begründete Ausnahme **synchron**, weil die Simulation erst mit den Kursen rechnen kann (ADR-07, Kapitel 3).

### 4.3.2 Umsetzung

Die Queue ist eine Tabelle `queue_message` mit Thema, Inhalt als JSON, Zeitpunkt, einem Zähler für Fehlversuche und einer Markierung für endgültig fehlgeschlagene Nachrichten (V10, BE #114). Sie wird über zwei Themen genutzt:

- **`nutzung.ereignis`:** Nach einer erfolgreichen Aktion, etwa einer Anmeldung oder Simulation, stellt der `UsageEventRecorder` ein Ereignis ein. Ein Verbraucher holt alle fünf Sekunden höchstens 50 Nachrichten ab, überträgt sie in die Tabelle der Nutzungsereignisse und löscht sie.
- **`kursdaten.vorladen`:** Wird ein Portfolio angelegt oder geändert, lädt der marktdaten-service die Kurse der Anlagen im Hintergrund in seinen Cache (alle 15 Sekunden höchstens drei Nachrichten, wegen des Anfragelimits; Hintergrund in Kapitel 3).

Drei Details haben sich als wichtig erwiesen:

1. **Keine doppelte Verarbeitung.** Die Verbraucher holen Nachrichten mit `SELECT … FOR UPDATE SKIP LOCKED` (PostgreSQL Documentation, „The Locking Clause“). Laufen mehrere Instanzen, überspringt jede die Zeilen, die eine andere gerade bearbeitet:

   ```java
   @Query(value = """
           SELECT * FROM queue_message
           WHERE topic = :topic AND failed = false
           ORDER BY id
           LIMIT :limit
           FOR UPDATE SKIP LOCKED
           """, nativeQuery = true)
   List<QueueMessage> lockNextBatch(@Param("topic") String topic, @Param("limit") int limit);
   ```

2. **Ein Ausfall der Queue darf die Aktion nicht treffen.** Das Einstellen läuft in einer eigenen Transaktion, Fehler werden nur protokolliert:

   ```java
   @Transactional(propagation = Propagation.REQUIRES_NEW)
   public void publish(String topic, Object payload) {
       String json = objectMapper.writeValueAsString(payload);
       queueMessageRepository.save(new QueueMessage(topic, json, Instant.now(clock)));
   }
   ```

   Ohne `REQUIRES_NEW` hätte ein Datenbankfehler beim Einstellen auch die Transaktion der eigentlichen Aktion als fehlgeschlagen markiert und etwa das Anlegen eines Portfolios zurückgerollt (Spring Framework Documentation, „Transaction Propagation“).

3. **Zeitpunkt der Aktion statt der Verarbeitung.** Sonst würde eine Anmeldung um 23:59:58 am Monatsende im Folgemonat gezählt. Die Monatsgrenzen rechnet die Auswertung nach deutscher Zeit (E38).

Hinter der Schnittstelle `MessagePublisher` ist die Tabelle austauschbar; ein Wechsel auf einen Broker ändert keine Aufrufer (Protected Variations). Erfasst wird in den Services statt in den Controllern, damit eine Aktion über REST und GraphQL gleich gezählt wird.

### 4.3.3 Fehler und Erkenntnisse

- **Die Reihenfolge der Schritte war entscheidend.** Erst Ereignistabelle (V9), dann Queue (V10), zuletzt die acht Erfassungsstellen – sonst hätten wir alle acht nachträglich umbauen müssen.
- **Kleine Pull Requests machten Fehler sichtbar.** Der PR mit der Queue (BE #120) war zunächst rot, weil `UsageEventConsumer` ohne Dateiendung `.java` übernommen worden war. Der PR mit V9 (BE #119) versprach das Feld `role` in der Login-Antwort, das erst BE #121 nachlieferte. Beides fiel auf, weil jeder PR nur einen Schritt enthielt; seitdem lesen wir den Diff vor dem Merge gegen die eigene Beschreibung.
- **Grenzen unserer Tests.** Monatsgrenzen sind mit vorgegebener Uhr getestet (`UsageReportServiceTest`). Unsere Unit-Tests laufen aber ohne Datenbank und zeigen nur, dass ein Fehler beim Einstellen nicht weitergeworfen wird (`recordDoesNotThrowWhenQueueFails`, `failingQueueDoesNotBreakTheAction`). Dass `REQUIRES_NEW` die Aktion tatsächlich schützt, haben wir aus der Dokumentation abgeleitet, nicht gegen PostgreSQL nachgewiesen.
- **KI-Unterstützung braucht eine Prüfinstanz.** Teile der Queue wurden mit KI-Unterstützung entworfen (KI-Log). Die eigentliche Prüfinstanz waren Unit-Tests und CI, weil der Entwurf erst dort ausgeführt wurde.
- **Was offen bleibt.** Nach drei Versuchen fehlgeschlagene Nachrichten bleiben unbemerkt in der Tabelle. Die Verarbeitungsdauer im Betrieb und den Nutzen des Vorladens für die erste Simulation haben wir nicht gemessen. Beides würden wir in einem weiteren Sprint sichtbar machen.

## 4.4 Zusammenfassung

| Anforderung der Guidelines | Umsetzung in LookUpStock | Was wir anders machen würden |
|---|---|---|
| Integration eines Fremdservice über mehr als einen GET-Aufruf | Twelve Data: Suche, Monatskurse, Einordnung der Fehlercodes, Abbildung auf eigene Meldungen; gekapselt im marktdaten-service | Annahmen über Tarif, Börsen und Bereinigung zu Projektbeginn mit Probeabrufen prüfen |
| Asynchrone Kommunikation zwischen Komponenten | Queue-Tabelle mit den Themen `nutzung.ereignis` und `kursdaten.vorladen`; synchrone Kursabfrage als begründete Ausnahme (ADR-07) | Fehlgeschlagene Nachrichten sichtbar machen, Verarbeitungsdauer messen |
| Infrastruktur-Optimierung | Such-Cache (`ConcurrentHashMap` im Arbeitsspeicher), Kurs-Cache (PostgreSQL), Vorladen der Kurse über die Queue | Längere Frist für Kurse bei einem Ausfall des Anbieters |

Quellen im Projekt: ADR-07, Simulationsmodell, Kurzformat-Tabellen (E7, E20, E22, E38, E45), NFRs (NF-01, NF-02, NF-06, NF-07, NF-08, NF-12), Abnahmeprotokolle Sprint 2 bis 4, Change Request CR-01, Pull Requests BE #119 bis #121, KI-Log.
