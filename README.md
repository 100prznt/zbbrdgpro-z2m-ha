# Konfigurieren der Sonoff ZigBee Bridge Pro als Z2M Koordinator in Home Assistant

Diese Anleitung basiert auf folgend genannten Quellen:
* [Sonoff_ZBBridge-P.html](https://zigbee.blakadder.com/Sonoff_ZBBridge-P.html)
* [Forum Mikrocontroller](https://www.mikrocontroller.net/topic/544765)
* [Github](https://github.com/espressif/esptool/releases/tag/v4.7.0)
* [tasmota-on-sonoff-zb-bridge-pro](https://notenoughtech.com/home-automation/tasmota-on-sonoff-zb-bridge-pro/#flash)


Damit die ZigBee Bridge Pro von Sonoff als ZigBee2MQTT Koordinator in Home Assistant genutzt werden kann, ist es erforderlich eine alternative Firmware zu installieren.

## Vorbereitung

Vor Beginn der Arbeiten sind folgende Hard- und Softwaretools bereit zu legen.

### Hardware

* [Sonoff ZigBee Bridge Pro](https://amzn.to/3SMInVV)
* USB zu TTL Seriell Adapter 3,3 V
* Jumper Kabel
* Stiftleiste (2,54 mm 5-polig)
* Lötkolben inkl. Lötutensilien
* kleiner Philipsschraubendreher

### Software

* [EspTool](https://github.com/espressif/esptool/releases/tag/v4.7.0) (ich habe erfolgreich Version v4.7.0 eingesetzt)
* [Tasmota Firmware](https://ota.tasmota.com/tasmota32/release/tasmota32-zbbrdgpro.factory.bin)

## Flashen

### Anschluss

Zuerst muss die Bridge geöffnet werden, hierzu sind die 4 Schrauben unter den Gummipuffern an der Unterseite zu entfernen.
Anschließend kann man die 5-polige Stiftleiste an die vorgesehene Stelle auf der Platine löten.
Der Seriell Adapter muss nun nach folgenden Schema mit der Stiftleiste verbunden werden.

| ZB Bridge Pro | Seriell Adapter |
|---------------|-----------------|
| TX0           | RXD             |
| RX0           | RXD             |
| GPIO0         | GND*            |
| 3V3           | 3,3 V           |
| GND           | GND             |

\*) Der GPIO0 Anschluss muss nur wärend des Bootvorganges auf GND gezogen werden und kann anschließend getrennt werden, der ESP32 befindet sich dann im Bootloader Modus.

### ESP32 Firmware Flashen

Nachdem die ZigBee Bridge Pro an den Seriell Adapter angeschlossen ist, kann dieser mit dem PC verbunden werden. Nach 3 Sekunden kann das Jumper Kabel vom GPIO0 Pin getrennt werden, der ESP32 befindet sich jetzt im Bootloader Modus und ist bereit zum Flashen der Firmware.

Im Gerätemanager des verwendeten PC muss nun der COM Anschluss identifiziert werden, welcher den Seriell Adapter zugewiesen wurde. Dann kann das EspTool gestartet werden.

![Screenshot EspTool]()

Folgende Einstellung müssen im EspTool vorgenommen werden:
1. COM Port auswählen, welcher dem Seriell Adapter zugewisen ist.
2. Firmware Datei (tasmota32-zbbrdgpro.factory.bin) auswählen.
3. Flashen starten

Wenn der Flash Vorgang erfolgreich war, kann der Seriell Adapter vom PC und der ZigBee Bridge getrennt werden.

## Tasmota konfigurieren

Die ZigBee Bridge Pro kann nun ganz normal über ein USB Netzteil (min. 1 A) und den vorgesehenen Micro-USB-Anschluss mit Spannung versorget werden. Nach ein paar Sekunden sollte die blaue WiFi LED blinken und Tasmota im AP Modus starten.

### WiFi einrichten

Für die Einrichtung der WiFi Verbindung nutze ich gerne das Smartphone. Hier sucht man einfach nach verfügbaren WiFi Netzen und verbindet sich mit dem Netz ```Tasmota***```. Anschließend öffnet sich die Konfigurationsoberfläche für die WiFi Verbindung.

Nachdem die ZigBee Bridge mit dem lokalen Netzwerk verbunden ist kann man wieder an den PC wechseln und die Tasmota Startseite aufrufen. Die IP findet man in der Netzwerkübersicht des Routers (FritzBox etc.).

## Firmware auf CC2652 flashen

Neben dem ESP32, welcher ja unter der Tasmota Firmware läuft, muss auch das ZigBee Modul (CC2652) mit einer alternativen Firmware versorgt werden. Die benötigten Dateien sind in der Firmware Datei (tasmota32-zbbrdgpro.factory.bin) enthalten und liegen somit auf der ZigBee Bridge bereit. Unter _Consoles -> Manage File System_ können die benötigten Dateien auch eingesehen werden:

* cc2652_flasher.be
* intelhex.be
* sonoff_zb_pro_flasher.be
* SonoffZBPro_coord_20220219.hex

Zum Flashen wird nun die Berry Console geöffnet, _Consoles -> Berry Scripting Console_ und folgende Befehle müssen zeilenweise ausgeführt werden:
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
Achtung, das Flashen dauert etwas länger und läuft völlig ohne Rückmeldung. Nach 10 Minuten sollte der Vorgang abgeschlossen sein und man kann über die Konsolenausgabe das Ergebnis prüfen. wichtig ist, dass man keinen Reset durchführt, sondern direkt zur Konsole, _Main Menu -> Consoles -> Console_ navigiert. Hier müssen sich folgende Einträge finden lassen:
```
11:28:01.563 FLH: Flashing started (takes 5 minutes during which Tasmota is unresponsive)
11:34:50.172 FLH: Flashing completed: OK
11:34:50.266 FLH: Flash crc32 0x000000 - 0x2FFFF = bytes('1598929A')
```
