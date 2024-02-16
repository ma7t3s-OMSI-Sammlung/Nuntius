# Nuntius
## Einleitung
Nuntius ist ein Fahrgastinformationsystem für OMSI, es soll neben einem eigenen Fahrscheindrucker auch verschiedene Innenanzeigen und später ggf. auch Matrizen umfassen.

Das besondere ist, dass die einzelnen Komponenten besonders gut miteinander versetzt sind, aber auch gleichzeitig für sich eigenständig sind und nur über eine Art "Schnittstelle" mit "der Außenwelt" verbunden werden.
So lässt sich das Nuntius-System besonders einfach in jeden Bus einbauen!

**<ins>Achtung:</ins> Das ganze ist noch Beta, Fehler etc. können immer Auftreten, diese bitte gerne Issues melden, genau so, wie Verbesserungsvorschläge! :)**

---

## Einbau
Grundsätzlich am besten vorher einen alten, ggf. vorhandenen Drucker vollständig ausbauen, um Konflikte zu vermeiden :)
An sich lässt sich das ganze Relativ einfach einbauen, bereitgestellt werden die notwendigen model(mesh)-Dateien, Texturen, Sounds und Scripts direkt in der passenden Ordnerstruktur, einfach in den Bus der Wahl kopieren.
Das Mesh muss natürlich ggf. mithilfe einer Daueranimation oder Blender an die richtige Stelle bewegt werden.

als "model (Ausschnit).cfg" liegt ein Model.cfg-Abschnitt bei, den ihr euch in die model.cfg eures Busses einfügen müsst, Texttextur-Einträge müssen natürlich ggf. angepasst werden, ansonsten sollte der bereits so, angezeigt werden.

Damit er auch das tut, was er soll, brauchen wir noch die Macro-Auswführungen, dazu müsst ihr einmal in die main-Datei des Busses gehen (die sollte einen frame- und init-Abschnitt haben) und dort jeweils folgendes hinzufügen:
- `(M.L.nuntius_printer_init)` unter `{init}`
- `(M.L.nuntius_printer_frame)` unter `{frame}`

Dann noch in der *.bus-Datei folgendes hinzufügen:

**unter Varlist:**
- script\nuntius\varlist.txt

**unter Strinvarlist:**
- script\nuntius\stringvarlist.txt

**unter Script:**
- script\nuntius\script_main.osc
- script\nuntius\script_api.osc

**unter Constfile:**
- script\nuntius\constfile.txt

Zuletzt braucht ihr noch die Ubuntu-Font für die Benutzeroberfläche des Druckers, dazu einfach das, was im Ordner "Fonts" bereitgestellt wird, entsprechend in den OMSI-Font-Ordner ziehen.

Damit ist das ganze soweit fertig, noch eine kleine Anmerkung zum Script:
Wie bereits vielleicht aufgefallen ist, werden zwei Script-Dateien eingebunden und mitgeliefert, eine main- und eine api-Script.
Das Main-Script ist das Hauptscript, es enthält sämtlichen eigentlichen Code, der definiert, wie das ganze System arbeitet. Dieses Script arbeitet **ausschließlich** mit eigenen Variablen und Macros (bzw. den vom OMSI-System). Es ist also 100% universell und sollte daher eigentlich nicht pro Bus angepasst werden.
Die spezifische Abstimmung auf den Bus un den Bus-spezifischen Variablen erfolgt ausschließlich im api-Script (API = application programming interface). Dies ist also die Schnittstelle, über die Nuntius ausschließlich mit dem Bus kommuniziert. Hier gibt es einige Macros, über die der Bus bestimmte Daten an das System übergeben kann (bzw. sollte), damit dieses Ordnungsgemäß funktioniert. Das funktioniert idR. so, dass am Ende des Macros auf dem obersten (bzw. den obersten) Stackplätzen jeweils die zu übermittelnden Werte stehen müssen. Aktuell gibt es da nur zwei Funktionen:

|Name|Beschreibung|
|---|---|
|`nuntius_api_power`|Dient dazu, dem System mitzuteilen, ob es gerade mit Strom versurgt sein soll, sogesehen also einfach der "Stromanschluss", steht am ende des Makros eine 1, heißt das "Strom ist an", steht am Ende eine 0, heißt das "Strom ist aus"|
|`nuntius_api_doorsOpen`|Teilt dem Script mit, ob gerade mindestens eine Tür des Busses geöffnet ist. (1 = Ja, 0 = Nein). Dies dient dazu, dass der Drucker beim öffnen der Türen automatisch in den Fahrscheinverkauf-Modus springen kann.Standardmäßig steht hier ein Code drinne, der die OMSI-globalen "PAX"-Variablen verwendet, der somit bereits für jeden Bus funktionieren sollte, soll das ganze spezifischer sein, kann man natürlich eine eigene Schnittstelle programmieren|

Somit können nun Daten an das System geschickt werden, mindestens genau so wichtig ist, aber, Daten vom Nuntius-System auszulesen, bzw. abzufragen, bswp. für eine Innenanzeige, etc. auch daran ist gedacht und es wird eine Reihe spezieller "API"-Variablen bereitgestellt, die extra dafür da sind, ausgelesen zu werden und verschiedene Informationen bereitstellen. Aktuell ist die Auswahl noch recht klein, soll aber noch umfassend erweitert werden. Grundsätzlich kann natürlich jede "interne" Nuntius-Variable auch ohne Probleme ausgelesen und verwendet werden. Eine genaue Dokumentation aller Markos und Variablen befindet sich in "Dokumentation.xlsx"

### Zahlenvariablen:
|Name|Beschreibung|
|---|---|
|`nuntius_api_lineNo`|Enthält die aktulle "reine" Liniennummer, ohne Suffix etc; Funktioniert nur im Hofdatei-Modus (bzw. aktuell eh scheinbar noch garnicht :D)|
|`nuntius_api_lineSuffix`|Funktionert ebenfalls nur im Hofdatei-Modus und enthält nur den Suffixcode (bswp. 10 für ein nachgestelltes "E", dürfte aktuell ebenfalls noch garnicht funktionieren|
|`nuntius_api_terminusIndex`|Enthält den Hofdatei-Index des aktuellen Ziels|

### Stringvariablen:
|Name|Beschreibung|
|---|---|
|`nuntius_api_timeStr`|Enthält die aktuelle Uhrzeit als String (bswp. für eine Innenanzeige)|
|`nuntius_api_busstop0`|Enthält den Namen der unmittelbar nächsten Bushaltestelle (bswp. für eine Innenanzeige)|
|`nuntius_api_busstop1`|Enthält den Namen der übernächsten Bushaltestelle|
|...|...|
|`nuntius_api_busstop9`|Analog zu oben|
|`nuntius_api_lineStr`|Enthält den aktuellen vollständigen Linienstring (z.B. "76E"), bzw. im Hofdatei-Modus nur die Liniennummer als String formatiert|
|`nuntius_api_terminusStr`|Enthält den String der des aktuellen Ziels (bswp. für eine Innenanzeige)|

Mit diesem Wissen solltet ihr das System bereits ohne Probleme in jeden Bus einbauen und integrieren können

---

## Weitere Einstellungen/Konfiguration
Über die beigelegte Constfile lässt sich bereits einiges an der Konfiguration des Drukers ändern und anpassen, ich erkläre kurz alle dort definierten Werte:

|Konstante|Erklärung|
|---|---|
|`nuntius_printer_driverID`|Fahrer-ID, bzw. "Benutzername" für den Login|
|`nuntius_printer_driverPIN`|Fahrer-PIN, bzw. "Passwort" für den Login|
|`nuntius_printer_batteryTimeActiveScreen`|Der Drucker verfügt über einen integrierten Akku, der aufgeladen wird, sobald das Geärt mit Strom versorgt wird, das bedeutet, dass der Drucker auch eine gewisse Zeit lang ohne Bordstrom "überleben" kann. Dieser Wert legt fest, wie lange (in Sekunden) der Drucker nach dem Abschalten der Stromzufuhr noch normal bedienbar sein soll bevor sich der Bildschirm ausschaltet|
|`nuntius_printer_batteryTime`|Ist die oben angegebene Zeit überschritten, schaltet der Drucker in "Standby", der Bildschrim geht dann aus und das Gerät lässt sich weder bedienen, noch "macht" aktiv etwas. Alle Daten bleiben aber gespeichert. Der Drucker ist sofort wieder betriebsbereit und im Vorherigen Zustand, sobald der Strom wieder eingeschaltet ist. Der batteryTime-Wert gibt an, wie lange dieser Standby-Zustand anhält. Wird auch dieser überschritten, schaltet sich der Drucker vollständig ab. Wird dann erst wieder der Strom eingeschaltet, muss er neu Booten und man muss sich neu einloggen usw.|
|`nuntius_printer_bootSpeed`|Faktor für die Bootgeschwindigkeit. Normal ist 1, zu Testzwecken kann man den Wert beliebig höher setzen, um den Startvorgang zu beschleunigen|

Zusätzlich ist noch die curve "Boot" vorhanden. Die am besten nicht anfassen, sie gibt den Verlauf des Bootvorgans an.
Auch hier sind grundsätzlich noch weitere Konfigruationen geplant

---

## Anpassen der Hofdatei
**[NEU!]** Das Nuntius-System bietet absofort die Möglichkeit, automatisch "Fahrtende" oder "Pause" am Ende einer Fahrt zu schildern. Damit dies auch über Maps hinweg ohne große Probleme geht, wird das genaue Verhalten hierfür über die Hofdatei definiert. Nuntius erwartet deshalb in der Hofdatei ein spezielles "Options"-Ziel. Im Prinzip ein normales zusätzlches Ziel, welches als solches nicht benötigt wird und nur verwendet wird, in Nuntius bestimmte "pro Map"-Einstellungen vorzunehmen. Es ist nicht zwangsläufig erforderlich. Das System funktioniert auch weiterhin ohne so ein Ziel und ist somit vollständig mit jeder Standard-Hofdatei kompatibel, kann aber eben einige bestimmte Funktionen nicht anbieten.

Damit das alles so funktioniert, muss dieses Ziel den Code 1234 tragen. Außerdem muss der erste String (regulär der IBIS1-String) "NUNTIUS_OPTIONS" lauten. Damit bestätigen wir, dass es sich hierbei tatsächlich um das Optionsziel handelt und nicht um ein normales, welches zufällig den Code 1234 hat.
Die darauffolgenden Strings repräsentieren dabei dabei jeweils bestimmte Einstellungen:
|String|Funktion|
|---|---|
|0. (IBIS1)|Identifikationsstring, muss exakt "NUNTIUS_OPTIONS" lauten.|
|1. (ANNAX 1)|(als String formatierte Zahl) Code des Zieles, welches als "Fahrtende"-Ziel verwendet werden soll|
|2. (ANNAX 2)|(als String formatierte Zahl) Code des Zieles, welches als "Pause"-Ziel verwendet werden soll|

Weitere Optionen sollen folgen.

---

## Bedienung/Handhabung des Druckers
comming soon...
