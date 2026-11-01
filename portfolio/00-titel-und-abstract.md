# LookUpStock

## Monte-Carlo-Simulation und fairer Vergleich von Anlagen und Portfolios als Web-Service

**Portfolio Webprogrammierung**
Duale Hochschule Baden-Württemberg Mosbach · Wirtschaftsinformatik · Kurs WWI25B · Wintersemester 2026/27
Abgabe: 08.11.2026

**Team LookUpStock**

| Name | Rolle im Projekt | GitHub |
|---|---|---|
| Mert Eminel | Product Owner | KujoBaraka |
| Paul Kaufmann | Entwickler | Paul02112004 |
| Peter Orban | Entwickler, Gruppensprecher | Pixxels02 |
| Carl Stibbe | Scrum Master | carlstibbe |

**Thema.** LookUpStock ist ein Web-Service, mit dem Privatanleger Aktien und ETFs suchen, daraus gewichtete Portfolios zusammenstellen und deren mögliche Entwicklung mit einer Monte-Carlo-Simulation als Bandbreite darstellen lassen. Zwei Anlagen oder Portfolios lassen sich unter identischen Annahmen vergleichen. Der Service besteht aus zwei Backend-Diensten, einem MPA- und einem SPA-Touchpoint, einer REST- und einer GraphQL-Schnittstelle und bindet den Kursdatenanbieter Twelve Data an.

**Projekt-Code und Dienst**

| | Link |
|---|---|
| Frontend (MPA und SPA) | https://github.com/LookUpStock/Look-Up-Stock-FrontEnd |
| Backend (core-service, marktdaten-service) | https://github.com/LookUpStock/Look-Up-Stock-BackEnd |
| Dokumentation (dieses Portfolio, API-Design, ADRs, Protokolle) | https://github.com/LookUpStock/Look-Up-Stock-Docs |
| Web-Service im Internet | https://lookupstockfrontend-96a9998805cb.herokuapp.com |

<div style="page-break-after: always;"></div>

## Abstract (deutsch)

Dieses Portfolio dokumentiert, wie ein vierköpfiges Studierendenteam in fünf einwöchigen Scrum-Sprints (28.09. bis 01.11.2026) den Web-Service LookUpStock konzipiert und umgesetzt hat. LookUpStock projiziert Anlagen und Portfolios mit einer Monte-Carlo-Simulation und vergleicht sie unter gleichen Annahmen. Im Mittelpunkt stehen nicht die Funktionen, sondern die Entscheidungen und ihre Folgen. Das Team begann mit einem monolithischen Backend und einer Anmeldung über den Request-Body und musste beides umbauen, als die Anforderungen an eine zustandslose, REST-konforme Schnittstelle und an eine nicht monolithische Architektur konsequent angewendet wurden. Die Aufteilung in einen core-service und einen marktdaten-service, die asynchrone Kommunikation über eine Queue-Tabelle in PostgreSQL und die Caches für den Fremdservice Twelve Data entstanden deshalb erst im letzten Sprint. Weitere Lernfelder waren die Grenzen eines kostenlosen Datentarifs, Merge-Konflikte in einem verteilten Team und ein Change Request kurz vor Projektende. Die zentrale Erkenntnis lautet: Die Anforderungen aller Beteiligten früh gegen die Architektur zu prüfen, ist deutlich günstiger als ein späterer Umbau.

## Abstract (English)

This portfolio documents how a team of four students designed and implemented the web service LookUpStock in five one-week Scrum sprints (28 September to 1 November 2026). LookUpStock projects assets and portfolios with a Monte Carlo simulation and compares them under identical assumptions. The focus is not on features but on decisions and their consequences. The team started with a monolithic backend and authentication via the request body, and had to rebuild both once the requirements for a stateless, REST-compliant interface and for a non-monolithic architecture were applied consistently. The split into a core service and a market data service, asynchronous communication through a queue table in PostgreSQL, and the caches for the third-party service Twelve Data were therefore only built in the last sprint. Further lessons concerned the limits of a free data plan, merge conflicts in a distributed team, and a change request shortly before the end of the project. The central insight is that checking the requirements of all stakeholders against the architecture early on is considerably cheaper than rebuilding it later.
