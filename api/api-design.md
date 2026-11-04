# API-Design – LookUpStock

Stand: 04.11.2026 · Basis-URL `/api/v1` · Datenformat JSON

Diese Markdown-Datei ist die bearbeitbare Fassung des API-Designs. Sie übernimmt den vollständigen Inhalt von `api-design.pdf` (Stand 02.10.2026; die PDF-Datei wurde am 25.10.2026 entfernt, maßgeblich ist diese Datei) und ergänzt die Entscheidungen des Product Owners vom 02.10.2026 (E1 bis E7) sowie die einzelnen Fehlerfälle für Portfolios und Simulationen.

**Anmeldung (ADR-06):** Der Token aus `POST /auth/login` wird bei allen Endpunkten, die eine Anmeldung erfordern, im Header `Authorization: Bearer <token>` mitgeschickt, nicht im Body. Fehlt der Header, hat er nicht das Format `Bearer <token>` oder ist der Token unbekannt bzw. abgelaufen, antwortet die API mit `401 UNAUTHORIZED` („Bitte melde dich an.“). Lesende Abfragen sind GET: Listen unter dem Sammelpfad (z. B. `GET /portfolios`), einzelne Einträge unter ihrer ID (z. B. `GET /portfolios/5`). Öffentliche Endpunkte (Registrieren, Anmelden, Anlagensuche, Gastmodus) brauchen keinen Header.

**Änderung 25.10.2026 (Docs #133, ADR-06):** Token aus allen Request-Bodys in den `Authorization`-Header verschoben. `POST /…/list` wurde zu `GET /…`, `POST /…/{id}/details` zu `GET /…/{id}` (Portfolios und Simulationen; die Pfade für gespeicherte Vergleiche sind entsprechend beschrieben, aber nicht umgesetzt, siehe Abschnitt „Vergleiche“). `DELETE` und `POST /auth/logout` haben keinen Body mehr. Bis ADR-04 galt: Token im JSON-Body, lesende Abfragen als POST.

**Änderung 25.10.2026 (Docs #149, BE #94):** Neuer Abschnitt „GraphQL (lesend)“: `POST /graphql` für Anlagensuche, eigene Portfolios und gespeicherte Simulationen; Anmeldung wie bei REST über den `Authorization`-Header (ADR-06). Schreibende Operationen laufen weiter über REST.

Änderung 30.10.2026 (Docs #165, CR-01, US-11, US-12): Neuer Abschnitt „Auswertungen“: GET /reports/usage liefert die neun Nutzungskennzahlen eines Monats (E39), GET /reports/usage/export dieselben Werte als CSV-Datei (E41). Beide nur mit der Rolle Geschäftsführung (E40); neuer Fehlercode 403 FORBIDDEN. POST /auth/login liefert zusätzlich das Feld role.

**Änderung 01.11.2026 (Docs #159, BE #108, ADR-07):** Abschnitt GraphQL um schreibende Operationen (Mutations) ergänzt: Portfolios anlegen, ändern und löschen, Simulation durchführen und speichern, gespeicherte Simulation löschen, Vergleich durchführen. Neuer Abschnitt „Interne Schnittstellen“ mit den Endpunkten des marktdaten-service und den Themen der Queue. Abschnitt „Anbindung der Finanzdaten-API“ an die Aufteilung in zwei Dienste angepasst. Für Aufrufer des core-service ändern sich Pfade und Antworten nicht.

**Änderung 04.11.2026 (Docs #213, BE #128, BE #129):** `GET /assets/{assetId}` gestrichen; der Endpunkt war seit Sprint 1 beschrieben, wurde aber nie umgesetzt und vom Frontend nicht gebraucht. Die nie dokumentierten Endpunkte `/assets/{symbol}/stats` und `/prices` sind im Code entfernt (Entscheidung im Sprint-Planning Sprint 6). Zählung von `POST /simulations` mit `save: true` nach E37 präzisiert (B8). Feld `micCode` in den Suchergebnissen ergänzt (wie im Code).

**Fehlerformat:** Fehler haben immer das Format `{ "error": { "code", "message", "details" } }`. `message` ist ein deutscher Text zur direkten Anzeige im Frontend. Gewichtungen in Prozent (Summe 100), Anteile in Ergebnissen als Dezimalzahl (0.31 = 31 %).

## Grundsätze und Regeln

- **Gewichtungen** werden in Prozent mit **höchstens 2 Nachkommastellen** angegeben (z. B. 33.33). Werte mit mehr Nachkommastellen werden mit `400 VALIDATION_ERROR` abgelehnt. Die Summe aller Gewichtungen eines Portfolios muss 100 ergeben, mit einer Toleranz von ± 0,01 Prozentpunkten. Jede Gewichtung muss größer als 0 sein.
- Das Backend verarbeitet Gewichtungen als **exakte Dezimalzahlen** (in Java `BigDecimal`), nicht als Gleitkommazahlen. Eine Summe wie 69.99 + 30.01 wird als genau 100.00 erkannt.
- Ein Portfolio hat einen **Namen** (Pflicht, höchstens 100 Zeichen) und **mindestens eine Position**. Eine **Höchstzahl an Anlagen** pro Portfolio gibt es nicht. Jede Anlage darf nur einmal enthalten sein; jede `assetId` muss eine bekannte Anlage bezeichnen.

## Anbindung der Finanzdaten-API

- Primärer Anbieter der Kurs- und Stammdaten ist **Twelve Data**.
- Ein Fallback-Anbieter wurde **nicht festgelegt**; fällt Twelve Data aus, greifen Kurs-Cache und `503` (NF-07).
- Das Frontend greift nie direkt auf die Finanzdaten-API zu. Seit ADR-07 ist die Anbindung in einem eigenen Dienst gekapselt, dem **marktdaten-service**. Der core-service, also die hier beschriebene API, fragt Suche und Kurse über das Interface `MarketDataProvider` beim marktdaten-service ab (`MarktdatenClient`, siehe Abschnitt „Interne Schnittstellen“). Der API-Key von Twelve Data liegt ausschließlich als Umgebungsvariable beim marktdaten-service (NF-06).
- Such-Cache und Kurs-Cache (jeweils 24 h) liegen im marktdaten-service. Ist Twelve Data nicht erreichbar, werden Kursdaten jünger als 24 h aus dem Cache weiterverwendet; andernfalls antworten die betroffenen Endpunkte mit `503 UPSTREAM_UNAVAILABLE` (NF-07). Dasselbe gilt, wenn der marktdaten-service selbst nicht erreichbar ist.

---

## Auth (US-01)

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/auth/register</code><br>Konto anlegen · Sprint 2</td>
<td>
<pre>
{
  "email": "anna@example.com",
  "password": "geheim123"
}
</pre>
</td>
<td>
<b>201 Created</b>
<pre>
{
  "id": 3,
  "email": "anna@example.com"
}
</pre>
<b>400 Bad Request</b>
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Das Passwort muss mindestens 8 Zeichen haben."
  }
}
</pre>
<b>409 Conflict</b>
<pre>
{
  "error": {
    "code": "CONFLICT",
    "message": "Diese E-Mail ist bereits registriert."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>POST</b></td>
<td><code>/auth/login</code><br>Anmelden · Sprint 2</td>
<td>
<pre>
{
  "email": "anna@example.com",
  "password": "geheim123"
}
</pre>
</td>
<td>
<b>200 OK</b>
<pre> { "token": "q8Z1vN3xR0tK7mW2pL5sA9dF4hJ6cB1eY0uG8iT3oQw", "expiresAt": "2026-10-13T10:00:00Z", "role": "USER" } </pre>
<code>role</code> ist <code>USER</code> oder <code>MANAGER</code> (Geschäftsführung, E40). Das Frontend blendet damit den Navigationseintrag „Auswertungen“ ein; geschützt wird allein durch die Prüfung im Backend.
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "E-Mail oder Passwort ist falsch."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>POST</b></td>
<td><code>/auth/logout</code><br>Abmelden · Sprint 2</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>204 No Content</b><br>
Kein Body. Der Token wird gelöscht und kann nicht mehr verwendet werden. Die Abmeldung ist idempotent: Auch ohne Header oder mit unbekanntem bzw. abgelaufenem Token antwortet die API mit 204 (ADR-04).
</td>
</tr>
</table>

## Anlagen (US-02)

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>GET</b></td>
<td><code>/assets/search?q=msci&amp;limit=10</code><br>Anlagen suchen · Sprint 1 · ohne Anmeldung</td>
<td>
Query: <code>q</code> (Pflicht, mind. 2 Zeichen: Ticker, ISIN oder Name), <code>limit</code> (optional, ganze Zahl von 1 bis 25, Standard 10; Werte außerhalb werden mit <code>400 VALIDATION_ERROR</code> abgelehnt). Kein Body.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "query": "msci",
  "results": [
    {
      "id": 17,
      "symbol": "EUNL",
      "isin": "IE00B4L5Y983",
      "name": "iShares Core MSCI World UCITS ETF",
      "exchange": "XETRA",
      "micCode": "XETR",
      "currency": "EUR",
      "type": "ETF"
    }
  ]
}
</pre>
<b>200 OK (keine Treffer)</b>
<pre>
{
  "query": "xyzabc",
  "results": []
}
</pre>
<b>400 Bad Request</b> (Suchbegriff zu kurz)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte mindestens 2 Zeichen eingeben."
  }
}
</pre>
<b>400 Bad Request</b> (<code>limit</code> außerhalb von 1 bis 25, z. B. <code>limit=26</code>)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Der Parameter limit muss zwischen 1 und 25 liegen.",
    "details": [
      {
        "field": "limit",
        "message": "Wert: 26"
      }
    ]
  }
}
</pre>
<b>503 Service Unavailable</b>
<pre>
{
  "error": {
    "code": "UPSTREAM_UNAVAILABLE",
    "message": "Kursdaten sind gerade nicht verfügbar."
  }
}
</pre>
</td>
</tr>
</table>

## Portfolios (US-03, US-06)

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/portfolios</code><br>Portfolio anlegen · Sprint 1</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "name": "Mein Weltportfolio",
  "positions": [
    {
      "assetId": 17,
      "weight": 70
    },
    {
      "assetId": 23,
      "weight": 30
    }
  ]
}
</pre>
<code>name</code>: Pflicht, höchstens 100 Zeichen. <code>positions</code>: mindestens eine Position, jede <code>assetId</code> höchstens einmal; <code>weight</code> größer als 0 mit höchstens 2 Nachkommastellen, Summe 100 (± 0,01).
</td>
<td>
<b>201 Created</b>
<pre>
{
  "id": 5,
  "name": "Mein Weltportfolio",
  "createdAt": "2026-09-30T10:12:00Z",
  "updatedAt": "2026-09-30T10:12:00Z",
  "positions": [
    {
      "assetId": 17,
      "symbol": "EUNL",
      "name": "iShares Core MSCI World",
      "weight": 70
    },
    {
      "assetId": 23,
      "symbol": "IS3N",
      "name": "iShares Core MSCI EM IMI",
      "weight": 30
    }
  ]
}
</pre>
<b>400 Bad Request</b> (Summe der Gewichtungen ungleich 100)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Die Gewichtungen müssen zusammen 100 % ergeben.",
    "details": [
      {
        "field": "positions",
        "message": "Summe der Gewichtungen ist 95 %."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Name fehlt oder ist länger als 100 Zeichen)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte einen Namen mit höchstens 100 Zeichen eingeben.",
    "details": [
      {
        "field": "name",
        "message": "Der Name hat 101 Zeichen."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (keine Position)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte mindestens eine Anlage hinzufügen.",
    "details": [
      {
        "field": "positions",
        "message": "Das Portfolio enthält keine Position."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Gewichtung kleiner oder gleich 0)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Jede Gewichtung muss größer als 0 % sein.",
    "details": [
      {
        "field": "positions",
        "message": "Gewichtung für assetId 23 ist 0 %."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (mehr als 2 Nachkommastellen)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Gewichtungen dürfen höchstens 2 Nachkommastellen haben.",
    "details": [
      {
        "field": "positions",
        "message": "Gewichtung für assetId 23 ist 33.333 %."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (doppelte Anlage)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Jede Anlage darf nur einmal im Portfolio enthalten sein.",
    "details": [
      {
        "field": "positions",
        "message": "assetId 17 ist mehrfach enthalten."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (unbekannte <code>assetId</code>)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Eine Anlage wurde nicht gefunden.",
    "details": [
      {
        "field": "positions",
        "message": "Unbekannte assetId 99999."
      }
    ]
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/portfolios</code><br>Eigene Portfolios auflisten · Sprint 1</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "portfolios": [
    {
      "id": 5,
      "name": "Mein Weltportfolio",
      "createdAt": "2026-09-30T10:12:00Z",
      "updatedAt": "2026-09-30T10:12:00Z",
      "positions": [
        {
          "assetId": 17,
          "symbol": "EUNL",
          "name": "iShares Core MSCI World",
          "weight": 70
        },
        {
          "assetId": 23,
          "symbol": "IS3N",
          "name": "iShares Core MSCI EM IMI",
          "weight": 30
        }
      ]
    }
  ]
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/portfolios/{portfolioId}</code><br>Ein Portfolio abrufen · Sprint 1</td>
<td>
Pfad: <code>portfolioId</code>, z. B. <code>/portfolios/5</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "id": 5,
  "name": "Mein Weltportfolio",
  "createdAt": "2026-09-30T10:12:00Z",
  "updatedAt": "2026-09-30T10:12:00Z",
  "positions": [
    {
      "assetId": 17,
      "symbol": "EUNL",
      "name": "iShares Core MSCI World",
      "weight": 70
    },
    {
      "assetId": 23,
      "symbol": "IS3N",
      "name": "iShares Core MSCI EM IMI",
      "weight": 30
    }
  ]
}
</pre>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Portfolio nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>PUT</b></td>
<td><code>/portfolios/{portfolioId}</code><br>Portfolio ändern · Sprint 2</td>
<td>
Pfad: <code>portfolioId</code>, z. B. <code>/portfolios/5</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "name": "Weltportfolio 80/20",
  "positions": [
    {
      "assetId": 17,
      "weight": 80
    },
    {
      "assetId": 23,
      "weight": 20
    }
  ]
}
</pre>
Regeln für <code>name</code> und <code>positions</code> wie bei <code>POST /portfolios</code>.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "id": 5,
  "name": "Weltportfolio 80/20",
  "createdAt": "2026-09-30T10:12:00Z",
  "updatedAt": "2026-10-06T08:00:00Z",
  "positions": [
    {
      "assetId": 17,
      "symbol": "EUNL",
      "name": "iShares Core MSCI World",
      "weight": 80
    },
    {
      "assetId": 23,
      "symbol": "IS3N",
      "name": "iShares Core MSCI EM IMI",
      "weight": 20
    }
  ]
}
</pre>
<b>400 Bad Request</b> (Summe der Gewichtungen ungleich 100)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Die Gewichtungen müssen zusammen 100 % ergeben."
  }
}
</pre>
<b>400 Bad Request</b> (Name fehlt oder ist länger als 100 Zeichen)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte einen Namen mit höchstens 100 Zeichen eingeben.",
    "details": [
      {
        "field": "name",
        "message": "Der Name fehlt."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (keine Position)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte mindestens eine Anlage hinzufügen.",
    "details": [
      {
        "field": "positions",
        "message": "Das Portfolio enthält keine Position."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Gewichtung kleiner oder gleich 0)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Jede Gewichtung muss größer als 0 % sein.",
    "details": [
      {
        "field": "positions",
        "message": "Gewichtung für assetId 23 ist -10 %."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (mehr als 2 Nachkommastellen)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Gewichtungen dürfen höchstens 2 Nachkommastellen haben.",
    "details": [
      {
        "field": "positions",
        "message": "Gewichtung für assetId 17 ist 80.005 %."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (doppelte Anlage)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Jede Anlage darf nur einmal im Portfolio enthalten sein.",
    "details": [
      {
        "field": "positions",
        "message": "assetId 23 ist mehrfach enthalten."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (unbekannte <code>assetId</code>)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Eine Anlage wurde nicht gefunden.",
    "details": [
      {
        "field": "positions",
        "message": "Unbekannte assetId 99999."
      }
    ]
  }
}
</pre>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Portfolio nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>DELETE</b></td>
<td><code>/portfolios/{portfolioId}</code><br>Portfolio löschen · Sprint 2</td>
<td>
Pfad: <code>portfolioId</code>, z. B. <code>/portfolios/5</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>204 No Content</b><br>
Kein Body<br>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Portfolio nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
</table>

## Simulationen (US-04, US-05, US-08)

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/simulations</code><br>Simulation berechnen, optional speichern · Sprint 1–3</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "portfolioId": 5,
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42,
    "targetAmount": 100000
  },
  "save": false,
  "name": null
}
</pre>
Pflicht: <code>portfolioId</code>, <code>startCapital</code>, <code>monthlyContribution</code>, <code>horizonYears</code>. Optional: <code>paths</code> (Standard 10.000, 1.000–50.000), <code>seed</code>, <code>targetAmount</code>. <code>save: true</code> speichert (ab Sprint 3, dann ist <code>name</code> Pflicht). Für die Nutzungskennzahlen zählt <code>save: true</code> nur als gespeicherte Simulation, ohne <code>save</code> nur als durchgeführte Simulation (E37, B8).<br>
Regeln: <code>startCapital</code> und <code>monthlyContribution</code> ≥ 0, nicht beide 0; <code>horizonYears</code> ganze Zahl von 1 bis 40; <code>seed</code> ganze Zahl; ohne <code>seed</code> erzeugt das System einen Seed und gibt ihn in <code>parameters.seed</code> zurück.
</td>
<td>
<b>200 OK (save: false) bzw. 201 Created (save: true)</b>
<pre>
{
  "id": null,
  "name": null,
  "portfolioId": 5,
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42,
    "targetAmount": 100000
  },
  "estimates": {
    "historyFrom": "2016-09-01",
    "historyTo": "2026-08-31",
    "assets": [
      {
        "assetId": 17,
        "annualReturn": 0.078,
        "annualVolatility": 0.152
      },
      {
        "assetId": 23,
        "annualReturn": 0.051,
        "annualVolatility": 0.183
      }
    ]
  },
  "timeline": [
    {
      "month": 0,
      "p5": 10000,
      "p50": 10000,
      "p95": 10000
    },
    {
      "month": 1,
      "p5": 9740.1,
      "p50": 10265.3,
      "p95": 10820.7
    },
    {
      "month": 240,
      "p5": 61200.1,
      "p50": 112340.55,
      "p95": 205880.9
    }
  ],
  "metrics": {
    "totalContributions": 58000,
    "medianEndValue": 112340.55,
    "p5EndValue": 61200.1,
    "p95EndValue": 205880.9,
    "valueAtRisk95": 0,
    "maxDrawdownMedian": 0.31,
    "targetProbability": 0.587
  },
  "runtimeMs": 840,
  "createdAt": "2026-09-30T10:15:00Z"
}
</pre>
<code>timeline</code> enthält einen Punkt pro Monat (Monat 0 bis <code>horizonYears</code> × 12); hier gekürzt auf drei Punkte.<br>
<code>metrics</code> enthält Einzahlungen, Endwert-Perzentile, Value at Risk (95 %) und den Median des maximalen Drawdowns. <code>targetProbability</code> bleibt null, solange kein Sparziel vorgegeben wird.<br>
<b>400 Bad Request</b> (Anlagehorizont außerhalb von 1 bis 40 Jahren)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Der Anlagehorizont muss eine ganze Zahl von 1 bis 40 Jahren sein.",
    "details": [
      {
        "field": "horizonYears",
        "message": "Wert: 41"
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Startkapital und Sparrate beide 0)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Startkapital und Sparrate dürfen nicht beide 0 sein.",
    "details": [
      {
        "field": "startCapital",
        "message": "Wert: 0"
      },
      {
        "field": "monthlyContribution",
        "message": "Wert: 0"
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (negativer oder fehlender Pflichtwert)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte alle Pflichtfelder mit Werten ab 0 ausfüllen.",
    "details": [
      {
        "field": "startCapital",
        "message": "Der Wert darf nicht negativ sein."
      },
      {
        "field": "monthlyContribution",
        "message": "Das Feld fehlt."
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Pfadanzahl außerhalb von 1.000 bis 50.000)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Die Anzahl der Simulationspfade muss zwischen 1.000 und 50.000 liegen.",
    "details": [
      {
        "field": "paths",
        "message": "Wert: 999"
      }
    ]
  }
}
</pre>
<b>400 Bad Request</b> (Seed keine ganze Zahl)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Der Seed muss eine ganze Zahl sein.",
    "details": [
      {
        "field": "seed",
        "message": "Wert: 4.2"
      }
    ]
  }
}
</pre>
<b>404 Not Found</b> (Portfolio nicht gefunden)
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Portfolio nicht gefunden."
  }
}
</pre>
<b>422 Unprocessable Entity</b> (weniger als 3 Jahre Kurshistorie; alle betroffenen Anlagen werden aufgeführt, hier zwei)
<pre>
{
  "error": {
    "code": "INSUFFICIENT_DATA",
    "message": "Für folgende Anlagen liegen weniger als 3 Jahre Kursdaten vor: iShares Core MSCI EM IMI, Beispiel-ETF mit kurzer Historie.",
    "details": [
      {
        "field": "assetId",
        "assetId": 23,
        "name": "iShares Core MSCI EM IMI",
        "message": "Weniger als 3 Jahre Kursdaten."
      },
      {
        "field": "assetId",
        "assetId": 42,
        "name": "Beispiel-ETF mit kurzer Historie",
        "message": "Weniger als 3 Jahre Kursdaten."
      }
    ]
  }
}
</pre>
<b>503 Service Unavailable</b> (Kursdaten nicht abrufbar und nicht im Cache)
<pre>
{
  "error": {
    "code": "UPSTREAM_UNAVAILABLE",
    "message": "Kursdaten sind gerade nicht verfügbar."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/simulations</code><br>Gespeicherte Simulationen auflisten · Sprint 3</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "simulations": [
    {
      "id": 12,
      "name": "Weltportfolio 20 Jahre",
      "portfolioId": 5,
      "createdAt": "2026-10-20T09:00:00Z"
    }
  ]
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/simulations/{simulationId}</code><br>Gespeicherte Simulation abrufen · Sprint 3</td>
<td>
Pfad: <code>simulationId</code>, z. B. <code>/simulations/12</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b><br>
Body wie bei <code>POST /simulations</code>, mit <code>id</code> und <code>name</code>, ohne Neuberechnung:
<pre>
{
  "id": 12,
  "name": "Weltportfolio 20 Jahre",
  "portfolioId": 5,
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42,
    "targetAmount": 100000
  },
  "estimates": {
    "historyFrom": "2016-09-01",
    "historyTo": "2026-08-31",
    "assets": [
      {
        "assetId": 17,
        "annualReturn": 0.078,
        "annualVolatility": 0.152
      },
      {
        "assetId": 23,
        "annualReturn": 0.051,
        "annualVolatility": 0.183
      }
    ]
  },
  "timeline": [
    {
      "month": 0,
      "p5": 10000,
      "p50": 10000,
      "p95": 10000
    },
    {
      "month": 240,
      "p5": 61200.1,
      "p50": 112340.55,
      "p95": 205880.9
    }
  ],
  "metrics": {
    "totalContributions": 58000,
    "medianEndValue": 112340.55,
    "p5EndValue": 61200.1,
    "p95EndValue": 205880.9,
    "valueAtRisk95": 0,
    "maxDrawdownMedian": 0.31,
    "targetProbability": 0.587
  },
  "runtimeMs": 840,
  "createdAt": "2026-09-30T10:15:00Z"
}
</pre>
<code>timeline</code> hier auf zwei Punkte gekürzt.<br>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Simulation nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>DELETE</b></td>
<td><code>/simulations/{simulationId}</code><br>Gespeicherte Simulation löschen · Sprint 3</td>
<td>
Pfad: <code>simulationId</code>, z. B. <code>/simulations/12</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>204 No Content</b><br>
Kein Body<br>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Simulation nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
</table>

## Vergleiche (US-07, US-08, US-10)

**Hinweis (25.10.2026):** Umgesetzt ist nur `POST /comparisons` mit genau zwei Items (US-07, BE #69). Vergleiche speichern (`save`, `name`) und die drei Endpunkte für gespeicherte Vergleiche werden nicht umgesetzt (E43, Change Request CR-01), ebenso mehr als zwei Items (US-10, Won't laut Sprint-Planning-Protokoll Sprint 4). `save` wird nicht ausgewertet. Die Beschreibung bleibt zur Nachvollziehbarkeit stehen.

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/comparisons</code><br>Vergleich berechnen · Sprint 3–4 (Speichern nicht umgesetzt, E43)</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "items": [
    {
      "label": "Mein Weltportfolio",
      "portfolioId": 5
    },
    {
      "label": "Nur S&amp;P 500",
      "assetId": 31
    }
  ],
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42
  },
  "save": false,
  "name": null
}
</pre>
Jedes Item hat entweder <code>portfolioId</code> oder <code>assetId</code>. Genau 2 Items (US-07); mehr als zwei (US-10) werden nicht umgesetzt. Alle Items werden mit denselben Zufallszahlen berechnet.
</td>
<td>
<b>200 OK bzw. 201 Created</b>
<pre>
{
  "id": null,
  "name": null,
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42
  },
  "results": [
    {
      "label": "Mein Weltportfolio",
      "portfolioId": 5,
      "timeline": [
        {
          "month": 0,
          "p5": 10000,
          "p50": 10000,
          "p95": 10000
        },
        {
          "month": 240,
          "p5": 61200.1,
          "p50": 112340.55,
          "p95": 205880.9
        }
      ],
      "metrics": {
        "totalContributions": 58000,
        "medianEndValue": 112340.55,
        "p5EndValue": 61200.1,
        "p95EndValue": 205880.9,
        "valueAtRisk95": 0,
        "maxDrawdownMedian": 0.31,
        "targetProbability": 0.587
      }
    },
    {
      "label": "Nur S&amp;P 500",
      "assetId": 31,
      "timeline": [
        {
          "month": 0,
          "p5": 10000,
          "p50": 10000,
          "p95": 10000
        },
        {
          "month": 240,
          "p5": 55800,
          "p50": 121900.4,
          "p95": 248300.7
        }
      ],
      "metrics": {
        "totalContributions": 58000,
        "medianEndValue": 121900.4,
        "p5EndValue": 55800,
        "p95EndValue": 248300.7,
        "valueAtRisk95": 0,
        "maxDrawdownMedian": 0.38,
        "targetProbability": 0.62
      }
    }
  ],
  "pairwise": [
    {
      "a": 0,
      "b": 1,
      "probabilityAOutperformsB": 0.46
    }
  ],
  "runtimeMs": 1620,
  "createdAt": "2026-10-20T09:00:00Z"
}
</pre>
<code>probabilityAOutperformsB</code>: Anteil der Pfade, in denen <code>results[a]</code> am Ende mehr wert ist als <code>results[b]</code>.<br>
<b>400 Bad Request</b>
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte genau zwei Einträge zum Vergleich auswählen."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/comparisons</code><br>Gespeicherte Vergleiche auflisten · <b>nicht umgesetzt (E43)</b></td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "comparisons": [
    {
      "id": 7,
      "name": "Welt vs. S&amp;P 500",
      "itemLabels": [
        "Mein Weltportfolio",
        "Nur S&amp;P 500"
      ],
      "createdAt": "2026-10-20T09:00:00Z"
    }
  ]
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/comparisons/{comparisonId}</code><br>Gespeicherten Vergleich abrufen · <b>nicht umgesetzt (E43)</b></td>
<td>
Pfad: <code>comparisonId</code>, z. B. <code>/comparisons/7</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>200 OK</b><br>
Body wie bei <code>POST /comparisons</code>, mit <code>id</code> und <code>name</code>, ohne Neuberechnung:
<pre>
{
  "id": 7,
  "name": "Welt vs. S&amp;P 500",
  "parameters": {
    "startCapital": 10000,
    "monthlyContribution": 200,
    "horizonYears": 20,
    "paths": 10000,
    "seed": 42
  },
  "results": [
    {
      "label": "Mein Weltportfolio",
      "portfolioId": 5,
      "timeline": [
        {
          "month": 0,
          "p5": 10000,
          "p50": 10000,
          "p95": 10000
        },
        {
          "month": 240,
          "p5": 61200.1,
          "p50": 112340.55,
          "p95": 205880.9
        }
      ],
      "metrics": {
        "medianEndValue": 112340.55
      }
    },
    {
      "label": "Nur S&amp;P 500",
      "assetId": 31,
      "timeline": [
        {
          "month": 0,
          "p5": 10000,
          "p50": 10000,
          "p95": 10000
        },
        {
          "month": 240,
          "p5": 55800,
          "p50": 121900.4,
          "p95": 248300.7
        }
      ],
      "metrics": {
        "medianEndValue": 121900.4
      }
    }
  ],
  "pairwise": [
    {
      "a": 0,
      "b": 1,
      "probabilityAOutperformsB": 0.46
    }
  ],
  "runtimeMs": 1620,
  "createdAt": "2026-10-20T09:00:00Z"
}
</pre>
<code>metrics</code> hier gekürzt; vollständig wie bei <code>POST /comparisons</code>.<br>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Vergleich nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>DELETE</b></td>
<td><code>/comparisons/{comparisonId}</code><br>Gespeicherten Vergleich löschen · <b>nicht umgesetzt (E43)</b></td>
<td>
Pfad: <code>comparisonId</code>, z. B. <code>/comparisons/7</code><br>
Header: <code>Authorization: Bearer &lt;token&gt;</code><br>
Kein Body.
</td>
<td>
<b>204 No Content</b><br>
Kein Body<br>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Vergleich nicht gefunden."
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
</td>
</tr>
</table>

## Gastmodus (US-09)

Beide Endpunkte sind öffentlich (kein `Authorization`-Header nötig) und speichern nichts. Die Beispielportfolios stehen im Code (`DemoPortfolio`), nicht in der Datenbank. Sie verwenden Anlagen an US-Börsen, weil Twelve Data im kostenlosen Tarif keine Xetra-Kurse liefert (F3, Docs #80; PO-Entscheidung vom 18.10.2026).

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>GET</b></td>
<td><code>/demo/portfolios</code><br>Beispielportfolios · Sprint 4 · ohne Anmeldung</td>
<td>Kein Body, keine Anmeldung</td>
<td>
<b>200 OK</b>
<pre>
{
  "portfolios": [
    {
      "id": 900,
      "name": "Beispiel: Weltportfolio",
      "positions": [
        {
          "symbol": "URTH",
          "name": "iShares MSCI World ETF",
          "weight": 100
        }
      ]
    },
    {
      "id": 901,
      "name": "Beispiel: Welt 70 / Schwellenländer 30",
      "positions": [
        {
          "symbol": "URTH",
          "name": "iShares MSCI World ETF",
          "weight": 70
        },
        {
          "symbol": "IEMG",
          "name": "iShares Core MSCI Emerging Markets ETF",
          "weight": 30
        }
      ]
    }
  ]
}
</pre>
Positionen ohne <code>assetId</code>: Beispielportfolios haben keine festen Anlage-IDs; die Anlagen werden erst bei der Simulation in <code>asset</code> gesucht oder angelegt.
</td>
</tr>
<tr>
<td><b>POST</b></td>
<td><code>/demo/simulations</code><br>Beispielsimulation ohne Speicherung · Sprint 4 · ohne Anmeldung</td>
<td>
<pre>
{
  "portfolioId": 901,
  "parameters": {
    "startCapital": 5000,
    "monthlyContribution": 100,
    "horizonYears": 15
  }
}
</pre>
Kein <code>Authorization</code>-Header nötig. Regeln für <code>parameters</code> wie bei <code>POST /simulations</code>, zusätzlich <code>paths</code> höchstens 10.000. Kein <code>save</code> und <code>name</code>.
</td>
<td>
<b>200 OK</b><br>
Body wie bei <code>POST /simulations</code> ohne Speichern: <code>id</code>, <code>name</code> und <code>createdAt</code> sind immer <code>null</code>.
<pre>
{
  "id": null,
  "name": null,
  "portfolioId": 901,
  "parameters": {
    "startCapital": 5000,
    "monthlyContribution": 100,
    "horizonYears": 15,
    "paths": 10000,
    "seed": 731,
    "targetAmount": null
  },
  "estimates": {
    "historyFrom": "2016-10-01",
    "historyTo": "2026-09-01",
    "assets": [
      { "assetId": 41, "annualReturn": 0.081, "annualVolatility": 0.152 },
      { "assetId": 42, "annualReturn": 0.044, "annualVolatility": 0.178 }
    ]
  },
  "timeline": [
    { "month": 0, "p5": 5000, "p50": 5000, "p95": 5000 },
    { "month": 180, "p5": 24100.3, "p50": 41250.8, "p95": 70980.2 }
  ],
  "metrics": {
    "totalContributions": 23000,
    "medianEndValue": 41250.8,
    "p5EndValue": 24100.3,
    "p95EndValue": 70980.2,
    "valueAtRisk95": 0,
    "maxDrawdownMedian": 0.29,
    "targetProbability": null
  },
  "runtimeMs": 610,
  "createdAt": null
}
</pre>
<code>timeline</code> hier gekürzt; die IDs in <code>estimates.assets</code> sind die Anlage-IDs in der Datenbank.<br>
<b>400 Bad Request</b> (mehr als 10.000 Pfade)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "In der Beispielsimulation sind höchstens 10.000 Simulationspfade möglich.",
    "details": [ { "field": "paths", "message": "Wert: 20000" } ]
  }
}
</pre>
Übrige Prüffehler (400) wie bei <code>POST /simulations</code>.<br>
<b>404 Not Found</b> (unbekanntes Beispielportfolio)
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Beispielportfolio nicht gefunden."
  }
}
</pre>
<b>422 Unprocessable Entity</b> und <b>503 Service Unavailable</b> wie bei <code>POST /simulations</code> (zu wenig Kursdaten bzw. Finanzdaten-API nicht erreichbar).
</td>
</tr>
</table>

## Auswertungen (US-11, US-12)
 
Nutzungskennzahlen für die Geschäftsführung (Change Request CR-01). Grundlage sind die anonym erfassten Nutzungsereignisse (E36, E37), die über die Queue übertragen werden (ADR-07); neue Ereignisse erscheinen deshalb mit einigen Sekunden Verzögerung.
 
**Zugriff (E40):** Beide Endpunkte erfordern den Header `Authorization: Bearer <token>` (ADR-06) und die Rolle `MANAGER`. Ohne Anmeldung `401 UNAUTHORIZED`, angemeldet ohne die Rolle `403 FORBIDDEN`. Die Rolle wird vom Team in der Datenbank vergeben (README des Backends), nicht über die API.
 
**Monat (E38):** Parameter `month` im Format `JJJJ-MM`, Kalendermonat nach deutscher Zeit (Europe/Berlin). Ohne `month` gilt der laufende Monat; er ist mit `"preliminary": true` gekennzeichnet. Monate vor dem Beginn der Erfassung und künftige Monate werden mit 400 abgelehnt, es werden keine Nullwerte geliefert.
 
**Kennzahlen (E39):** feste Reihenfolge, `label` ist der deutsche Anzeigetext und wird im Dashboard und in der CSV-Datei unverändert verwendet.
 
| `key` | `label` | Quelle |
|---|---|---|
| `registrations` | Neue Registrierungen | Ereignis |
| `logins` | Anmeldungen | Ereignis |
| `portfoliosCreated` | Angelegte Portfolios | Ereignis |
| `simulationsRun` | Durchgeführte Simulationen | Ereignis |
| `simulationsSaved` | Gespeicherte Simulationen | Ereignis |
| `comparisonsRun` | Durchgeführte Vergleiche | Ereignis |
| `demoSimulationsRun` | Beispielsimulationen (Gäste) | Ereignis |
| `assetSearches` | Anlagensuchen | Ereignis |
| `totalAccounts` | Nutzerkonten insgesamt am Monatsende | Zählung der Nutzerkonten |
 
Für den laufenden Monat ist `totalAccounts` der Stand zum Zeitpunkt der Abfrage.
 
<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>GET</b></td>
<td><code>/reports/usage?month=2026-10</code><br>Kennzahlen eines Monats · Sprint 5 · Rolle Geschäftsführung</td>
<td>Kein Body. Header <code>Authorization: Bearer &lt;token&gt;</code>. Query-Parameter <code>month</code> optional (<code>JJJJ-MM</code>), ohne Angabe der laufende Monat.</td>
<td>
<b>200 OK</b>
<pre>
{
  "month": "2026-10",
  "preliminary": true,
  "trackingSince": "2026-10-31",
  "metrics": [
    { "key": "registrations", "label": "Neue Registrierungen", "value": 4 },
    { "key": "logins", "label": "Anmeldungen", "value": 27 },
    { "key": "portfoliosCreated", "label": "Angelegte Portfolios", "value": 9 },
    { "key": "simulationsRun", "label": "Durchgeführte Simulationen", "value": 41 },
    { "key": "simulationsSaved", "label": "Gespeicherte Simulationen", "value": 12 },
    { "key": "comparisonsRun", "label": "Durchgeführte Vergleiche", "value": 7 },
    { "key": "demoSimulationsRun", "label": "Beispielsimulationen (Gäste)", "value": 15 },
    { "key": "assetSearches", "label": "Anlagensuchen", "value": 63 },
    { "key": "totalAccounts", "label": "Nutzerkonten insgesamt am Monatsende", "value": 38 }
  ]
}
</pre>
<code>trackingSince</code> ist der Tag des ersten erfassten Ereignisses (Beginn der Erfassung). Eine über die Oberfläche gespeicherte Simulation zählt einmal in <code>simulationsRun</code> (beim Simulieren) und einmal in <code>simulationsSaved</code> (beim Speichern mit <code>save: true</code>); das Speichern selbst erhöht <code>simulationsRun</code> nicht (E37, B8).<br>
<b>400 Bad Request</b> (Monat vor Beginn der Erfassung oder in der Zukunft)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Für diesen Monat liegen keine Auswertungen vor."
  }
}
</pre>
<b>400 Bad Request</b> (falsches Format)
<pre>
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Bitte gib den Monat im Format JJJJ-MM an.",
    "details": [ { "field": "month", "message": "Wert: 10-2026" } ]
  }
}
</pre>
<b>401 Unauthorized</b>
<pre>
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Bitte melde dich an."
  }
}
</pre>
<b>403 Forbidden</b>
<pre>
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Dafür fehlt die Berechtigung."
  }
}
</pre>
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/reports/usage/export?month=2026-10</code><br>Kennzahlen als CSV-Datei · Sprint 5 · Rolle Geschäftsführung</td>
<td>Wie <code>GET /reports/usage</code>.</td>
<td>
<b>200 OK</b><br>
Header:<br>
<code>Content-Type: text/csv; charset=UTF-8</code><br>
<code>Content-Disposition: attachment; filename="lookupstock-nutzung-2026-10.csv"</code><br>
Body (UTF-8 mit BOM, Trennzeichen Semikolon, Zeilenende CRLF, E41):
<pre>
Monat;Kennzahl;Wert
2026-10;Neue Registrierungen;4
2026-10;Anmeldungen;27
2026-10;Angelegte Portfolios;9
2026-10;Durchgeführte Simulationen;41
2026-10;Gespeicherte Simulationen;12
2026-10;Durchgeführte Vergleiche;7
2026-10;Beispielsimulationen (Gäste);15
2026-10;Anlagensuchen;63
2026-10;Nutzerkonten insgesamt am Monatsende;38
</pre>
Dieselben Bezeichnungen, Werte und Reihenfolge wie <code>GET /reports/usage</code>. Weil der Token im Header steht (ADR-06), lädt das Frontend die Datei per <code>fetch</code> und bietet sie als Blob zum Download an; ein einfacher Link reicht nicht.<br>
<b>400</b>, <b>401</b>, <b>403</b> wie bei <code>GET /reports/usage</code>, als JSON im Fehlerformat.
</td>
</tr>
</table>
Auswertungen gibt es nur über REST, nicht über GraphQL.

## GraphQL (BE #94, BE #108)

Ergänzend zu REST gibt es eine GraphQL-Schnittstelle mit vergleichbarem Funktionsumfang. Abfragen (Queries) liefern dieselben Daten wie die lesenden REST-Endpunkte, aber der Client wählt die Felder selbst und kann z. B. eine gespeicherte Simulation samt Portfolio und Kennzahlen in einer Anfrage abrufen. Schreibende Operationen (Mutations, seit Sprint 5) rufen dieselben Services mit denselben Prüfregeln und Meldungen auf wie die REST-Endpunkte. Ausgenommen sind Registrieren, Anmelden, Gastmodus und Auswertungen; sie gibt es nur über REST. Das Schema steht in `core-service/src/main/resources/graphql/schema.graphqls` (Backend).

**Anmeldung (ADR-06):** wie bei REST über den Header `Authorization: Bearer <token>`. Der Header wird zentral gelesen; geprüft wird er erst in den Abfragen, die eine Anmeldung brauchen. `searchAssets` ist ohne Anmeldung nutzbar.

**Fehlerformat:** GraphQL antwortet auch bei Fehlern mit `200 OK`. Fehler stehen im Feld `errors`; `message` ist derselbe deutsche Text wie bei REST, `extensions.code` derselbe Code (`UNAUTHORIZED`, `NOT_FOUND`, `VALIDATION_ERROR`, `INSUFFICIENT_DATA`, `CONFLICT`, `UPSTREAM_UNAVAILABLE`). Die Einzelfehler, bei REST in `error.details`, stehen in `extensions.details`, z. B. `[{ "field": "name", "message": "…" }]` oder bei `INSUFFICIENT_DATA` `[{ "field": "assetId", "assetId": 8, "name": "…", "message": "…" }]`. Das betroffene Feld in `data` ist dann `null`; bei Listen-Abfragen wie `savedSimulations` ist `data` insgesamt `null`, weil das Feld laut Schema nicht `null` sein darf (`!`).

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/graphql</code><br>Lesende Abfragen · Sprint 4 · Anmeldung je nach Abfrage</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code> (außer bei <code>searchAssets</code>)
<pre>
{
  "query": "query { savedSimulations { id name createdAt portfolio { name } metrics { medianEndValue targetProbability } } }"
}
</pre>
Optional <code>variables</code> für Argumente, z. B. <code>{ "query": "query($id: ID!) { portfolio(id: $id) { name positions { symbol weight } } }", "variables": { "id": "5" } }</code>.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "data": {
    "savedSimulations": [
      {
        "id": "12",
        "name": "Sparplan 20 Jahre",
        "createdAt": "2026-10-18T19:30:00Z",
        "portfolio": { "name": "Mein Weltportfolio" },
        "metrics": { "medianEndValue": 104250.6, "targetProbability": null }
      }
    ]
  }
}
</pre>
<b>200 OK mit Fehler</b> (ohne gültigen Header)
<pre>
{
  "errors": [
    {
      "message": "Bitte melde dich an.",
      "path": ["savedSimulations"],
      "extensions": { "classification": "UNAUTHORIZED", "code": "UNAUTHORIZED" }
    }
  ],
  "data": null
}
</pre>
</td>
</tr>
</table>

**Abfragen**

| Abfrage | Anmeldung | Liefert | Entspricht REST |
|---|---|---|---|
| `searchAssets(query: String!, limit: Int)` | nein | Liste von `Asset`; `limit` 1 bis 25, Standard 10 | `GET /assets/search` |
| `portfolios` | ja | eigene Portfolios, neueste zuerst | `GET /portfolios` |
| `portfolio(id: ID!)` | ja | ein eigenes Portfolio; fremde oder unbekannte ID → Fehler `NOT_FOUND` (NF-04) | `GET /portfolios/{id}` |
| `savedSimulations` | ja | eigene gespeicherte Simulationen, neueste zuerst | `GET /simulations` |
| `savedSimulation(id: ID!)` | ja | eine eigene gespeicherte Simulation; fremde oder unbekannte ID → Fehler `NOT_FOUND` (NF-04) | `GET /simulations/{id}` |

**Typen**

| Typ | Felder |
|---|---|
| `Asset` | `id`, `symbol`, `isin`, `name`, `exchange`, `micCode`, `currency`, `type` |
| `Portfolio` | `id`, `name`, `createdAt` (ISO-8601, UTC), `updatedAt`, `positions` |
| `Position` | `assetId`, `symbol`, `name`, `weight` (Prozent) |
| `SavedSimulation` | `id`, `name`, `portfolioId`, `createdAt`, `portfolio` und `metrics` (werden nur geladen, wenn die Abfrage sie anfordert) |
| `Metrics` | `totalContributions`, `medianEndValue`, `p5EndValue`, `p95EndValue`, `valueAtRisk95`, `maxDrawdownMedian` (Anteil, 0.31 = 31 %), `targetProbability` (`null` ohne Zielbetrag) |

IDs sind im GraphQL-Typ `ID` und kommen daher als Zeichenkette (`"12"`). Als Argument sind Zahl und Zeichenkette erlaubt (`id: 5` oder `id: "5"`). Die Oberfläche GraphiQL zum Ausprobieren ist auf Heroku ausgeschaltet und lässt sich lokal mit `GRAPHIQL_ENABLED=true` in der `.env` einschalten.

### Mutations (BE #108)

Alle Mutations erfordern die Anmeldung über den Header `Authorization: Bearer <token>`; ohne gültigen Header folgt der Fehler `UNAUTHORIZED`, bevor etwas geprüft oder geändert wird. Die Felder der Eingabetypen sind bewusst optional: Ein fehlender Wert liefert dieselbe Meldung wie bei REST (z. B. „Bitte einen Namen mit höchstens 100 Zeichen eingeben.“) statt eines allgemeinen GraphQL-Fehlers.

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>POST</b></td>
<td><code>/graphql</code><br>Portfolio anlegen · Sprint 5 · Anmeldung erforderlich</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "query": "mutation($input: PortfolioInput!) { createPortfolio(input: $input) { id name positions { symbol weight } } }",
  "variables": {
    "input": {
      "name": "Mein Weltportfolio",
      "positions": [
        { "assetId": "1", "weight": 70 },
        { "assetId": "2", "weight": 30 }
      ]
    }
  }
}
</pre>
</td>
<td>
<b>200 OK</b>
<pre>
{
  "data": {
    "createPortfolio": {
      "id": "5",
      "name": "Mein Weltportfolio",
      "positions": [
        { "symbol": "URTH", "weight": 70.0 },
        { "symbol": "IEMG", "weight": 30.0 }
      ]
    }
  }
}
</pre>
<b>200 OK mit Fehler</b> (Gewichtungen ergeben nicht 100)
<pre>
{
  "errors": [
    {
      "message": "Die Gewichtungen müssen zusammen 100 % ergeben.",
      "path": ["createPortfolio"],
      "extensions": {
        "classification": "BAD_REQUEST",
        "code": "VALIDATION_ERROR",
        "details": [
          { "field": "positions", "message": "Die Gewichtungen müssen zusammen 100 % ergeben." },
          { "field": "positions", "message": "Summe der Gewichtungen ist 95 %." }
        ]
      }
    }
  ],
  "data": null
}
</pre>
</td>
</tr>
<tr>
<td><b>POST</b></td>
<td><code>/graphql</code><br>Simulation durchführen und speichern · Sprint 5 · Anmeldung erforderlich</td>
<td>
Header: <code>Authorization: Bearer &lt;token&gt;</code>
<pre>
{
  "query": "mutation($input: SimulationInput!) { simulate(input: $input) { id name parameters { seed paths } metrics { medianEndValue p5EndValue p95EndValue } runtimeMs } }",
  "variables": {
    "input": {
      "portfolioId": "5",
      "parameters": {
        "startCapital": 10000,
        "monthlyContribution": 200,
        "horizonYears": 20,
        "seed": 42
      },
      "save": true,
      "name": "Sparplan 20 Jahre"
    }
  }
}
</pre>
Ohne <code>save</code> oder mit <code>save: false</code> wird nur gerechnet; <code>id</code>, <code>name</code> und <code>createdAt</code> sind dann <code>null</code>.
</td>
<td>
<b>200 OK</b>
<pre>
{
  "data": {
    "simulate": {
      "id": "12",
      "name": "Sparplan 20 Jahre",
      "parameters": { "seed": 42.0, "paths": 10000 },
      "metrics": {
        "medianEndValue": 104250.6,
        "p5EndValue": 71230.4,
        "p95EndValue": 168900.1
      },
      "runtimeMs": 412.0
    }
  }
}
</pre>
<b>200 OK mit Fehler</b> (weniger als 3 Jahre Kursdaten)
<pre>
{
  "errors": [
    {
      "message": "Für folgende Anlagen liegen weniger als 3 Jahre Kursdaten vor: Junger ETF.",
      "path": ["simulate"],
      "extensions": {
        "classification": "BAD_REQUEST",
        "code": "INSUFFICIENT_DATA",
        "details": [
          { "field": "assetId", "assetId": 8, "name": "Junger ETF", "message": "Weniger als 3 Jahre Kursdaten." }
        ]
      }
    }
  ],
  "data": null
}
</pre>
</td>
</tr>
</table>

| Mutation | Liefert | Entspricht REST |
|---|---|---|
| `createPortfolio(input: PortfolioInput!)` | `Portfolio` | `POST /portfolios` |
| `updatePortfolio(id: ID!, input: PortfolioInput!)` | `Portfolio`; fremde oder unbekannte ID → `NOT_FOUND` (NF-04) | `PUT /portfolios/{id}` |
| `deletePortfolio(id: ID!)` | `true`; fremde oder unbekannte ID → `NOT_FOUND` | `DELETE /portfolios/{id}` (204) |
| `simulate(input: SimulationInput!)` | `SimulationResult`; mit `save: true` zusätzlich gespeichert, `name` dann Pflicht und vor dem Rechnen geprüft | `POST /simulations` |
| `deleteSavedSimulation(id: ID!)` | `true`; fremde oder unbekannte ID → `NOT_FOUND` | `DELETE /simulations/{id}` (204) |
| `compare(input: ComparisonInput!)` | `ComparisonResult` mit gemeinsamem Seed und Schätzzeitraum (ADR-05) | `POST /comparisons` |

Unterschied zu REST: GraphQL kennt keine HTTP-Statuscodes je Fehler. Statt 400, 404, 422 oder 503 steht der Code in `extensions.code`; statt `204 No Content` beim Löschen liefert die Mutation `true`.

**Eingabetypen**

| Typ | Felder |
|---|---|
| `PortfolioInput` | `name`, `positions` (Liste von `PositionInput`) |
| `PositionInput` | `assetId` (`ID`), `weight` (Prozent, höchstens 2 Nachkommastellen) |
| `SimulationParametersInput` | `startCapital`, `monthlyContribution`, `horizonYears` (`Int`, 1 bis 40), `paths` (`Int`, 1.000 bis 50.000, Standard 10.000), `seed` (`Float`, ganze Zahl; exakt bis 2^53), `targetAmount` |
| `SimulationInput` | `portfolioId` (`ID`), `parameters`, `save` (`Boolean`), `name` |
| `ComparisonInput` | `items` (Liste von `ComparisonItemInput`), `parameters` |
| `ComparisonItemInput` | `label`, genau eines von `portfolioId` und `assetId` |

**Ergebnistypen**

| Typ | Felder |
|---|---|
| `SimulationResult` | `id`, `name`, `createdAt` (alle `null`, wenn nicht gespeichert), `portfolioId`, `parameters`, `estimates`, `timeline`, `metrics`, `runtimeMs` |
| `SimulationParameters` | `startCapital`, `monthlyContribution`, `horizonYears`, `paths`, `seed` (der verwendete, auch wenn keiner angegeben war; NF-08), `targetAmount` |
| `Estimates` | `historyFrom`, `historyTo` (Schätzzeitraum, ISO-8601), `assets` (Liste von `AssetEstimate`: `assetId`, `annualReturn`, `annualVolatility`) |
| `TimelinePoint` | `month` (ab 0), `p5`, `p50`, `p95` |
| `ComparisonResult` | `parameters`, `estimates`, `results` (Liste von `ComparisonItem`: `label`, `portfolioId`, `assetId`, `timeline`, `metrics`), `pairwise` (Liste von `PairwiseComparison`: `a`, `b`, `probabilityAOutperformsB`), `runtimeMs`, `createdAt` |

Ganze Zahlen über 32 Bit (`seed`, `runtimeMs`) sind als `Float` typisiert, weil GraphQL-`Int` auf 32 Bit begrenzt ist.

## Interne Schnittstellen (ADR-07)

Diese Schnittstellen nutzt nur der core-service. Sie sind nicht Teil der öffentlichen API und für das Frontend nicht erreichbar.

### marktdaten-service

Eigene Heroku-App; Basisadresse in der Umgebungsvariable `MARKTDATEN_URL` des core-service. Jede Anfrage an `/internal/**` braucht den Header `X-Internal-Api-Key` mit dem gemeinsamen Schlüssel aus `INTERNAL_API_KEY`; fehlt er oder passt er nicht, folgt `401 UNAUTHORIZED`. Fehler haben dasselbe Format wie die öffentliche API. Der core-service wartet höchstens 5 Sekunden und gibt jeden Fehler des marktdaten-service als `503 UPSTREAM_UNAVAILABLE` weiter (synchrone Ausnahme laut ADR-07).

<table>
<tr><th>Method</th><th>Ressource</th><th>Data In</th><th>Data Out</th></tr>
<tr>
<td><b>GET</b></td>
<td><code>/internal/search?q=msci</code><br>Anlagensuche · Such-Cache 24 h (NF-02)</td>
<td>Header <code>X-Internal-Api-Key</code>. Query-Parameter <code>q</code> (Pflicht). Mindestlänge und <code>limit</code> prüft der core-service.</td>
<td>
<b>200 OK</b>
<pre>
[
  {
    "symbol": "URTH",
    "name": "iShares MSCI World ETF",
    "exchange": "NYSE",
    "micCode": "ARCX",
    "currency": "USD",
    "type": "ETF"
  }
]
</pre>
<b>400</b> ohne <code>q</code> · <b>401</b> ohne gültigen Schlüssel · <b>503</b> Twelve Data nicht erreichbar
</td>
</tr>
<tr>
<td><b>GET</b></td>
<td><code>/internal/prices?symbol=URTH&amp;mic=ARCX&amp;interval=1month&amp;outputsize=120</code><br>Historische Kurse · Kurs-Cache 24 h für Monatswerte (NF-07)</td>
<td>Header <code>X-Internal-Api-Key</code>. <code>symbol</code> Pflicht; <code>mic</code> optional (ohne Angabe Standardbörse des Anbieters); <code>interval</code> Standard <code>1month</code>; <code>outputsize</code> 1 bis 5000, Standard 120. Symbol als Query-Parameter, damit auch Symbole mit „/“ funktionieren.</td>
<td>
<b>200 OK</b>, aufsteigend nach Datum, 6 Nachkommastellen
<pre>
[
  { "date": "2016-11-01", "close": 71.235000 },
  { "date": "2016-12-01", "close": 72.880000 }
]
</pre>
Enthält auch den laufenden Monat; abgeschlossene Monate wählt der core-service aus (E22). Unbekanntes Symbol oder Börse außerhalb des Tarifs: leere Liste.<br>
<b>400</b> ohne <code>symbol</code> oder <code>outputsize</code> außerhalb · <b>401</b> · <b>503</b>
</td>
</tr>
</table>

Ohne Schlüssel erreichbar ist nur `GET /actuator/health`.

### Queue (asynchrone Kommunikation)

Nachrichten liegen als Zeilen in der Tabelle `queue_message` (Migration V10 des core-service). Verbraucher holen sie in festen Abständen mit `FOR UPDATE SKIP LOCKED` ab, löschen sie nach erfolgreicher Verarbeitung und markieren sie nach drei Fehlversuchen als fehlgeschlagen. Der Absender wartet nicht auf die Verarbeitung; schlägt das Einstellen fehl, bleibt die eigentliche Aktion trotzdem erfolgreich.

| Thema | Absender → Verbraucher | Auslöser | Inhalt (`payload`) | Verarbeitung |
|---|---|---|---|---|
| `nutzung.ereignis` | core-service → core-service (`UsageEventConsumer`) | erfolgreiche Aktion an einer der acht Stellen aus E37 | `{ "type": "PORTFOLIO_CREATED", "occurredAt": "2026-11-01T14:21:00Z" }` | alle 5 s höchstens 50 Nachrichten; Übertrag in die Tabelle der Nutzungsereignisse (US-11) |
| `kursdaten.vorladen` | core-service → marktdaten-service (`PricePreloadConsumer`) | Portfolio angelegt oder geändert | `{ "instruments": [ { "symbol": "URTH", "micCode": "ARCX" } ] }` | alle 15 s höchstens 3 Nachrichten; fehlende oder veraltete Monatswerte in den Kurs-Cache laden (NF-07). Wenige Nachrichten je Durchlauf wegen des Kontingents von Twelve Data |
