# Installation
Für Node-RED sollte ein eigener User verwendet werden, der zur `sudo`-Gruppe hinzugefügt wird und dessen Passwort auch gesetzt ist:

```bash
$ sudo useradd -d /opt/nodered -m -s /bin/bash nodered
$ sudo usermod -a -G sudo nodered
$ sudo passwd nodered
```

Ab jetzt sollte mit diesem User gearbeitet werden.

Node-RED kann über das Raspbian-Repository mit `apt install ...` installiert werden, aber man erhält dann eine veraltete Version von Node-RED und Node.js. Aus diesem Grund sollte die Installation von Node-RED und node.js entsprechend der [Anleitung auf der Node-RED-Homepage](https://nodered.org/docs/getting-started/raspberrypi) erfolgen, wobei die Frage nach der Installation Pi-spezifischer Nodes bejaht werden sollte:

```bash
nodered@raspberrypi ~ $ bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
Running Node-RED install for user nodered at /opt/nodered on raspbian


We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for nodered: 

This can take 20-30 minutes on the slower Pi versions - please wait.

  Stop Node-RED                       ✔
  Remove old version of Node-RED      ✔
  Remove old version of Node.js       ✔   
  Install Node.js 18 LTS              ✔   v18.18.0   Npm 9.8.1
  Clean npm cache                     ✔
  Install Node-RED core               ✔   3.1.0
  Move global nodes to local          -
  Npm rebuild existing nodes          ✔
  Install extra Pi nodes              ✔
  Add shortcut commands               ✔
  Update systemd script               ✔
                                      

Any errors will be logged to   /var/log/nodered-install.log
All done.
You can now start Node-RED with the command  node-red-start
  or using the icon under   Menu / Programming / Node-RED
Then point your browser to localhost:1880 or http://{your_pi_ip-address}:1880

Started :  Sun Oct  8 15:47:05 CEST 2023 
Finished:  Sun Oct  8 15:50:32 CEST 2023
```

Zum Starten eignet sich folgender Befehl:

```bash
$ sudo systemctl start nodered
```

Damit Node-RED beim Systemstart ebenfalls gestartet wird (via Systemd), muss folgender Befehl ausgeführt werden:

```bash
$ sudo systemctl enable nodered
Created symlink /etc/systemd/system/multi-user.target.wants/nodered.service → /lib/systemd/system/nodered.service.
```

# Installation benötigter Bibliotheken
Folgende Module müssen über `Manage Palette -> Install` installiert werden:
- `node-red-contrib-cron-plus`
- `node-red-contrib-config`
- `node-red-dashboard`
- `node-red-contrib-ui-led`

Einige weitere Bibliotheken müssen manuell in der Shell installiert werden, während man sich im Verzeichnis `~/.node-red` befindet:
- `date-fns`
- `date-fns-tz`

Dazu muss der nachfolgende Befehl für jeden Namen aus der vorangegangenen Liste einmal ausgeführt werden, wobei `<name>` jeweils durch den Listeneintrag ersetzt wird:

```bash
$ npm i <name>
```

Die Bibliothek `date-fns` muss noch in der Datei `~/.node-red/settings.js` eingetragen werden. Dazu in der Datei nach `functionGlobalContext`) suchen und wie folgt ändern:

```javascript
functionGlobalContext: {                                                         
  datefns:require('date-fns'),
  datefnstz:require('date-fns-tz')                                     
},
```

# Zeitzone
Damit die Berechnungen funktionieren, muss die Zeitzone in Node-RED richtig gesetzt sein.
Dazu muss in der Datei `~/.node-red/settings.js` oberhalb von `module.exports = {` eingetragen werden:
```
process.env.TZ = "Europe/Berlin"
```

# Flow-Import
Der Import der Flows in Node-RED erfolgt über das Menü `Import`. Nach Klick in den zentralen, roten Bereich des Import-Dialoges kann dort das [Flow-JSON für Heatpump-Tibber-PV](https://raw.githubusercontent.com/camueller/node-red-contrib-heatpump-tibber-pv/main/flow.json) eingefügt werden. Durch Klicken des `Import`-Buttons werden die Flows in Node-RED importiert. 

# Konfiguration
## Config
Dieser Flow verwendet einen **Config**-Node zur Konfiguration der folgenden **Pflicht-Parameter**:

- `estimatedPowerConsumption`: die geschätzte Leistungsaufnahme der Wärmepumpe in kW (z.B. 4.4)
- `powerConsumptionHours`: die maximale Anzahl der Stunden, in denen die Wärmepumpe täglich laufen  soll (z.B. 8)
- `solcastResourceId`: die von [Solcast](https://solcast.com/) für die Anlage vergebene Resource Id (sieht so aus, wobei die "x" für Ziffern/Buchstaben stehen: xxxx-xxxx-xxxx-xxxx)
- `solcastApiKey`: der bei [Solcast](https://solcast.com/) generierte API key
- `tibberAuthorizationToken`: der bei [Tibber Developer](https://developer.tibber.com/settings/access-token generierte Access Token für die Scopes`tibber_graph`, `homes` und `price`
- `feedInTariff`: die Einspeisevergütung inkl. MwSt. in Euro/KWh (z.B. 0.1231)

 Zusätzlich können folgende **optionale Parameter** gesetzt werden:
- `mandatoryHours`: die Pflichtstunden (als Teil der maximalen Anzahl der Stunden), in denen die Wärmepumpe unabhängig vom Preis laufen soll. Der Wert muss als Array angegeben werden, wobei ein Array-Element
  - als Zahl angegeben wird für tägliche Gültigkeit (z.B. [8] für 8 Uhr)
  - als Array mit 2 Elementen angegeben wird für Gültigkeit mit Wochentagbezug, wobei Sonntag=0, Montag=1, Dienstag=2, Mittwoch=3, Donnerstag=4, Freitag=5, Samstag=6 (z.B. [0,8] für Sonntags 8 Uhr)

## Solid State Relay
Der Node `Solid State Relay` verwendet einen GPIO-Port des Raspberry Pi, um das Solid State Relay zu schalten. Am Node selbst muss der verwendete Pin konfiguriert werden.

# Deploy
Wenn zuvor alle benötigten Bibliotheken installiert wurden, sollten dabei keine Fehler auftreten.
