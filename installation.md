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
# Node-RED Service-Log in Datei schreiben (optional)
Das Service-Log von Node-RED kann mit dem Befehl `node-red-log` angezeigt werden. Standardmäßig wird es nicht in eine Datei geschrieben.

Zur besseren Nachvollziehbarkeit erzeugt der Flow einige Einträge im Service-Log. Wenn diese in eine Datei geschrieben werden sollen, muss nachfolgende Befehle zur Installation der notwendigen Konfigurationsdateien ausgeführt werden:

```bash
sudo wget https://github.com/camueller/node-red-contrib-heatpump-tibber-pv/raw/master/nodered-logging.service -P /etc/systemd/system
sudo wget https://github.com/camueller/node-red-contrib-heatpump-tibber-pv/raw/master//nodered-logging.conf -P /etc/rsyslog.d
```

Ausserdem muss in der Datei `/etc/systemd/journald.conf` folgende Zeile hinzugefügt werden, damit die Einträge nicht doppelt in der Log-Datei erscheinen:
```
ForwardToSyslog=no
```
Abschliesssend müssen einige Dienste neu gestartet werden:
```bash
sudo systemctl daemon-reload
sudo systemctl enable nodered-logging
sudo systemctl start nodered-logging
sudo systemctl restart rsyslog
```

# Installation benötigter Bibliotheken
Folgende Module müssen über `Manage Palette -> Install` installiert werden:
- `node-red-contrib-cron-plus`
- `node-red-dashboard`
- `node-red-contrib-ui-led`

Einige weitere Bibliotheken müssen manuell in der Shell installiert werden, während man sich im Verzeichnis `~/.node-red` befindet:
- `date-fns@2.30.0`
- `date-fns-tz@2.0.0`

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

# Deploy
Wenn zuvor alle benötigten Bibliotheken installiert wurden, sollten dabei keine Fehler auftreten. Für die fehlerfreie Ausführung des Flows ist zusätzlich die [Konfiguration der Parameter](configuration.md) erforderlich.
