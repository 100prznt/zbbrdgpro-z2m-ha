<img src="docu\sonoff_zigbee_bridge_pro.png" alt="Sonoff ZigBee Bridge Pro" height="110" /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src="docu\logo-z2m.png" alt="Zigbee2MQTT Logo" height="110" /> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <picture>
  <source media="(prefers-color-scheme: dark)" srcset="docu\home-assistant-wordmark-vertical-color-on-dark.png">
  <source media="(prefers-color-scheme: light)" srcset="docu\home-assistant-wordmark-vertical-color-on-light.png">
  <img alt="Home Assistant Logo" height="110" src="docu\home-assistant-logomark-color-on-light.png">
</picture>


# Konfigurieren der Sonoff ZigBee Bridge Pro als Z2M Koordinator in Home Assistant

Damit die __ZigBee Bridge Pro von Sonoff__ als Zigbee2MQTT Koordinator in Home Assistant genutzt werden kann, ist es erforderlich eine alternative Firmware zu installieren. In diesem Repository wird das ganue Vorgehen dokumentiert.

## Vorbereitung

Vor Beginn der Arbeiten sind folgende Hard- und Softwaretools bereit zu legen.

### Hardware

* [Sonoff ZigBee Bridge Pro](https://amzn.to/3SMInVV)
* [USB zu TTL Seriell Adapter 3,3 V](https://amzn.to/4brv9Fu)
* [Jumper Kabel](https://amzn.to/3w89Dpd)
* [Stiftleiste (2,54 mm 5-polig)](https://amzn.to/3HVmDAV)
* Lötkolben inkl. Lötutensilien
* kleiner Phillips Schraubendreher

### Software

* [EspTool](https://github.com/espressif/esptool/releases/tag/v4.7.0) (v4.7.0 erfolgreich getestet)
* [Tasmota Firmware](https://ota.tasmota.com/tasmota32/release/tasmota32-zbbrdgpro.factory.bin) [Kopie in diesem Repository](tasmota32-zbbrdgpro.factory.bin)

## ESP32 Flashen

### Anschluss

Zuerst muss die Bridge geöffnet werden, hierzu sind die 4 Schrauben unter den Gummipuffern an der Unterseite zu entfernen.
Anschließend kann man die 5-polige Stiftleiste an die vorgesehene Stelle auf der Platine löten.
Der Seriell Adapter muss nun nach folgenden Schema mit der Stiftleiste verbunden werden.

| ZB Bridge Pro | Seriell Adapter |
|---------------|-----------------|
| TX0           | RXD             |
| RX0           | RXD             |
| GPIO0         | GND[^1]         |
| 3V3           | 3,3 V           |
| GND           | GND             |

[^1]: Der GPIO0 Anschluss muss nur beim Bootvorgang auf GND gezogen werden und kann anschließend getrennt werden, der ESP32 befindet sich dann im Bootloader Modus.

### EspTool

Nachdem die ZigBee Bridge Pro an den Seriell Adapter angeschlossen ist, kann dieser mit dem PC verbunden werden. Nach 3 Sekunden kann das Jumper Kabel vom GPIO0 Pin getrennt werden, der ESP32 befindet sich jetzt im Bootloader Modus und ist bereit zum Flashen der Firmware.

Im Gerätemanager des verwendeten PC muss nun der COM Anschluss identifiziert werden, welcher den Seriell Adapter zugewiesen wurde. Dann kann das EspTool gestartet werden.

TODO ![Screenshot EspTool]()

Folgende Einstellung müssen im EspTool vorgenommen werden:
1. COM Port auswählen, welcher dem Seriell Adapter zugewisen ist.
2. Firmware Datei (tasmota32-zbbrdgpro.factory.bin) auswählen.
3. Flashen starten

Wenn der Flash Vorgang erfolgreich war, kann der Seriell Adapter vom PC und der ZigBee Bridge getrennt werden.

## Tasmota konfigurieren

Die ZigBee Bridge Pro kann nun ganz normal über ein USB Netzteil (min. 1 A) und den vorgesehenen Micro-USB-Anschluss mit Spannung versorget werden. Nach ein paar Sekunden sollte die blaue WiFi LED blinken und Tasmota im AP Modus starten.

### WiFi einrichten

Für die Einrichtung der WiFi Verbindung nutze ich gerne das Smartphone. Hier sucht man einfach nach verfügbaren WiFi Netzen und verbindet sich mit dem Netz `Tasmota***`. Anschließend öffnet sich die Konfigurationsoberfläche für die WiFi Verbindung.

Nachdem die ZigBee Bridge mit dem lokalen Netzwerk verbunden ist kann man wieder an den PC wechseln und die Tasmota Startseite aufrufen. Die IP findet man in der Netzwerkübersicht des Routers (FritzBox etc.).

### CC2652 Flashen

Neben dem ESP32, welcher ja unter der Tasmota Firmware läuft, muss auch das ZigBee Modul (CC2652) mit einer alternativen Firmware versorgt werden. Die benötigten Dateien sind in der Firmware Datei (tasmota32-zbbrdgpro.factory.bin) enthalten und liegen somit auf der ZigBee Bridge bereit. Unter __Consoles__ -> __Manage File System__ können die benötigten Dateien auch eingesehen werden:

* [cc2652_flasher.be](berry\cc2652_flasher.be)
* [intelhex.be](berry\intelhex.be)
* [sonoff_zb_pro_flasher.be](berry\sonoff_zb_pro_flasher.be)
* [SonoffZBPro_coord_20220219.hex](cc2652\SonoffZBPro_coord_20220219.hex)

Zum Flashen wird nun die Berry Console geöffnet, __Consoles__ -> __Berry Scripting Console__ und folgende Befehle müssen zeilenweise ausgeführt werden:
```
import sonoff_zb_pro_flasher as cc
cc.load("SonoffZBPro_coord_20220219.hex")
cc.check()
```

Der letzte Befehl benötigt etwas Zeit zur ausführung und sollte folgendes zurück geben: 
```
11:26:19.675 FLH: Starting verification of HEX file
11:26:28.042 FLH: Verification of HEX file OK
```

Sollte hier etwas anderes zurück gegeben werden stimmt etwas nicht mit den übertragenen Dateien. Ggf. schafft hier ein erneutes Fashen der ESP32 Frimware abhilfe. Wenn die Verifikation erfolgreich war, kann mit folgendem Befehl das Übertragen der Firmware auf das CC2652 Modul gestartet werden:
```
cc.flash()
```
Achtung, das Flashen dauert etwas länger und läuft völlig ohne Rückmeldung. Nach 10 Minuten sollte der Vorgang abgeschlossen sein und man kann über die Konsolenausgabe das Ergebnis prüfen. wichtig ist, dass man keinen Reset durchführt, sondern direkt zur Konsole, __Main Menu__ -> __Consoles__ -> __Console__ navigiert. Hier müssen sich folgende Einträge finden lassen:
```
11:28:01.563 FLH: Flashing started (takes 5 minutes during which Tasmota is unresponsive)
11:34:50.172 FLH: Flashing completed: OK
11:34:50.266 FLH: Flash crc32 0x000000 - 0x2FFFF = bytes('1598929A')
```

## Konfigurieren für Zigbee2MQTT

Aktuell läuft die Bridge im Mode ZigBee2Tasmota. Für die Verwendung in Home Assistant ist es allerdings zu empfehlen den Modus umzustellen, sodass die ZHA Integration oder das Zigbee2MQTT Addon genutzt werden kann.

Über die Autokonfiguration, __Configuration__ -> __Auto-configuration__ lässt sich der Modus umstellen, hier muss __Sonoff ZBPro TCP__ als Konfiguration angewählt werden und mit __Apply Configuration__ übernommen werden. Nach einem Neustart sollte die ZigBee Bridge im TCP Modus starten.

### Prüfen der Konfiguration

Unter dem Eintrag __Configuration__ -> __Template__ muss der Eintrag wie folgt lauten:
```
{"NAME":"TCP ZBBridge Pro","GPIO":[0,0,576,0,480,0,0,0,0,1,1,5792,0,0,0,5472,0,320,5793,5504,0,640,608,32,0,0,0,0,0,1,0,0,0,0,0,0],"FLAG":0,"BASE":1}
```

Der Port, unter welchem die ZigBee Bridge erreichbar ist, lässt sich über die Regel, welche die TCPBridge startet konfigurieren. Hierzu muss folgendes in der Konsole, __Main Menu__ -> __Consoles__ -> __Console__ eingegeben werden:
```
backlog rule1 on system#boot do TCPStart 8888 endon ; rule1 1 ; TCPStart 8888 
```
Der Port (im Beispiel 8888) kann beliebig geändert werden.

## Einrichten Zigbee2MQTT

Aufgrund der umfangreicheren Produktdatenbank bevorzuge ich Zigbee2MQTT, anstelle von ZHA. Nach dem das Add On in Home Assistant installiert ist, muss der Port zur ZigBee Bridge hinterlegt werden. Daszu wird in der Konfiguration unter Serial folgender Eintrag erstellt:
```
port: tcp://192.168.1.100:8888 
```
Wobei IP und Port, im Beispiel 192.168.1.100:8888, natürlich nach den eigenen Einstellungen angepasst werden muss. 



> [!NOTE]
> Diese Anleitung basiert auf folgend genannten Quellen:
> * [Zigbee Device Compatibility Repository: Zigbee Bridge Pro](https://zigbee.blakadder.com/Sonoff_ZBBridge-P.html)
> * [Mikrocontroller.net: Sonoff ZigBee Bridge Pro & Tasmota](https://www.mikrocontroller.net/topic/544765)
> * [Espressif Systems: EspTool](https://github.com/espressif/esptool/releases/tag/v4.7.0)
> * [Notenoughtech.com: How to flash Tasmota on Sonoff ZB Bridge Pro](https://notenoughtech.com/home-automation/tasmota-on-sonoff-zb-bridge-pro/)
