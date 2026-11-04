# 6 Reflexion Carl Stibbe

*Docs #178 · Carl Stibbe (carlstibbe)*

Meine Aufgaben lagen vor allem im Frontend und bei der Bereitstellung der Anwendung. Ich habe an der SPA mit clientseitigem Routing (FE #53), am Dashboard mit Monatsauswahl und rollenabhängiger Navigation (FE #51) sowie an Responsive-Fixes gearbeitet. In Sprint 6 kamen Fehlerbehebungen an der Navigation und am Dashboard dazu (FE #67 und FE #70). Außerdem habe ich die Heroku-Apps konfiguriert und den übrigen Bereitstellungsaufbau weitgehend allein betreut. Die Einrichtung der Datenbank hat Paul übernommen. Zu meinen Aufgaben gehörten unter anderem die getrennten Apps für core-service und marktdaten-service sowie deren Konfiguration. Als Scrum Master habe ich zusätzlich auf die Abläufe und die Nachvollziehbarkeit der Aufgaben geachtet.

## Was ich gelernt habe

Mein wichtigster Lernschritt lag im Frontend und bei der SPA. Eine Seite ist nicht fertig, nur weil sie lokal richtig aussieht: Navigation, Rollenprüfung, API-Aufrufe und Deployment müssen zusammenpassen. Bei der SPA wurde das besonders deutlich. Die MPA sollte erhalten bleiben, während die neue Einstiegsseite Ansichten ohne vollständiges Neuladen wechselt. Dafür musste ich die Vorgaben aus ADR-08 und dem Sprint-Planning mit dem vorhandenen Frontend und den Schnittstellen abgleichen. Auch bei kleineren Responsive-Fixes habe ich gemerkt, dass eine Änderung bei 360 Pixeln gezielt geprüft werden muss und nicht automatisch aus der Desktopansicht folgt.

Beim Betrieb auf Heroku habe ich gelernt, wie stark Konfiguration und Reihenfolge den Erfolg eines Deployments beeinflussen. Frontend, core-service und marktdaten-service sind getrennte Apps; zugleich greifen die Backend-Dienste auf dieselbe Datenbank zu. Damit wurde für mich praktisch sichtbar, dass ein erfolgreicher Build allein noch nicht belegt, dass die Anwendung nach dem Start funktioniert. Zu einem Deploy gehören auch die Prüfung der Config Vars, der Build-Logs und der Health-Endpunkte. Die Apps und ihre Bereitstellung weitgehend selbst zu betreuen, hat mir einen besseren Blick für das Gesamtsystem gegeben.

## Rückschläge und Zusammenarbeit

Ein wiederkehrendes Problem war, dass einzelne Issues nur einen Titel, aber keine Beschreibung enthielten. Bei größeren Frontend-Aufgaben musste ich deshalb Anforderungen aus Sprint-Planning-Protokollen, ADRs und Testdokumenten zusammentragen, bevor ich sicher wusste, was umgesetzt werden sollte. Das kostete Zeit und machte deutlich, dass eine gute technische Lösung mit einer klaren Aufgabenbeschreibung beginnt. Für künftige Sprints würde ich Akzeptanzkriterien und Abhängigkeiten direkt im Issue festhalten und früh prüfen, ob die beteiligten Repositories denselben Stand haben.

Auch die Arbeit mit Branches und Pull Requests hat mir gezeigt, wie wichtig eine saubere Trennung der Aufgaben ist. Änderungen an Navigation oder Routing können mehrere Seiten betreffen; deshalb helfen kleine, issuebezogene Branches und ein Abgleich mit `main`, bevor ein Fix weitergegeben wird. Beim Heroku-Betrieb gilt Ähnliches: Änderungen müssen mit den Merges im Frontend und Backend abgestimmt werden, damit ein Nachtest wirklich den erwarteten Stand prüft.

## Arbeiten mit KI

KI-Unterstützung hat mir beim Einordnen von Anforderungen, beim Entwurf von Änderungen und beim Prüfen möglicher Ursachen geholfen. Gleichzeitig habe ich gelernt, Vorschläge nicht als Anforderung zu behandeln. Gerade bei leeren Issues kann ein Modell fehlenden Kontext plausibel ergänzen, obwohl die tatsächliche Vorgabe in einem anderen Dokument steht. Deshalb habe ich Ergebnisse mit den Projektunterlagen, dem vorhandenen Code und den Abnahmetestfällen abgeglichen. Für mich ist KI damit ein hilfreiches Werkzeug, wenn ich die Verantwortung für Prüfung und Entscheidung selbst behalte.

## Bewertung des Umfangs

Rückblickend war der Umfang für die kurze Bearbeitungszeit sehr groß und zeitweise stressig. Neben den Aufgaben aus der Fallstudie kamen mit der SPA, dem Dashboard, den Responsive-Anforderungen und der Bereitstellung auf Heroku weitere technische Themen hinzu. Gerade im Frontend mussten Navigation, Rollen, API-Aufrufe und bestehende MPA-Seiten zusammenpassen. Parallel dazu liefen Abstimmung, Reviews und Dokumentation weiter. Ich hätte den Aufwand dafür zu Beginn stärker unterschätzt.

Ohne KI-Unterstützung wäre es für mich deutlich schwieriger gewesen, die vielen Anforderungen in der verfügbaren Zeit zu verstehen und umzusetzen. Sie half mir, technische Zusammenhänge schneller zu erschließen, Lösungswege zu vergleichen und bei Fehlern gezielter weiterzusuchen. Trotzdem musste ich die Vorschläge mit dem Code, den ADRs und den Abnahmekriterien abgleichen; gerade bei unvollständigen Issues durfte ich fehlende Vorgaben nicht einfach annehmen. Rückblickend waren die Kombination aus KI als Unterstützung, kleinen Arbeitsschritten und der Prüfung am laufenden System entscheidend, um die Aufgaben im gesetzten Zeitrahmen zu bewältigen.
