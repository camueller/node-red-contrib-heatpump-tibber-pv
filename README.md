# Node-RED basierte Steuerung einer Wärmepumpe mit Tibber und PV-Anlage
## Ziel
Steuerung einer Wärmepumpe zur Erzielung geringstmöglicher Stromkosten durch Nutzung des dynamischen Tarifs mit stündlichen Preisen von [Tibber](https://tibber.com/) und einer PV Anlage

## Voraussetzungen
Folgende Vorausstzungen müssen vorliegen:
- Wärmepumpe, die über EVU-Sperre gesteuert werden kann
- PV-Anlage
- Strombezug durch [Tibber](https://tibber.com/) mit stündlichen Preisen
- Raspberry Pi (mit Anpassungen auch  Server oder NAS) auf dem [Node-RED](https://nodered.org/) läuft

## Funktionsweise
### Überblick
Wenn eine Wärmepumpe (insbesondere solche mit großem Pufferspeicher) nur eine feste, **maximale Anzahl von Stunden täglich** (bei mit 8 Stunden) einschalten darf, ist es sinnvoll, dafür die **Stunden mit den geringstmöglichen Stromkosten** zu wählen.

Dazu müssen die **Stromkosten für jede Stunde eines Tages berechnet** werden wie folgt:

Von der **Leistungsaufnahme der Wärmepumpe** wird zunächst die **prognostizierte Leistung der PV-Anlage** abgezogen. Die Energie für diese **verbleibende Leistungsaufnahme** (falls positiv) muss gekauft werden und wird mit dem **zu dieser Stunde gültigen Preis** bewertet. Zu diesen Kosten muss allerdings noch der **Verzicht auf die Einspeisevergütung** für die **prognostizierte Leistung der PV-Anlage** addiert werden, damit die Kosten von Stunden mit PV-Leistung und solchen ohne PV-Leistung vergleichbar sind.

Entsprechend der gewünschten **maximalen Anzahl von täglichen Betriebsstunden der Wärmepumpe** werden die **Stunden mit den geringsten Stromkosten gewählt**. Es lassen sich **optional Pflichtstunden** (auch mit Wochentagbezug) definieren, die unabhängig von den Stromkosten gewählt werden.

**Zu Beginn jeder Stunde** wird geprüft, ob diese Stunde als kostengünstig klassifiziert wurde. Dementsprechend wird das **Signal zum Ein- oder Ausschalten** an die Wärmepumpe gesendet.

### Wärmepumpe
Normalerweise dient die **EVU-Sperre** dazu, dass der Energieversorger die Wärmepumpe zu Spitzenlastzeiten (in der Regel maximal 3x täglich für 2 Stunden) abschalten kann. Dazu ist in der Wärmepumpe ein Relais verbaut, dessen Kontakte geöffnet werden, wenn die Wärempumpe nicht laufen soll.

Bei der **Steuerung über Node-RED** darf dieses Relais natürlich nicht mehr mit dem Rundsteuerempfänger des Energieversogers verbunden sein. Stattdessen wird der Leiter dieses Relais durch ein **zusätzliches Solid-State-Relais** geschaltet. Das Solid-State-Relais wiederum wird mit einem GPIO-Pin des Raspberry Pi verbunden und durch Node-RED gesteuert. Wenn das Solid-State-Relais aus ist, ist auch das EVU-Sperre-Relais der Wärmepumpe aus und die EVU-Sperre ist aktiv.

Die **grundsätzliche Idee dieser Steuerung** besteht darin, die EVU-Sperre immer aktiv zu haben außer in den Stunden, die als kostengünstig klassifiziert wurden. Damit die Wärmepumpe läuft, wenn die EVU-Sperre nicht aktiv ist, müssen die Schaltzeiten in der Wärmepumpensteuerung so angepasst werden, dass sie 24 Stunden täglich einschalten würde.

### Prognose der Leistung der PV-Anlage
Zur Prognose der Leistung der PV-Anlage wird der der Dienst [Solcast](https://solcast.com/) verwendet, der diese [kostenlos für Hauseigentümer mit Dachsolaranlagen bereitstellt](https://solcast.com/free-rooftop-solar-forecasting). Dafür muss man sich einen **kostenlosen Account** anlegen, bei dem Angaben zur  Lage, Ausrichtung und Leistung der PV-Anlage gemacht werden müssen. Die für die Anlage vergebende Resource Id sowie der API-Key müssen in der Flow-Configuration angegeben werden.

Die Prognose der PV-Leistung erfolgt mit einer Auflösung von 30 Minuten. Der Flow fasst deshalb die zwei Werte einer Stunde zusammen und verwendet den Durchschnitt als prognostizierte PV-Leistung für diese Stunde.

Bei Verwendung eines kostenlosen Accounts ist die Anzahl der Abfragen der PV-Progrnosen beschränkt auf 10 innerhalb von 24 Stunden. Das ist kein Problem im realen Betrieb, aber möglicherweise beim Testen.

## Flow
Die oben beschriebene Funktionsweise wird durch [folgenden Flow](flow.json) erreicht: 
![Flow](flow.png)

Der Flow besteht im wesentlichen aus drei Teilen
1. Abfrage der PV-Prognose - wird täglich um 23:30 Uhr ausgeführt
1. Abfrage der Strompreise für den Folgetag - wird täglich um 23:30 Uhr ausgeführt
1. Prüfung, ob die aktuelle Stunde als kostengünstig klassifiziert ist, und Aktualisierung des Schaltzustandes - wird zu jeder vollen Stunde ausgeführt

## Dashboard
Die Visualisierung des Flows über das Dashboard zeigt mehrere Diagramme für den Tag:
![Dashboard](dashboard.png)

Ganz oben wird das **Zeitfenster** angezeigt, das für die letzte Prüfung auf die kostengünstigesten Stunden verwendet wurde. Das sollte immer die aktuelle Stunde sein. 

Darunter wird der aktuelle **Schaltzustand** angeigt, d.h. wenn das aktuell Zeitfenster als kostengünstig klassifiziert wurde, sollte ein grünes Licht anzeigen, dass der Schalter eingeschaltet ist bzw. ein rotes Licht, dass der Schalter ausgeschaltet ist.

Im Diagramm **PV-Prognose** wird die prognostizierte, stündliche Leistung der PV-Anlage in kW gezeigt. Unterhalb des Diagramms wird angezeigt, für welches Datum die Werte gelten oder eine Fehlermeldung, falls die Werte nicht abgerufen werden konnten.

Das Diagramm **Strompreis** zeigt den stündlichen Preis für eine kWh an. Unterhalb des Diagramms wird angezeigt, für welches Datum die Werte gelten oder eine Fehlermeldung, falls die Werte nicht abgerufen werden konnten.

Unter Berücksichtgung 
- der Leistungsaufnahme der Wärmepumpe
- der prognostizierten Leistung der PV-Anlage
- der stündlichen Strompreise
- der Einspeisevergütung

zeigt das Diagramm die stündlichen **Stromkosten** entsprechend der konfigurierten *Leistungsaufnahme der Wärmepumpe* an.

Im Diagramm **Kostengünstigste Stunden** werden entsprechend der *maximalen Anzahl von täglichen Betriebsstunden* diejenigen Stunden angezeigt, die als kostengünstig klassifiziert wurden oder Pflichtstunden sind. In diesen Stunden darf die Wärmepumpe einschalten. Im Bild sind zwar die Kosten für die Zeit ab 20 Uhr (83.7 Cent) geringer als um 8 Uhr (89.5 Cent), aber 8 Uhr ist Samstags bei mir als Pflichtstunde konfiguriert.

[Installation und Konfiguration](installation.md)
