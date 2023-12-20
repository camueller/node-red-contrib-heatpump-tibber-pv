# Konfiguration
## Config
Dieser Flow verwendet einen **Config**-Node zur Konfiguration der folgenden **Pflicht-Parameter**:

- `estimatedPowerConsumption`: die geschätzte Leistungsaufnahme der Wärmepumpe in kW (z.B. 4.4)
- `powerConsumptionHours`: die maximale Anzahl der Stunden, in denen die Wärmepumpe täglich laufen  soll (z.B. 8)
- `solcastResourceId`: die von [Solcast](https://solcast.com/) für die Anlage vergebene Resource Id (sieht so aus, wobei die "x" für Ziffern/Buchstaben stehen: xxxx-xxxx-xxxx-xxxx)
- `solcastApiKey`: der bei [Solcast](https://solcast.com/) generierte API key
- `tibberAuthorizationToken`: der bei [Tibber Developer](https://developer.tibber.com/settings/access-token) generierte Access Token für die Scopes`tibber_graph`, `homes` und `price`
- `feedInTariff`: die Einspeisevergütung inkl. MwSt. in Euro/KWh (z.B. 0.1231)

Zusätzlich können folgende **optionale Parameter** gesetzt werden:
- `mandatoryHours`: die Pflichtstunden (als Teil der maximalen Anzahl der Stunden), in denen die Wärmepumpe unabhängig vom Preis laufen soll. Der Wert muss als Array angegeben werden, wobei ein Array-Element
    - als Zahl angegeben wird für tägliche Gültigkeit (z.B. `[8]` für 8 Uhr)
    - als Array für Gültigkeit mit Wochentagbezug, wobei Sonntag=0, Montag=1, Dienstag=2, Mittwoch=3, Donnerstag=4, Freitag=5, Samstag=6
        - 2 Elemente (z.B. `["1", 4]` für Montags 4 Uhr)
            - erstes Element: konkreter Wochentag oder Wochentagsbereich
            - zweites Element: konkrete Stunde oder Stundenbereich
        - 2 Elemente mit erstem Element als Wochentagsbereich
        - 3 Elemente (z.B. `["1-5", "1-4", 2]` für Montag bis Freitag mindestens 2 Stunden zwischen 1:00 Unr und 4:59 Uhr)
            - erstes Element konkreter Wochentag oder Wochentagsbereich
            - zweites Element: Stundenbereich
            - drittes Element: Anzahhl der Stunden innerhalb des Stundenbereichs, wobei die kostengünstigsten Stunden gewählt werden
        - 4 Elemente (z.B: `["1-5", "1-6", 3, [2, 4]]` für Montag bis Freitag mindestens 3 Stunden zwischen 1:00 Unr und 6:59 Uhr, wobei die Stunden ab 2:00 Uhr und 4:00 Uhr auf jeden Fall enthalten sind)
            - erstes Element konkreter Wochentag oder Wochentagsbereich
            - zweites Element: Stundenbereich
            - drittes Element: Anzahhl der Stunden innerhalb des Stundenbereichs, wobei die kostengünstigsten Stunden gewählt werden
            - viertes Element: Pflichtstunden innerhalb des Stundenbereichs
- `fallbackHours`: falls Strome-Preise oder PV-Prognose nicht abgerufen werden können, wird die Wärmepumpe in den mit `fallbackHours` identifizierten Stunden laufen. Die Syntax des Wertes ist identisch mit `mandatoryHours`, wobei hier aber nur eine Zahl oder ein Array mit 2 Elemente angegeben werden können.
- `pvForecastFactor`: wird mit der prognostizierten PV-Leistung multipliziert als Korrekturfaktor, wenn die Prognosen regelmäßig abweichen
