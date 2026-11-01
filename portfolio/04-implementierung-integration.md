# 4 Implementierung: Integration von Twelve Data, Queue und Caches

*Docs #176 · Peter Orban (Pixxels02), Mert Eminel (KujoBaraka)*

Dieses Kapitel beschreibt, wie LookUpStock den Fremdservice Twelve Data einbindet, wie die Anwendung mit dessen Grenzen umgeht und wie Caches und eine Queue die Infrastruktur entlasten. Im Mittelpunkt stehen die Entscheidungen, die wir dabei getroffen haben, und die Fehler, aus denen sie entstanden sind. Den Schnitt in zwei Dienste und den Aufruf vom core-service zum marktdaten-service beschreibt Kapitel 3; hier geht es um das, was innerhalb und zwischen diesen Diensten mit den Daten von Twelve Data passiert.

## 4.1 Twelve Data als Fremdservice

### 4.1.1 Ausgangslage und Auswahl

Für die Monte-Carlo-Simulation braucht LookUpStock historische Monatskurse jeder Anlage, für den Portfolio-Editor eine Suche nach Aktien und ETFs. Nach einem Vergleich mehrerer Anbieter in Sprint 1 (BE #16) hat der Product Owner Twelve Data festgelegt (E7). Genutzt werden zwei Endpunkte: die Symbolsuche (`symbol_search`) und die Kursreihe (`time_series` mit `interval=1month`).

Die Integration geht damit über einen einzelnen GET-Aufruf hinaus: Antworten werden in eigene Datentypen übersetzt, Fehlercodes des Anbieters eingeordnet und auf verständliche Meldungen abgebildet, Ergebnisse zwischengespeichert und im Hintergrund vorgeladen.

### 4.1.2 Kapselung hinter einer Schnittstelle

Von Anfang an greift die Fachlogik nur über die Schnittstelle `MarketDataProvider` auf Kursdaten zu (NF-12). Das hatte zwei Gründe: Alle Unit-Tests sollen ohne Internet und ohne API-Key laufen, wofür es die Attrappe `MockMarketDataProvider` mit festen Testdaten gibt (BE #53, BE #89), und ein Anbieterwechsel soll nur eine Klasse betreffen. Der API-Key liegt ausschließlich als Umgebungsvariable auf dem Server (NF-06).

Diese frühe Entscheidung hat sich in Sprint 5 ausgezahlt: Mit ADR-07 wanderte die gesamte Anbindung an Twelve Data in den marktdaten-service, und im core-service implementiert seitdem der `MarktdatenClient` dieselbe Schnittstelle (Kapitel 3). Kernlogik und Tests des Core mussten dafür nicht geändert werden.

### 4.1.3 Grenzen des Anbieters und was wir daraus gelernt haben

Die meisten Probleme mit dem Fremdservice zeigten sich nicht in den Unit-Tests, sondern erst in den Abnahmetests gegen den echten Dienst:

- **Tarif statt Fehler (F3, Sprint 2):** Eine Simulation mit einem Xetra-ETF meldete „weniger als 3 Jahre Kursdaten“, mit einer US-Aktie funktionierte sie. Ursache war, dass der kostenlose Tarif keine Xetra-Kurse liefert und unsere Anbindung die Fehlerantwort nicht von einer kurzen Kursreihe unterschied. Twelve Data meldet solche Fehler meist mit HTTP 200 und einem eigenen Fehlercode im JSON-Body. Wir ordnen diese Codes seitdem ausdrücklich ein (gekürzt):

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

  Im core-service unterscheidet der `ParameterEstimator` dann eine leere Kursreihe („Für folgende Anlagen liefert der Kursdatenanbieter keine Kurse … Bitte die Anlage an einer anderen Börse auswählen.“) von einer zu kurzen (weniger als 36 Monatswerte), beides als `422`. Die Beispielportfolios und die manuellen Testdaten nutzen seitdem Anlagen an US-Börsen (PO-Entscheidung vom 18.10.2026, E45).
- **Reproduzierbarkeit (F4, Sprint 2):** Derselbe Seed lieferte beim ersten Lauf ein anderes Ergebnis als bei den folgenden. Der erste Lauf rechnete mit Kursen in voller Genauigkeit direkt vom Anbieter, die späteren mit den auf sechs Nachkommastellen gerundeten Werten aus dem Cache. Seit dem Fix werden frisch geladene Kurse vor der Verwendung ebenso gerundet (NF-08, siehe 4.2).
- **Datenqualität (B5, B6, Sprint 3 und 4):** Twelve Data liefert den laufenden, noch nicht abgeschlossenen Monat mit; er wird nun verworfen (E22). Die Suche nach dem Ticker „ETHA“ endete mit `500`, weil die Spalte für den Börsencode zu kurz war (Migration V8).
- **Anfragelimit:** Der kostenlose Tarif erlaubt nur wenige Anfragen pro Minute. Jede vermiedene Anfrage ist daher nicht nur schneller, sondern hält die Anwendung überhaupt nutzbar. Das ist der wichtigste Grund für die Caches in Abschnitt 4.2 und dafür, dass das Vorladen über die Queue höchstens drei Nachrichten je Durchlauf verarbeitet (4.3).
- **Bereinigte Kurse (beim Schreiben dieses Kapitels aufgefallen):** Das Simulationsmodell geht von der Gesamtrendite aus, also von Kursen, die um Splits und Dividenden bereinigt sind (E20). Unser Aufruf von `time_series` setzt den Parameter `adjust` aber nicht, es gilt die Voreinstellung des Anbieters. Laut Twelve Data sind Tages-, Wochen- und Monatskurse um Splits bereinigt; ob auch Dividenden eingerechnet werden, lässt die Beschreibung offen (Twelve Data 2026). Ob E20 im Betrieb erfüllt ist, haben wir nicht geprüft.

**Erkenntnis:** Eine Schnittstelle mit Attrappe macht die Fachlogik testbar, schützt aber nicht vor falschen Annahmen über den echten Dienst. F3 hätten wir schon in Sprint 1 bemerken können: Die Beispiele im API-Design verwendeten von Beginn an Xetra-ETFs (EUNL, IS3N), und ein einziger Abruf genau dieser Anlagen mit dem kostenlosen Schlüssel hätte die Tarifgrenze gezeigt, bevor Testdaten, Testfälle und Beispiele darauf aufbauten. Der Anbietervergleich in BE #16 prüfte, ob die Daten grundsätzlich verfügbar sind, nicht, ob sie in unserem Tarif und in der von uns angenommenen Form ankommen. Bei der nächsten API-Anbindung würden wir zuerst einen kleinen Katalog von Probeabrufen gegen den echten Dienst ausführen und das Ergebnis festhalten: die Anlagen aus den eigenen Beispielen, je eine Anlage an jeder benötigten Börse, ein unbekanntes Symbol, das Verhalten beim Anfragelimit und die Bereinigung der Kurse.

## 4.2 Caches

LookUpStock nutzt zwei Caches mit unterschiedlicher Technik, weil die Daten unterschiedlich lange gebraucht werden. Beide liegen seit ADR-07 im marktdaten-service.

| | Such-Cache | Kurs-Cache |
|---|---|---|
| Inhalt | Treffer je Suchbegriff | Monatskurse je Anlage (Symbol, Börse, Datum) |
| Speicherort | Arbeitsspeicher (`ConcurrentHashMap`) | PostgreSQL, Schema `marktdaten`, Tabelle `price_cache` |
| Gültigkeit | 24 Stunden | 24 Stunden; genau 24 Stunden alt gilt als veraltet |
| Bei Ausfall von Twelve Data | Treffer jünger als 24 Stunden | Kurse jünger als 24 Stunden, sonst `503` |
| Anforderung | NF-02 (Antwortzeit der Suche) | NF-01 (Simulationsdauer), NF-07 (Ausfallsicherheit) |

**Such-Cache.** In Sprint 2 fehlte er ganz; die Abnahme hat das als F5 mit hoher Priorität vermerkt. Wir haben ihn bewusst im Arbeitsspeicher umgesetzt: Suchergebnisse sind schnell neu zu beschaffen, ein Verlust beim Neustart ist unkritisch, und ein Datenbankzugriff würde die gewünschte Antwortzeit unnötig belasten. `ConcurrentHashMap` war nötig, weil mehrere Anfragen gleichzeitig suchen können. Der Nutzen ist messbar: In der Abnahme Sprint 3 antworteten 20 Suchen aus dem Cache in höchstens 58 ms (AT-02-08); NF-02 verlangt höchstens 1 s.

**Kurs-Cache.** Kurse werden für jede Simulation gebraucht und überstehen Neustarts nur in der Datenbank. Der Ablauf im `PriceService` ist kurz (gekürzt):

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

Die Zeile mit `setScale(6, …)` ist die Lehre aus F4: Frische Kurse werden genauso gerundet wie die in der Spalte `NUMERIC(19,6)` gespeicherten, sonst liefert derselbe Seed je nach Herkunft der Kurse unterschiedliche Ergebnisse. Mit Kursen im Cache erfüllte die Simulation NF-01 in der Abnahme Sprint 3 mit 20 von 20 Läufen unter drei Sekunden (1.497 bis 1.602 ms). Liegen keine Kurse jünger als 24 Stunden vor und ist Twelve Data nicht erreichbar, antwortet das Backend mit `503` und der Meldung „Kursdaten sind gerade nicht verfügbar.“

Objektiv betrachtet bringt der Cache bei einem Ausfall weniger, als wir anfangs dachten. NF-07 verlangt, Kurse jünger als 24 Stunden weiterzuverwenden. Genau diese Kurse werden aber ohnehin ohne Aufruf von Twelve Data aus dem Cache geliefert. Ein eigenes Ausfallverhalten gibt es damit nicht: Ältere Kurse werden bei einem Ausfall nicht verwendet, auch wenn sie für eine Simulation über 20 Jahre kaum einen Unterschied machen würden. Eine längere Frist für den Ausfallfall, etwa einige Tage mit einem Hinweis an den Nutzer, hätte die Verfügbarkeit deutlich erhöht. Diese Abwägung hätten wir bei der Formulierung von NF-07 treffen sollen.

Beim Umzug in den marktdaten-service (ADR-07) wird der Kurs-Cache über Symbol und Börsencode statt über die Datenbank-ID der Anlage angesprochen, weil der neue Dienst die Tabellen des Core nicht kennen soll. Die alte Tabelle `price_history` wurde mit V11 entfernt; in Kauf genommen haben wir, dass der Cache nach dem Umzug einmal neu aufgebaut wird.

**Erkenntnis:** Ein Cache ist nicht nur eine Optimierung. Er verändert Ergebnisse (F4) und muss deshalb genauso getestet werden wie die Fachlogik; die Grenzfälle „genau 24 Stunden alt“ und „Cache veraltet und Anbieter nicht erreichbar“ sind deshalb mit vorgegebener Uhr getestet (`PriceServiceTest`).

## 4.3 Queue und asynchrone Kommunikation

### 4.3.1 Anforderung und Entscheidung

Die Portfolio-Guidelines verlangen asynchrone Kommunikation zwischen den Komponenten; begründete Ausnahmen sind zugelassen. Gleichzeitig kam mit dem Change Request CR-01 die Erfassung von Nutzungsereignissen für ein Dashboard der Geschäftsführung (US-11). Diese Erfassung darf eine Simulation weder verlangsamen noch scheitern lassen.

ADR-07 hält drei Varianten fest: einen Message Broker als Heroku-Add-on, eine Queue-Tabelle in der vorhandenen PostgreSQL-Datenbank und Spring-Events innerhalb eines Dienstes. Spring-Events schieden aus, weil sie keine Kommunikation zwischen Komponenten sind und Nachrichten bei einem Neustart verloren gehen. Gegen einen Broker sprachen Kosten, zusätzliche Zugangsdaten und die Einarbeitung in einer Woche. Wir haben uns für die **Queue-Tabelle** entschieden: Sie kostet nichts, übersteht Neustarts und lässt sich mit der vorhandenen Test-Datenbank der CI prüfen. Den Nachteil, dass Verbraucher die Tabelle in festen Abständen abfragen und Nachrichten erst nach einigen Sekunden verarbeitet werden, nehmen wir für Nutzungszahlen bewusst in Kauf.

Die Kursabfrage vom Core zum marktdaten-service bleibt dagegen **synchron**. Das ist die begründete Ausnahme: Die Simulation kann erst rechnen, wenn die Kurse vorliegen (ADR-07, Kapitel 3).

### 4.3.2 Umsetzung

Die Queue ist eine Tabelle `queue_message` mit Thema, Inhalt als JSON, Zeitpunkt, einem Zähler für Fehlversuche und einer Markierung für endgültig fehlgeschlagene Nachrichten (V10, BE #114). Sie wird über zwei Themen genutzt:

- **`nutzung.ereignis`:** Nach einer erfolgreichen Aktion, etwa einer Anmeldung oder Simulation, stellt der `UsageEventRecorder` ein Ereignis mit Art und Zeitpunkt ein. Ein Verbraucher holt alle fünf Sekunden höchstens 50 Nachrichten ab, überträgt sie in die Tabelle der Nutzungsereignisse und löscht sie.
- **`kursdaten.vorladen`:** Wird ein Portfolio angelegt oder geändert, lädt der marktdaten-service die Kurse der enthaltenen Anlagen im Hintergrund in seinen Cache (alle 15 Sekunden höchstens drei Nachrichten, wegen des Anfragelimits). Wie es zu diesem zweiten Thema kam, beschreibt Kapitel 3.

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

   Ohne `REQUIRES_NEW` liefe das Einstellen in der Transaktion der eigentlichen Aktion. Ein Datenbankfehler beim Einstellen hätte dann auch diese Transaktion als fehlgeschlagen markiert und zum Beispiel das Anlegen eines Portfolios zurückgerollt (Spring Framework Documentation, „Transaction Propagation“).

3. **Zeitpunkt der Aktion statt der Verarbeitung.** Die Nachricht trägt den Zeitpunkt, an dem die Aktion stattfand. Sonst würde eine Anmeldung um 23:59:58 am Monatsende im Folgemonat gezählt, weil der Verbraucher sie erst Sekunden später verarbeitet. Die Monatsgrenzen selbst rechnet die Auswertung nach deutscher Zeit (E38).

Hinter der Schnittstelle `MessagePublisher` ist die Tabelle austauschbar: Ein späterer Wechsel auf einen Broker ändert keine Aufrufer (Protected Variations). Erfasst wird in den Services statt in den Controllern, damit eine Aktion über REST und über GraphQL gleich gezählt wird.

### 4.3.3 Fehler und Erkenntnisse

- **Die Reihenfolge der Schritte war entscheidend.** Zuerst entstand die Tabelle der Ereignisse (V9), dann der Transportweg über die Queue (V10), erst danach die acht Erfassungsstellen in den Services. Hätten wir die Erfassungsstellen zuerst direkt an die Datenbank angebunden, hätten wir alle acht nach der Entscheidung für die Queue noch einmal umbauen müssen.
- **Kleine Pull Requests machten Fehler sichtbar.** Der PR mit der Queue (BE #120) war zunächst rot, weil die Klasse `UsageEventConsumer` ohne Dateiendung `.java` übernommen worden war; ein Folge-Commit hat das behoben. Der PR mit der Migration V9 (BE #119) versprach in der Beschreibung auch das Feld `role` in der Login-Antwort, das erst ein weiterer PR (BE #121) nachlieferte. Beides fiel bei der Durchsicht auf, weil jeder PR nur einen Schritt enthielt und die Beschreibung mit dem Diff verglichen werden konnte. Gelernt haben wir daraus, den Diff vor dem Merge gegen die eigene Beschreibung zu lesen, nicht nur die CI abzuwarten.
- **Neue Technik und Grenzen unserer Tests.** Neu waren für uns Sperren auf Zeilenebene mit `SKIP LOCKED`, das Verhalten von Transaktionen bei verschachtelten Aufrufen und Monatsgrenzen in einer festen Zeitzone. Die Monatsgrenzen sind mit vorgegebener Uhr getestet (`UsageReportServiceTest`). Bei den Transaktionen stoßen unsere Unit-Tests dagegen an eine Grenze: Sie laufen ohne Datenbank und zeigen nur, dass ein Fehler beim Einstellen nicht weitergeworfen wird (`recordDoesNotThrowWhenQueueFails`, `failingQueueDoesNotBreakTheAction`). Dass `REQUIRES_NEW` die Transaktion der Aktion tatsächlich schützt, haben wir aus der Dokumentation abgeleitet, nicht mit einem Integrationstest gegen PostgreSQL nachgewiesen.
- **KI-Unterstützung braucht eine Prüfinstanz.** Teile der Queue wurden mit KI-Unterstützung entworfen (KI-Log). Die Unit-Tests und die CI waren dabei die eigentliche Prüfinstanz, weil der Entwurf erst dort ausgeführt wurde.
- **Was offen bleibt.** Nachrichten, die nach drei Versuchen als fehlgeschlagen markiert sind, bleiben in der Tabelle, ohne dass jemand davon erfährt. Wie lange die Verarbeitung im Betrieb tatsächlich dauert und ob das Vorladen die erste Simulation spürbar beschleunigt, haben wir nicht gemessen. In einem weiteren Sprint würden wir beides sichtbar machen, etwa mit einer Kennzahl für fehlgeschlagene Nachrichten im Dashboard und einer Messung der Simulationsdauer mit und ohne vorgeladene Kurse.

## 4.4 Zusammenfassung

| Anforderung der Guidelines | Umsetzung in LookUpStock | Was wir anders machen würden |
|---|---|---|
| Integration eines Fremdservice über mehr als einen GET-Aufruf | Twelve Data: Suche, Monatskurse, Einordnung der Fehlercodes, Abbildung auf eigene Meldungen; gekapselt im marktdaten-service | Annahmen über Tarif, Börsen und Bereinigung zu Projektbeginn mit Probeabrufen prüfen |
| Asynchrone Kommunikation zwischen Komponenten | Queue-Tabelle mit den Themen `nutzung.ereignis` und `kursdaten.vorladen`; synchrone Kursabfrage als begründete Ausnahme (ADR-07) | Fehlgeschlagene Nachrichten sichtbar machen, Verarbeitungsdauer messen |
| Infrastruktur-Optimierung | Such-Cache (`ConcurrentHashMap` im Arbeitsspeicher), Kurs-Cache (PostgreSQL), Vorladen der Kurse über die Queue | Längere Frist für Kurse bei einem Ausfall des Anbieters |

Quellen im Projekt: ADR-07, Simulationsmodell, Kurzformat-Tabellen (E7, E20, E22, E38, E45), NFRs (NF-01, NF-02, NF-06, NF-07, NF-08, NF-12), Abnahmeprotokolle Sprint 2 bis 4, Change Request CR-01, Pull Requests BE #119 bis #121, KI-Log.
