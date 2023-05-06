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

Dazu müssen am Ende eines Tages die **Stromkosten für jede Stunde des Folgetages berechnet** werden. Von der Leistungsaufnahme der Wärmepumpe wird zunächst die **prognostizierte Leistung der PV-Anlage** abgezogen. Die **verbleibende Leistungsaufnahme** (falls positiv) wird mit dem **zu dieser Stunde gültigen Preis** bewertet. Entsprechend der gewünschten **maximalen Anzahl von täglichen Betriebsstunden der Wärmepumpe** werden die **Stunden mit den geringsten Stromkosten gewählt**. Zu Beginn jeder Stunde wird das **Signal zum Ein- oder Ausschalten** an die Wärmepumpe gesendet.  

### Prognose der Leistung der PV-Anlage
Zur Prognose der Leistung der PV-Anlage wird der der Dienst [Solcast](https://solcast.com/) verwendet, der diese [kostenlos für Hauseigentümer mit Dachsolaranlagen bereitstellt](https://solcast.com/free-rooftop-solar-forecasting). Dafür muss man sich einen **kostenlosen Account** anlegen, bei dem Angaben zur  Lage, Ausrichtung und Leistung der PV-Anlage gemacht werden müssen. Die für die Anlage vergebende Resource Id muss in der Flow-Configuration angegeben werden.

Die Prognose der PV-Leistung erfolgt mit einer Auflösung von 30 Minuten. Der Flow fasst deshalb die zwei Werte einer Stunde zusammen und verwendet den Durchschnitt als prognostizierte PV-Leistung für diese Stunde.

Bei Verwendung eines kostenlosen Accounts ist die Anzahl der Abfragen der PV-Progrnosen beschränkt auf 10 innerhalb von 24 Stunden. Das ist kein Problem im realen Betrieb, aber möglicherweise beim Testen.

## Flow
Die oben beschriebene Funktionsweise wird durch [folgenden Flow](flow.json) erreicht: 
![Flow](flow.png)

## Dashboard
Die Visualisierung des Flows über das Dashboard zeigt mehrere Diaggramme für den Tag:
![Dashboard](dashboard.png)

In *PV Forecast* wird die prognostizierte, stündliche Leistung der PV-Anlage in kW gezeigt.

Das Diagramm *Hourly energy price** zeigt den stündlichen Preis für eine kWh.

Unter Berücksichtgung der Leistungsaufnahme der Wärmepumpe, der prognostizierten Leistung der PV-Anlage und der stündlichen Strompreise zeigt das Diagramm *Hourly cost* die Stromkosten.

Im Diagramm *Most cost efficient hours* werden die Stunden angezeigt, in denen die Wärmepumpe einschalten darf, damit die geringsten Stunden genutzt werden.

Oberhalb der Diagramme wird das **Datum angezeigt**, für das die Stunden mit den geringsten Stromkosten gewählt wurden. Es sollte immen mit dem aktuellen Datum übereinstimmen.

## Installation
... folgt noch

## Konfiguration
... folgt noch
