# Flow
Der folgende Abbildung des Flows repräsentiert den Inhalt der Datei [flow.json](flow.json) hier im Repository.
![Flow](flow.png)

Der Flow besteht im wesentlichen aus sechs Teilen, wobei Teil 1 bis 4 täglich direkt nach Mittnacht (00:00:31 Uhr) ausgeführt werden:

## Teil 1
sichert die die kostengünstigsten Stunden des vorangegangenen Tages, bevor diese mit den werte des aktuellen Tages überschrieben werden

![Flow1](flow1.png)

## Teil 2
beinhaltet die Abfrage der PV-Prognose

![Flow2](flow2.png)

## Teil 3
beinhaltet die Abfrage der Strompreise, wobei diese in eine Nachricht mit folgender Payload konvertiert werden (Preise in ct/kWh inkl. aller Gebühren und Steuern):
```json
{
  "2024-06-16T00": 0.2152,
  "2024-06-16T01": 0.1979,
  "2024-06-16T02": 0.1914,
  "2024-06-16T03": 0.189,
  "2024-06-16T04": 0.1881,
  "2024-06-16T05": 0.1885,
  "2024-06-16T06": 0.189,
  "2024-06-16T07": 0.1864,
  "2024-06-16T08": 0.1855,
  "2024-06-16T09": 0.184,
  "2024-06-16T10": 0.1736,
  "2024-06-16T11": 0.1697,
  "2024-06-16T12": 0.1617,
  "2024-06-16T13": 0.1498,
  "2024-06-16T14": 0.1431,
  "2024-06-16T15": 0.151,
  "2024-06-16T16": 0.1735,
  "2024-06-16T17": 0.1827,
  "2024-06-16T18": 0.2524,
  "2024-06-16T19": 0.3041,
  "2024-06-16T20": 0.3273,
  "2024-06-16T21": 0.3334,
  "2024-06-16T22": 0.3287,
  "2024-06-16T23": 0.2979
}
```

![Flow3](flow3.png)

## Teil 4
berechnet aus diesen Daten die stündlichen Kosten für den Betrieb der Wärmepumpe und bestimmt die kostengünstigsten Stunden

![Flow4](flow4.png)

## Teil 5
setzt Standard-Werte für die kostengünstigsten Stunden, für den Fall, dass die Abfrage der PV-Prognose und/oder die Abfrage der Strompreise nicht erfolgreich sind

![Flow5](flow5.png)

## Teil 6
erlaubt das Hinzufügen eines Stundenbereichs zu den bereits als kostengünstig klassifizierten Stunden. Das entspricht der Partyzeit-Funktion bei vielen Heizungssteuerungen.

![Flow6](flow6.png)

## Teil 7
führt zu jeder vollen Stunde die Prüfung durch, ob die aktuelle Stunde als kostengünstig klassifiziert ist und aktualisiert den Schaltzustand des Solid-State-Relays

![Flow7](flow7.png)
