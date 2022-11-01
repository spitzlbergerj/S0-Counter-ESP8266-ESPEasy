# S0-Counter-ESP8266-ESPEasy

Dies ist ein einfacher S0 Counter zum "Digitalisieren" von Stromzählern mit S0 Schnittstelle auf Basis von Wemos D1 mini pro ESP8266 mit ESPEasy. Die ausgelesenen Werte werden an eine Homematic übertragen. Dort bzw. in Node-red erfolgt dann die Auswertung und auch die Sicherung des alten Zählerstandes bei einem Stromausfall am ESP8266, da dort die Werte nicht persistent gespeichert werden.

Achtung: Dies ist eine "revisionssichere" Digitalisierung des Zählerstandes! Es können Pulse verloren gehen, der Zählerstand kann bei Stromausfall am ESP8266 verloren gehen, usw.

Dies ist work in progress!


## D1 mini - Layout

![ESP8266 D1 Mini](https://i0.wp.com/randomnerdtutorials.com/wp-content/uploads/2019/05/ESP8266-WeMos-D1-Mini-pinout-gpio-pin.png?w=715&quality=100&strip=all&ssl=1)

Ich verwende jedoch einen D1 mini Pro, an den eine externe WLAN Antenne angeschlossen werden kann. Hierzu ist auf dem D1 mini Pro ein SMD Widerstand umzulöten!

## Schaltplan

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/Steckplatine.png"  width="400">

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/Schaltplan.png"  width="400">

## Konfiguration ESPEasy

### Hardware

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/ESPEasy-S0-Counter-Hardware.png"  width="400">

### Devices

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/ESPEasy-S0-Counter-Devices.png"  width="400">

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/ESPEasy-S0-Counter-Device-OLED.png"  width="400">

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/ESPEasy-S0-Counter-Device-PulseCounter.png"  width="400">

### Rules

<img src="https://github.com/spitzlbergerj/S0-Counter-ESP8266-ESPEasy/blob/main/img/ESPEasy-S0-Counter-RuleSet1.png"  width="400">

```
// -------------------------------------------------------
// Setzen der Pulse pro kWh pro Zählereinheit
// -------------------------------------------------------
//
On System#Boot do
  let,11,101     // Stromzähler 1: 100 Pulse pro kWh
  let,12,1001    // Stromzähler 2: 1000 Pulse pro kWh
  let,13,1001    // Stromzähler 3: 1000 Pulse pro kWh
endon


// -------------------------------------------------------
// Upload der Divisoren zur Homematic
// -------------------------------------------------------
//
On WiFi#Connected do
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter1Divisor%22).State([var#11])
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter2Divisor%22).State([var#12])
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter3Divisor%22).State([var#13])
endon

// -------------------------------------------------------
// aktuelles Total geteilt durch Pulse pro kWh = Zählerstand
// -------------------------------------------------------
//
on StromCounter1#* do
  let,1,%eventvalue2%/[var#11]
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter1TotalAkt%22).State(%eventvalue2%)
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter1Stand%22).State([var#1])
endon

on StromCounter2#* do
  let,2,%eventvalue2%/[var#12]
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter2TotalAkt%22).State(%eventvalue2%)
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter2Stand%22).State([var#2])
endon

on StromCounter3#* do
  let,3,%eventvalue2%/[var#13]
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter3TotalAkt%22).State(%eventvalue2%)
  SendToHTTP 192.168.178.160,8181,/egal.exe?ret=dom.GetObject(%22StromCounter3Stand%22).State([var#3])
endon
```

# Stromzähler Eltako

Auswerten möchte ich Stromzähler von Eltako des Typs DSZ12 und DSZ15. Über die S0 Schnittstelle wird folgendes beschrieben:


* Impulsausgang S0 nach DIN EN 62053-31

* potenzialfrei durch einen Optokoppler

* max. 30VDC/2OmA u. min.5VDC

* Impedanz100 Ohm

* Impulslänge 50ms

* 100 Imp./kWh (DSZ12) bzw. 1000 Imp./KWH (DSZ15) 

Laut Datenblatt soll die Ansteuerung also mit mindestens 5 Volt erfolgen. Viele andere Projekte zeigen, dass die 3V3 der GPIO Pins auch ausreichen sollten. 


# Kommandos

### Übertragung per HTTP

Setzen eines Zählerstandes: http://192.168.178.146/control?cmd=ESP8266GStromPuls1.SetPulseCounterTotal,10000



## Umgang mit internen Variablen

Internal variables
A really great feature to use is the internal variables. You set them like this:

Let,<n>,<value>
Where n must be a positive integer (type uint32_t) and the value a floating point value. To use the values in strings you can either use the %v7% syntax or [var#7]. BUT for formulas you need to use the square brackets in order for it to compute, i.e. [var#12].

If you need to make sure the stored value is an integer value, use the [int#n] syntax. (i.e. [int#12]) The index n is shared among [var#n] and [int#n].

On the “System Variables” page of the web interface all set values can be inspected including their values. If none is set, “No variables set” will be shown.

If a specific system variable was never set (using the Let command), its value will be considered to be 0.0.
 Quelle: https://espeasy.readthedocs.io/en/latest/Rules/Rules.html

# Quellen

ESPEasy Command Reference: https://www.letscontrolit.com/wiki/index.php/ESPEasy_Command_Reference
https://espeasy.readthedocs.io/en/latest/Reference/Command.html

ESPEasy Pulse Counter: https://espeasy.readthedocs.io/en/latest/Plugin/P003.html

ESPEasy Kommandos von remote: https://nerdiy.de/howto-espeasy-befehle-ausfuehren/
ESPEasy Node-red: https://nerdiy.de/howto-node-red-gpio-eines-espeasy-geraets-steuern/

