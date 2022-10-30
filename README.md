# S0-Counter-ESP8266-ESPEasy



# Kommandos

## Übertragung per HTTP

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

