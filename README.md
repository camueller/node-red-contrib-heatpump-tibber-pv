# Node-RED basierte Steuerung einer Wärmepumpe mit Tibber und PV-Anlage
## Ziel
Steuerung einer Wärmepumpe zur Erzielung geringstmöglicher Stromkosten durch Nutzung des dynamischen Tarifs mit stündlichen Preisen von [Tibber](https://tibber.com/) und einer PV Anlage.

Ich habe zu diesem Thema auch ein Video erstellt, welches die Zusammenhänge und die praktische Umsetzung verständlich machen sollte:
<a href="https://www.youtube.com/watch?v=sGIVslsUMZ4"><img align="left" src="youtube.png"></a>
<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>

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

## Flow und Dashboard 
<a><img align="left" src="flow_thumbnail.png"></a>
Die oben beschriebene Funktionsweise wurde implementiert durch [diesen Flow](flow.md).

Um das Verhalten der Steuerung transparent zu machen, beinhaltet der Flow auch die [Visualisierung der Daten in einem Dashboard](dashboard.md).

## Installation und Konfiguration
<a><img align="left" src="install.png"></a>
Zur Nutzung des Flows inst die [Installation von Node-RED und einigen Bibliotheken erforderlich](installation.md).

Damit der Flow tatsächlich funktioniert ist die [Konfiguration einiger Werte](configuration.md) erforderlich.

## Unterstützung
<a href="doc/Support_DE.md"><img align="left" src="support.jpeg"></a>
Diskussionen und Fragen rund um den *Smart Appliance Enabler* sind in [Github Diskussion](https://github.com/camueller/node-red-contrib-heatpump-tibber-pv/discussions) richtig platziert.

Darüber hinaus biete ich einen [persönlichen, individuellen, kostenpflichtigen Support mit Beratung](Support.md) an - quasi das "Rundum-sorglos Paket".
<br><br><br>