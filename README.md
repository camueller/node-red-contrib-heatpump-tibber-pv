# Node-RED basierte Steuerung einer Wärmepumpe mit Tibber und PV-Anlage
## Ziel
Steuerung einer Wärmepumpe zur Erzielung geringstmöglicher Stromkosten durch Nutzung des dynamischen Tarifs mit stündlichen Preisen von [Tibber](https://tibber.com/) und einer PV Anlage

## Voraussetzungen
Folgende Vorausstzungen müssen vorliegen:
- steuerbare Wärmepumpe (SG-Ready oder EVU-Signal-Simulation)
- PV-Anlage
- Strombezug durch [Tibber](https://tibber.com/) mit stündlichen Preisen
- Raspberry Pi (mit Anpassungen auch  Server oder NAS) auf dem [Node-RED](https://nodered.org/) läuft

## Funktionsweise
### Überblick
Wenn eine Wärmepumpe (insbesondere solche mit großem Pufferspeicher) nur eine feste, **maximale Anzahl von Stunden täglich** (bei mit 8 Stunden) einschalten darf, ist es sinnvoll, dafür die **Stunden mit den geringstmöglichen Stromkosten** zu wählen.

Dazu müssen die **Stromkosten für jede Stunde eines Tages berechnet** werden wie folgt: Von der **Leistungsaufnahme der Wärmepumpe** wird zunächst die **prognostizierte Leistung der PV-Anlage** abgezogen. Die Energie für diese **verbleibende Leistungsaufnahme** (falls positiv) muss gekauft werden und wird mit dem **zu dieser Stunde gültigen Preis** bewertet. Zu diesen Kosten muss allerdings noch der **Verzicht auf die Einspeisevergütung** für die **prognostizierte Leistung der PV-Anlage** addiert werden, damit die Kosten von Stunden mit PV-Leistung und solchen ohne PV-Leistung vergleichbar sind.

Entsprechend der gewünschten **maximalen Anzahl von täglichen Betriebsstunden der Wärmepumpe** werden die **Stunden mit den geringsten Stromkosten gewählt**. Es lassen sich optional Stunden (auch mit Wochentagbezug) definieren, die unabhängig von de Stromkosten gewählt werden.

Zu Beginn jeder Stunde wird geprüft, ob diese Stunde als kostengünstig klassifiziert wurde. Dementsprechend wird das **Signal zum Ein- oder Ausschalten** an die Wärmepumpe gesendet.

### Prognose der Leistung der PV-Anlage
Zur Prognose der Leistung der PV-Anlage wird der der Dienst [Solcast](https://solcast.com/) verwendet, der diese [kostenlos für Hauseigentümer mit Dachsolaranlagen bereitstellt](https://solcast.com/free-rooftop-solar-forecasting). Dafür muss man sich einen **kostenlosen Account** anlegen, bei dem Angaben zur  Lage, Ausrichtung und Leistung der PV-Anlage gemacht werden müssen. Die für die Anlage vergebende Resource Id sowie der API-Key müssen in der Flow-Configuration angegeben werden.

Die Prognose der PV-Leistung erfolgt mit einer Auflösung von 30 Minuten. Der Flow fasst deshalb die zwei Werte einer Stunde zusammen und verwendet den Durchschnitt als prognostizierte PV-Leistung für diese Stunde.

Bei Verwendung eines kostenlosen Accounts ist die Anzahl der Abfragen der PV-Progrnosen beschränkt auf 10 innerhalb von 24 Stunden. Das ist kein Problem im realen Betrieb, aber möglicherweise beim Testen.

## Flow
Die oben beschriebene Funktionsweise wird durch [folgenden Flow](flow.json) erreicht: 
![Flow](flow.png)

## Dashboard
Die Visualisierung des Flows über das Dashboard zeigt mehrere Diaggramme für den Tag:
![Dashboard](dashboard.png)

Ganz oben wird das Zeitfenster angezeigt, das für die letzte Prüfung auf die kostengünstigesten Stunden verwendet wurde. Das sollte immer die aktuelle Stunde sein. 

Darunter wird der aktuelle Schaltzustand angeigt, d.h. wenn das aktuell Zeitfenster als kostengünstig klassifiziert wurde, sollte ein gründes Licht anzeigen, dass der Schalter eingeschaltet ist.

Im Diagramm *PV-Prognose* wird die prognostizierte, stündliche Leistung der PV-Anlage in kW gezeigt. Unterhalb des Diagramms wird angezeigt, für welches Datum die Werte gelten oder eine Fehlermeldung, falls die Werte nicht abgerufen werden konnten.

Das Diagramm *Strompreis* zeigt den stündlichen Preis für eine kWh an. Unterhalb des Diagramms wird angezeigt, für welches Datum die Werte gelten.

Unter Berücksichtgung 
- der Leistungsaufnahme der Wärmepumpe
- der prognostizierten Leistung der PV-Anlage
- der stündlichen Strompreise
- der Einspeisevergütung
zeigt das Diagramm *Stromkosten* selbige an.

Im Diagramm *Kostengünstigste Stunden* werden die Stunden angezeigt, in die als kostengünstig klassifiziert wurden und in denen die Wärmepumpe einschalten darf.

[Installation und Konfiguration](installation.md)
