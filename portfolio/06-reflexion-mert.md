# 6 Reflexion Mert Eminel

*Docs #178 · Mert Eminel (KujoBaraka)*

Ich war im Projekt Product Owner und zugleich Kunde. Damit lagen die Anforderungen bei mir: die Kurzformat-Tabellen mit den Akzeptanzkriterien und den PO-Festlegungen E1 bis E43, die Priorisierung im Product Backlog, die Abnahmetests und Abnahmeprotokolle jedes Sprints und der Umgang mit dem Change Request CR-01. Im Frontend habe ich die Anlagensuche, die Anmeldemaske, die Beispielsimulation für Gäste und den CSV-Export umgesetzt und mehrere Befunde aus den Abnahmen behoben. Für mich war es das erste Projekt, in dem ich nicht nur mitgearbeitet, sondern entschieden habe, was gebaut wird und wann etwas fertig ist.

## Was ich gelernt habe

### Anforderungen müssen prüfbar sein

Meine ersten Akzeptanzkriterien klangen vollständig, waren es aber nicht. In den Abnahmetestfällen von Sprint 1 stand bei der Simulation noch „[OFFEN: Definition von ‚3 Jahre Kurshistorie‘]“. Erst als ein Testfall geschrieben werden musste, wurde klar, dass niemand wusste, ob drei Kalenderjahre, 36 Monatswerte oder etwas anderes gemeint war. Daraus wurde die Festlegung E8: genau 36 Monatswerte. So sind nach und nach über 40 Festlegungen entstanden, meist mit wörtlichen Meldungstexten und genauen Grenzwerten. Ich habe gelernt, dass ein Kriterium erst dann fertig ist, wenn man daraus einen Testfall mit einem eindeutigen erwarteten Ergebnis ableiten kann.

### Die Abnahme ist eine ehrliche Rückmeldung

In Sprint 2 wurden von 16 Testfällen der Kern-Story US-04 nur 7 bestanden, 7 konnten noch gar nicht durchgeführt werden. Das war unangenehm, aber genau richtig: Das Protokoll zeigte dem Team schwarz auf weiß, was fehlte, und in Sprint 3 waren alle 16 bestanden. Ich habe gelernt, eine Story auch dann nicht abzunehmen, wenn sie fast fertig ist, und das sachlich mit dem verletzten Kriterium zu begründen statt mit einem Gefühl.

### Entscheiden heißt auch weglassen

Die Ereigniskarte zum Reporting für die Geschäftsführung kam am 23.10.2026, mitten im Projekt. Die vollständige Umsetzung hätte mehr als die Hälfte der Kapazität von Sprint 5 gekostet. Ich habe mich für eine teilweise Umsetzung entschieden: Dashboard und CSV-Export ja, automatischer Monatsbericht nein, dafür wurde der Rest von „Vergleiche speichern“ gestrichen. Entscheidend war ein Gedanke, der mir vorher nicht bewusst war: Nutzungsdaten, die nicht erfasst werden, lassen sich später nicht nachholen. Verschieben wäre deshalb teurer gewesen als eine kleinere Lösung sofort.

## Rückschläge

Als Product Owner habe ich Funktionen abgenommen, an denen ich selbst mitgeschrieben hatte. Das ist in einem kleinen Team kaum zu vermeiden, macht aber blind für die eigenen Annahmen. Deutlich wurde mir das beim Bedienbarkeitstest NF-09, den Carl durchgeführt hat: Zwei von drei Personen wussten nach dem Speichern eines Portfolios nicht, wie sie zur Simulation kommen. Diesen Weg hatte keine meiner Anforderungen beschrieben, weil er für mich selbstverständlich war.

Der zweite Rückschlag betrifft die Dokumentation. In der Abschlussprüfung von Sprint 6 zeigte sich, dass einige Dokumente den Code nicht mehr abbildeten: Diagramme zeigten noch einen einzigen Dienst, Testklassen trugen ihre geplanten statt ihrer echten Namen, und Fehlertexte im API-Design wichen vom Code ab. Keine dieser Abweichungen war falsch gemeint; sie sind entstanden, weil Dokumente in einem eigenen Pull Request nachgezogen wurden statt im selben wie der Code.

## Arbeiten mit KI

Ich habe KI vor allem für Entwürfe genutzt: Testfälle, Protokolle, Kurzformat-Tabellen und Teile des Frontend-Codes. Das hat viel Schreibarbeit gespart. Gleichzeitig formuliert die KI Meldungstexte und Namen, die plausibel klingen, aber nicht dem Code entsprechen. Ich habe mir deshalb angewöhnt, jede Angabe im Dokument gegen den Code zu prüfen. Die Abnahmen selbst habe ich immer selbst im Browser durchgeführt, und jede Entscheidung als Product Owner ist meine geblieben.

## Was gut funktioniert hat

- Die PO-Festlegungen mit Nummern machten Entscheidungen nachvollziehbar; im Code, in Tests und in Diskussionen konnte man sich auf „E37“ beziehen statt auf eine Erinnerung.
- Die Abnahmeprotokolle haben den Fortschritt messbar gemacht und Fehler mit Priorität direkt in Issues übersetzt.
- Der Change Request wurde nach einem festen Schema bewertet, mit Optionen, Aufwand und Begründung, statt aus dem Bauch heraus.

## Was ich beim nächsten Mal anders machen würde

1. Akzeptanzkriterien direkt mit Grenzwerten und Meldungstexten formulieren, nicht erst beim Schreiben der Tests.
2. Früh mit echten Nutzern testen, nicht erst kurz vor Schluss.
3. Dokumentation im selben Pull Request wie den Code ändern.

## Fazit

Die Rolle des Product Owners war für mich anfangs vor allem Verantwortung für Dokumente. Am Ende habe ich sie als Verantwortung für Klarheit verstanden: Je genauer eine Anforderung formuliert war, desto weniger Rückfragen gab es und desto schneller ging die Abnahme. Dass aus einer Idee zu Monte-Carlo-Simulationen ein laufender Web-Service mit zwei Diensten, MPA und SPA und einem Dashboard für die Geschäftsführung geworden ist, ist für mich das schönste Ergebnis. Ich nehme aus dem Projekt mit, wie viel ein Team erreicht, wenn jeder sich auf klare Absprachen verlassen kann.
