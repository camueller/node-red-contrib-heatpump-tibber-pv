# Flow
Der folgende Abbildung des Flows repräsentiert den Inhalt der Datei [flow.json](flow.json) hier im Repository.
![Flow](flow.png)

Der Flow besteht im wesentlichen aus sechs Teilen, wobei Teil 1 bis 4 täglich um 23:30 Uhr ausgeführt werden:

- **Teil 1** beinhaltet die Abfrage der PV-Prognose
  ![Flow1](flow1.png)
- **Teil 2** beinhaltet die Abfrage der Strompreise für den Folgetag
  ![Flow2](flow2.png)
- **Teil 3** berechnet aus diesen Daten die stündlichen Kosten für den Betrieb der Wärmepumpe am Folgetag
  ![Flow3](flow3.png)
- **Teil 4** setzt Standard-Werte für die kostengünstigsten Stunden, für den Fall, dass die Abfrage der PV-Prognose und/oder die Abfrage der Strompreise nicht erfolgreich sind
  ![Flow4](flow4.png)
- **Teil 5** erlaubt das Hinzufügen eines Stundenbereichs zu den bereits als kostengünstig klassifizierten Stunden des aktuellen Tages. Das entspricht der Partyzeit-Funktion bei vielen Heizungssteuerungen. ![Flow5](flow5.png)
- **Teil 6** führt zu jeder vollen Stunde die Prüfung durch, ob die aktuelle Stunde als kostengünstig klassifiziert ist und aktualisiert den Schaltzustand des Solid-State-Relays     ![Flow6](flow6.png)
