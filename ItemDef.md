

## Howto use homematic door contact sensors ##
```
Contact corFrontDoor "Front Door [%s]" <frontdoor> (gRCor, gLock) { homematic="HEQ0358465:1#STATE" }
Text item=corFrontDoor
```

## Howto use homematic window contact sensors ##
```
Number lrWindowRight "Window Right [MAP(contact.map):%d]" <contact> (gRLvng) { homematic="IEQ0203214:1#STATE" }
Text item=lrWindowRight
```

transform/contact.map:
```
0=CLOSED
1=TILTED
2=OPEN
-=UNKNOWN
```

## Howto read Homematic heater valve state ##
```
Dimmer lrHeaterRight "Heater Right [%d %%]" <heating> (gRLvng)                { homematic="IEQ0537568:1#VALVE_STATE" }
Text item=lrHeaterRight
```

## Howto use Homematic temperature regulator: ##
```
Number lrTempSet "Target Temperature [%d °C]" <temperature> (gRLvng, gRBed) { homematic="IEQ0053616:2#SETPOINT" }
Setpoint item=lrTempSet step=0.5 minValue=15 maxValue=30
```

## How to get special characters like "%" in a label text: ##
```
Number Humidity "Humidity [%.1f %%]"
```

## Howto configure Homematic light switch: ##
```
Switch brLightCeil "Ceiling" (gRBed, gLight) { homematic="IEQ0001542:1#STATE" }
Switch item=brLightCeil
```

## How to configure Homematic temperature and humidity sensor: ##
```
Number lrTemp "Current Temp [%.1f °C]" <temperature> (gRLvng, gWthrDta) { homematic="IEQ0053616:1#TEMPERATURE" }
Number lrHumid "Humidity [%d %%]" <waterdrop> (gRLvng, gWthrDta) { homematic="IEQ0053616:1#HUMIDITY" }

Text item=lrTemp
Text item=lrHumid
```

## How to configure Homematic motion and brightness sensors: ##
```
Switch corMotion "Motion Detected" (gRCor) { homematic="GEQ0128171:1#MOTION" }
Number corBright "Brightness [%.1f %%]" (gRCor) { homematic="GEQ0128171:1#BRIGHTNESS" }

Switch item=corMotion
Text item=corBright
```
I dont like that the motion switch is "writeable". Maybe someone can post a proper rendering object for the motion detector.

## How to configure a switch to be a pushbutton: ##
[Thread http://knx-user-forum.de/openhab/27123-einfacher-taster-openhab.html](German.md)

Item:
```
Switch Garage_Gate { binding="xxx", autoupdate="false"}
```

Sitemap:
```
Switch item=Garage_Gate label="Garage" mappings=[ON="Go!"]
```
The magic happens with autoupdate="false" which keeps the state even an ON command has been received. This way, it's always off unless you explicitly post an update to this item.

## How to control a homematic roller shutter with an EnOcean Rocker ##

Item:
```
Rollershutter Blinds_Left <rollershutter> (Shutters) {homematic="id=JEQXXXXXX, channel=1, parameter=LEVEL", enocean="{id=00:00:00:00, eep=F6:02:01}"}
```

## How to control a homematic dimmer with an EnOcean Rocker (OnOff Profile) ##

Item:
```
Dimmer Lights_Left <lights> (Lights) {homematic="id=GEQXXXXXX, channel=2, parameter=LEVEL", enocean="{id=00:00:00:00, channel=A, eep=F6:02:01}"}}
```