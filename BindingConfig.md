This page contains samples for binding configurations. These samples are sorted by binding.



## How to send Date and Time from NTP to KNX ##

This example sends the current date and time from the NTP to the KNX binding

```
DateTime Date "Date & Time [%1$td.%1$tm.%1$tY %1$tT]" { ntp="Europe/Berlin:de_DE", knx="11.001:0/7/1, 10.001:0/7/2" } 
```

**Items**:
> 0/0/1 is the GA for Date
> 0/0/2 is the GA for Time

Additional information on date and time formatting can be found
[here](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/Formatter.html#syntax)


## How to get temperatures from OW-SERVER via HTTP binding ##

Requirements:
  * [OW-SERVER with Ethernet](http://www.embeddeddatasystems.com/OW-SERVER-1-Wire-to-Ethernet-Server-Revision-2_p_152.html)
  * [DS18B20](http://www.mikrocontroller.net/part/DS18B20)-Sensors (or DS18S20) connected via 1-wire-bus

Instructions:
  1. Go to http://`<ow-server-IP>`/devices.htm and look for the ROMId-Value
> > [https://lh3.googleusercontent.com/-XdaTtDlzyEU/Ud7\_uB4fPCI/AAAAAAAALTY/pmajPpkk6uo/s300/owserver\_devices.PNG](https://lh3.googleusercontent.com/-XdaTtDlzyEU/Ud7_uB4fPCI/AAAAAAAALTY/pmajPpkk6uo/w780-h445-no/owserver_devices.PNG)
  1. Add an Number-Item to your items-configuration like this one

```
// Example:
Number Temp_Kitch "Küche [%.1f °C]" { http="<[http://192.168.1.16/details.xml:60000:REGEX(.*?<ROMId>A7000002CC4D2228</ROMId>.*?<Temperature Units=\"Centigrade\">(.*?)</Temperature>.*)]" }
```


> Replace the ip address and the ROMId-value with your data.


## How to get humidity from OW-SERVER via HTTP binding ##

Device: OW-ENV-TH (EDS0065)

// Example:
```
Number Humidity "Humidity [%.1f %%]" { http="<[http://192.168.1.16/details.xml:5000:REGEX(.*?<ROMId>C30010000027767E</ROMId>.*?<Humidity Units=\"PercentRelativeHumidity\">(.*?)</Humidity>.*)]" }
```

## How to get contact from OW-SERVER via HTTP binding ##

Device: D2C (DS2406)

// Example:
```
Number Door "Door [MAP(contact.map):%d]" { http="<[http://192.168.1.16/details.xml:5000:REGEX(.*?<ROMId>BD0000009D93DC12</ROMId>.*?<InputLevel_A>(.*?)</InputLevel_A>.*)]" }
```
You may want to change the query-interval (here 5000ms) to a few seconds.
You can get the value for InputLevel\_B, too. ;)

contact.map:
```
0=open
1=close
-=UNKNOWN
```


## How to turn on/off a switch from OW-SERVER via HTTP binding ##

Device: D2C (DS2406)

// Example:
```
Switch Lamp "Switch [MAP(switch.map):%d]" { http="<[http://192.168.1.16/details.xml:5000:REGEX(.*?<ROMId>BD0000009D93DC12</ROMId>.*?<InputLevel_B>(.*?)</InputLevel_B>.*)]" }
```
This only reads the state of the switch.

switch.map:
```
1=ON
0=OFF
-=undefiniert
```

To turn the switch on or off you need to define two rules:
```
rule "Turn Lamp on"
when 
	Item Lamp changed to ON
then
	sendHttpGetRequest("http://192.168.1.16/devices.htm?rom=BD0000009D93DC12&variable=FlipFlop_B&value=0")
end

rule "Turn Lamp off"
when 
	Item Lamp changed to OFF
then
	sendHttpGetRequest("http://192.168.1.16/devices.htm?rom=BD0000009D93DC12&variable=FlipFlop_B&value=1")
end
```

## How to read the status from a OneWire sensor DS2413 (2 port I/O) ##

item definition:
```
Number WindowContact1   "Window 1 is [MAP(contact.map):%s]"                  (All) { onewire = "3C.16AA13000000#sensed.A" }
Number WindowContact2   "Window 2 is [MAP(contact.map):%s]"                  (All) { onewire = "3C.16AA13000000#sensed.B" }
```

sitemap definition:
```
Text item=WindowContact1
Text item=WindowContact2
```

map file contact.map
```
1=closed
0=opened
```

Sample output for this definition would then be "Window 1 is opened" or "Window 2 is closed".

## How to get data from Kostal Piko solar inverter via HTTP binding ##

[https://lh4.googleusercontent.com/-pa3EqaUPe\_E/Ud8lK56J\_-I/AAAAAAAALT0/KNOfi7gWe\_c/s300/openhab\_kostal\_piko\_screenshot.PNG](https://lh4.googleusercontent.com/-pa3EqaUPe_E/Ud8lK56J_-I/AAAAAAAALT0/KNOfi7gWe_c/w563-h634-no/openhab_kostal_piko_screenshot.PNG)


```
/* AC-Leistung */
Number Solar_Aktuell "aktuell [%.0f W]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?aktuell</td>.*? (.*?)</td>.*)]" }

/* Energie */
Number Solar_Gesamt "Gesamtenergie [%.0f kWh]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?Gesamtenergie</td>.*? (.*?)</td>.*)]" }
Number Solar_Tagesenergie "Tagesenergie [%.2f kWh]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?Tagesenergie</td>.*? (.*?)</td>.*)]" }

/* PV-Generator */
Number Solar_PVG_Str1_Spannung "String 1 Spannung [%.0f V]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?String 1</u></td>.*?Spannung</td>.*? (.*?)</td>.*)]" }
Number Solar_PVG_Str1_Strom "String 1 Strom [%.2f A]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?String 1</u></td>.*?Strom</td>.*? (.*?)</td>.*)]" }

Number Solar_PVG_Str2_Spannung "String 2 Spannung [%.0f V]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?String 2</u></td>.*?Spannung</td>.*? (.*?)</td>.*)]" }
Number Solar_PVG_Str2_Strom "String 2 Strom [%.2f A]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?String 2</u></td>.*?Strom</td>.*? (.*?)</td>.*)]" }


/* Ausgangsleistung */
Number Solar_AL_L1_Spannung "L1 Spannung[%.0f V]"  { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L1</u></td>.*?Spannung.*?Spannung</td>.*? (.*?)</td>.*)]" }
Number Solar_AL_L1_Leistung "L1 Leistug [%.0f W]"  { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L1</u></td>.*?Leistung</td>.*? (.*?)</td>.*)]" }

Number Solar_AL_L2_Spannung "L2 Spannung [%.0f V]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L2</u></td>.*?Spannung.*?Spannung</td>.*? (.*?)</td>.*)]" }
Number Solar_AL_L2_Leistung "L2 Leistug [%.0f W]"  { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L2</u></td>.*?Leistung</td>.*? (.*?)</td>.*)]" }

Number Solar_AL_L3_Spannung "L3 Spannung [%.0f V]" { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L3</u></td>.*?Spannung</td>.*? (.*?)</td>.*)]" }
Number Solar_AL_L3_Leistung "L3 Leistug [%.0f W]"  { http="<[http://pvserver:password@192.168.0.27/index.fhtml:60000:REGEX(.*?L3</u></td>.*?Leistung</td>.*? (.*?)</td>.*)]" }
```

Make sure you replace "password" with your password and edit the ip address.

## How to send commands to Telldus Tellstick ##

This a simple example of how to command your tellstick devices from Openhab. For event triggered inbound integration, check the [Integration with other applications](https://code.google.com/p/openhab-samples/wiki/integration#Telldus_Tellstick) page.

Please note that if you use the inbound integration you must name you item `td_device_<number>` where `<number>` is the tellstick device enumeration as listed from command: `tdtool -l`. Obviously you also need to change enumeration after `--on` and `--off` in the exec binding.

Item definition:
```
Switch td_device_5 "Tellstick device 5" {exec=">[ON:tdtool --on 5] >[OFF:tdtool --off 5]"}
```


## How to get power on a TV connected to HDMI with exec binding and update the status automatically ##

This is an example of how to power on a TV connected to the openhab server via HDMI. First you have to install cec-client utility on your host (you can see more details in http://blog.endpoint.com/2012/11/using-cec-client-to-control-hdmi-devices.html page)

The next thing is use the exec and the samsung binding (I can't switch on the TV with the samsung binding, and I can't switch off with the cec-client). My item definition shows like:

```
Switch          TV_GF_Living_TV_power   "Power"     (GF_Living_TV)  { exec="ON:/usr/local/bin/samsungTvStart.sh, OFF:/bin/true", samsungtv="OFF:Livingroo
m:KEY_POWEROFF, ON:Livingroom:KEY_POWERON" }
```

And the script /usr/local/bin/samsungTvStart.sh is
```
echo 'on 0' | cec-client -s
```

The next thing is automatically check and update the status. I use a shell script that I run every minute with cron. The script /usr/local/bin/samsungTvCheck.sh is

```
#!/bin/bash
OH_URL=[OPENHAB_URL]
OH_USER=[OPENHAB_USER]
OH_PASS=[OPENHAB_PASS]
OH_ITEM=[OPENHAB_ITEM]
RESULT=`echo pow 0 | cec-client -d 1 -s | grep "power status:" | awk '{ print $3; }'`

case $RESULT in
        on)
                curl --user $OH_USER:$OH_PASS --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "ON" $OH_URL/rest/items/$OH_ITEM/state
                exit 0
                ;;
        *)
                curl --user $OH_USER:$OH_PASS --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "OFF" $OH_URL/rest/items/$OH_ITEM/state
                exit 1
esac
```

and you can add a line to your cron (in linux systems) with the command

```
crontab -e
*/1 * * * * /usr/local/bin/samsungTvCheck.sh
```

Note that you have to change [OPENHAB\_URL](OPENHAB_URL.md), [OPENHAB\_USER](OPENHAB_USER.md),  [OPENHAB\_PASS](OPENHAB_PASS.md) and [OPENHAB\_ITEM](OPENHAB_ITEM.md) according to your installation. This script update the status of the item, and you can see if your childs has switch on the tv ;)