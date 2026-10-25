# API-Design – LookUpStock

Stand: 25.10.2026 · Basis-URL `/api/v1` · Datenformat JSON

Diese Markdown-Datei ist die bearbeitbare Fassung des API-Designs. Sie übernimmt den vollständigen Inhalt von `api-design.pdf` (Stand 02.10.2026; die PDF-Datei ist veraltet, maßgeblich ist diese Datei) und ergänzt die Entscheidungen des Product Owners vom 02.10.2026 (E1 bis E7) sowie die einzelnen Fehlerfälle für Portfolios und Simulationen.

**Anmeldung (ADR-06):** Der Token aus `POST /auth/login` wird bei allen Endpunkten, die eine Anmeldung erfordern, im Header `Authorization: Bearer <token>` mitgeschickt, nicht im Body. Fehlt der Header, hat er nicht das Format `Bearer <token>` oder ist der Token unbekannt bzw. abgelaufen, antwortet die API mit `401 UNAUTHORIZED` („Bitte melde dich an.“). Lesende Abfragen sind GET: Listen unter dem Sammelpfad (z. B. `GET /portfolios`), einzelne Einträge unter ihrer ID (z. B. `GET /portfolios/5`). Öffentliche Endpunkte (Registrieren, Anmelden, Anlagensuche, Gastmodus) brauchen keinen Header.

**Änderung 25.10.2026 (Docs #133, ADR-06):** Token aus allen Request-Bodys in den `Authorization`-Header verschoben. `POST /…/list` wurde zu `GET /…`, `POST /…/{id}/details` zu `GET /…/{id}` (Portfolios und Simulationen; die Pfade für gespeicherte Vergleiche sind entsprechend beschrieben, aber nicht umgesetzt, siehe Abschnitt „Vergleiche“). `DELETE` und `POST /auth/logout` haben keinen Body mehr. Bis ADR-04 galt: Token im JSON-Body, lesende Abfragen als POST.

**Fehlerformat:** Fehler haben immer das Format `{ "error": { "code", "message", "details" } }`. `message` ist ein deutscher Text zur direkten Anzeige im Frontend. Gewichtungen in Prozent (Summe 100), Anteile in Ergebnissen als Dezimalzahl (0.31 = 31 %).

## Grundsätze und Regeln

- **Gewichtungen** werden in Prozent mit **höchstens 2 Nachkommastellen** angegeben (z. B. 33.33). Werte mit mehr Nachkommastellen werden mit `400 VALIDATION_ERROR` abgelehnt. Die Summe aller Gewichtungen eines Portfolios muss 100 ergeben, mit einer Toleranz von ± 0,01 Prozentpunkten. Jede Gewichtung muss größer als 0 sein.
- Das Backend verarbeitet Gewichtungen als **exakte Dezimalzahlen** (in Java `BigDecimal`), nicht als Gleitkommazahlen. Eine Summe wie 69.99 + 30.01 wird als genau 100.00 erkannt.
- Ein Portfolio hat einen **Namen** (Pflicht, höchstens 100 Zeichen) und **mindestens eine Position**. Eine **Höchstzahl an Anlagen** pro Portfolio gibt es nicht. Jede Anlage darf nur einmal enthalten sein; jede `assetId` muss eine bekannte Anlage bezeichnen.

## Anbindung der Finanzdaten-API

- Primärer Anbieter der Kurs- und Stammdaten ist **Twelve Data**.
- Ein Fallback-Anbieter ist **noch nicht entschieden**.
- Das Frontend greift nie direkt auf die Finanzdaten-API zu. Der Zugriff erfolgt nur im Backend über das Interface `MarketDataProvider`; der API-Key liegt ausschließlich als Umgebungsvariable im Backend (NF-06).
- Ist die Finanzdaten-API nicht erreichbar, werden Kursdaten jünger als 24 h aus dem Cache weiterverwendet; andernfalls antworten die betroffenen Endpunkte mit `503 UPSTREAM_UNAVAILABLE` (NF-07).

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
<pre>
{
  "token": "q8Z1vN3xR0tK7mW2pL5sA9dF4hJ6cB1eY0uG8iT3oQw",
  "expiresAt": "2026-10-13T10:00:00Z"
}
</pre>
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
<tr>
<td><b>GET</b></td>
<td><code>/assets/{assetId}</code><br>Details einer Anlage · Sprint 1 · ohne Anmeldung</td>
<td>Pfad: <code>assetId</code>, z. B. <code>/assets/17</code>. Kein Body.</td>
<td>
<b>200 OK</b>
<pre>
{
  "id": 17,
  "symbol": "EUNL",
  "isin": "IE00B4L5Y983",
  "name": "iShares Core MSCI World UCITS ETF",
  "exchange": "XETRA",
  "currency": "EUR",
  "type": "ETF"
}
</pre>
<b>404 Not Found</b>
<pre>
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Anlage nicht gefunden."
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
Pflicht: <code>portfolioId</code>, <code>startCapital</code>, <code>monthlyContribution</code>, <code>horizonYears</code>. Optional: <code>paths</code> (Standard 10.000, 1.000–50.000), <code>seed</code>, <code>targetAmount</code>. <code>save: true</code> speichert (ab Sprint 3, dann ist <code>name</code> Pflicht).<br>
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
          "assetId": 17,
          "symbol": "EUNL",
          "name": "iShares Core MSCI World",
          "weight": 100
        }
      ]
    }
  ]
}
</pre>
</td>
</tr>
<tr>
<td><b>POST</b></td>
<td><code>/demo/simulations</code><br>Beispielsimulation ohne Speicherung · Sprint 4 · ohne Anmeldung</td>
<td>
<pre>
{
  "portfolioId": 900,
  "parameters": {
    "startCapital": 5000,
    "monthlyContribution": 100,
    "horizonYears": 15
  }
}
</pre>
Kein <code>Authorization</code>-Header nötig. <code>paths</code> höchstens 10.000; kein <code>save</code> und <code>name</code>.
</td>
<td>
<b>200 OK</b><br>
Body wie bei <code>POST /simulations</code>, <code>id</code> ist immer <code>null</code>:
<pre>
{
  "id": null,
  "portfolioId": 900,
  "parameters": {
    "startCapital": 5000,
    "monthlyContribution": 100,
    "horizonYears": 15,
    "paths": 10000,
    "seed": 731
  },
  "timeline": [
    {
      "month": 0,
      "p5": 5000,
      "p50": 5000,
      "p95": 5000
    },
    {
      "month": 180,
      "p5": 24100.3,
      "p50": 41250.8,
      "p95": 70980.2
    }
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
  "createdAt": "2026-11-10T12:00:00Z"
}
</pre>
</td>
</tr>
</table>