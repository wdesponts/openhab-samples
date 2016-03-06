

## How to redirect your log entries to the syslog ##
You just need to add some lines to your logback.xml.

Like:
```
<appender name="SYSLOG" class="ch.qos.logback.classic.net.SyslogAppender">
 <syslogHost>localhost</syslogHost>
 <facility>AUTH</facility>
 <suffixPattern>{}openhab: [%thread] %logger %msg</suffixPattern>
</appender>

<root level="INFO">
 <appender-ref ref="SYSLOG" />
</root>
```

## How to do a proper ICMP ping on Linux ##

Java is not capable to open a raw socket to do a ICMP ping (see https://code.google.com/p/openhab/issues/detail?id=134). As a workaround, you can use the exec binding on Linux:
```
Switch PingedItem { exec="<[/bin/sh@@-c@@ping -c 1 192.168.0.1 | grep \"packets transmitted\" | sed -e \"s/.*1 received.*/ON/\" -e \"s/.*0 received.*/OFF/\":30000:REGEX((.*))]" }
```

## How to add current or forecast weather icons to your sitemap ##

This example gets the weather information from the Wunderground online api service (for details see http://www.wunderground.com/weather/api/) and shows the current weather icon and weather state (such as partly cloudy) in one line like it would show any other Openhab item.

  * in the items file, create a string as follows:

```
  String w   "Today [%s]"   <w>  { http="<[http://api.wunderground.com/api/<your api key>/forecast/q/Luxembourg/Luxembourg.xml:3600000:XSLT(wunderground_icon_forecast.xsl)]"} 
```

  * copy the weather icons to your images folder and rename and convert them to png, e.g. "w-partly\_cloudy.png". The weather icon sets can be found here:

> http://www.wunderground.com/weather/api/d/docs?d=resources/icon-sets

  * create a stylesheet named wunderground\_icon\_forecast.xsl containing the following lines (place it under configurations/transform):

```
<?xml version="1.0"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="2.0">
        
        <xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

        <xsl:template match="/">
                <!-- format: hh:mm:ss -->
                <xsl:value-of select="//response/forecast/txt_forecast/forecastdays/forecastday/icon/text()" />
        </xsl:template>

</xsl:stylesheet>
```

  * display the string in the .sitemap file:

```
  Text item=w 
```

## How to configure openHAB to start automatically on Linux ##

Create a new file in /etc/init.d/openhab using your preferred editor (e.g. nano) and copy the code below.

```
#! /bin/sh
### BEGIN INIT INFO
# Provides:          openhab
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: OpenHAB Daemon
### END INIT INFO

# Author: Thomas Brettinger 

# Do NOT "set -e"

# PATH should only include /usr/* if it runs after the mountnfs.sh script
PATH=/sbin:/usr/sbin:/bin:/usr/bin

DESC="Open Home Automation Bus Daemon"
NAME=openhab
DAEMON=/usr/bin/java
PIDFILE=/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
ECLIPSEHOME="/opt/openhab";
HTTPPORT=8080
HTTPSPORT=8443
TELNETPORT=5555
RUN_AS=ben

# get path to equinox jar inside $eclipsehome folder
cp=$(find $ECLIPSEHOME/server -name "org.eclipse.equinox.launcher_*.jar" | sort | tail -1);

DAEMON_ARGS="-Dosgi.clean=true -Declipse.ignoreApp=true -Dosgi.noShutdown=true -Djetty.port=$HTTPPORT -Djetty.port.ssl=$HTTPSPORT -Djetty.home=$ECLIPSEHOME -Dlogback.configurationFile=$ECLIPSEHOME/configurations/logback.xml -Dfelix.fileinstall.dir=$ECLIPSEHOME/addons -Djava.library.path=$ECLIPSEHOME/lib -Djava.security.auth.login.config=$ECLIPSEHOME/etc/login.conf -Dorg.quartz.properties=$ECLIPSEHOME/etc/quartz.properties -Djava.awt.headless=true -jar $cp -console ${TELNETPORT}"

# Exit if the package is not installed
[ -x "$DAEMON" ] || exit 0

# Read configuration variable file if it is present
[ -r /etc/default/$NAME ] && . /etc/default/$NAME

# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh

# Define LSB log_* functions.
# Depend on lsb-base (>= 3.2-14) to ensure that this file is present
# and status_of_proc is working.
. /lib/lsb/init-functions

#
# Function that starts the daemon/service
#
do_start()
{
    # Return
    #   0 if daemon has been started
    #   1 if daemon was already running
    #   2 if daemon could not be started
    start-stop-daemon --start --quiet --make-pidfile --pidfile $PIDFILE --chuid $RUN_AS --chdir $ECLIPSEHOME --exec $DAEMON --test > /dev/null \
        || return 1
    start-stop-daemon --start --quiet --background --make-pidfile --pidfile $PIDFILE --chuid $RUN_AS --chdir $ECLIPSEHOME --exec $DAEMON -- $DAEMON_ARGS \
        || return 2
    # Add code here, if necessary, that waits for the process to be ready
    # to handle requests from services started subsequently which depend
    # on this one.  As a last resort, sleep for some time.
    return 0
}

#
# Function that stops the daemon/service
#
do_stop()
{
    # Return
    #   0 if daemon has been stopped
    #   1 if daemon was already stopped
    #   2 if daemon could not be stopped
    #   other if a failure occurred
    start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
    RETVAL="$?"
    [ "$RETVAL" = 2 ] && return 2
    # Wait for children to finish too if this is a daemon that forks
    # and if the daemon is only ever run from this initscript.
    # If the above conditions are not satisfied then add some other code
    # that waits for the process to drop all resources that could be
    # needed by services started subsequently.  A last resort is to
    # sleep for some time.
    start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
    [ "$?" = 2 ] && return 2
    # Many daemons don't delete their pidfiles when they exit.
    rm -f $PIDFILE
    return "$RETVAL"
}

#
# Function that sends a SIGHUP to the daemon/service
#
do_reload() {
    #
    # If the daemon can reload its configuration without
    # restarting (for example, when it is sent a SIGHUP),
    # then implement that here.
    #
    do_stop
    sleep 1
    do_start
    return 0
}

case "$1" in
  start)
    log_daemon_msg "Starting $DESC"
    do_start
    case "$?" in
        0|1) log_end_msg 0 ;;
        2) log_end_msg 1 ;;
    esac
    ;;
  stop)
    log_daemon_msg "Stopping $DESC" 
    do_stop
    case "$?" in
        0|1) log_end_msg 0 ;;
        2) log_end_msg 1 ;;
    esac
    ;;
  status)
       status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
       ;;
  #reload|force-reload)
    #
    # If do_reload() is not implemented then leave this commented out
    # and leave 'force-reload' as an alias for 'restart'.
    #
    #log_daemon_msg "Reloading $DESC" "$NAME"
    #do_reload
    #log_end_msg $?
    #;;
  restart|force-reload)
    #
    # If the "reload" option is implemented then remove the
    # 'force-reload' alias
    #
    log_daemon_msg "Restarting $DESC" 
    do_stop
    case "$?" in
      0|1)
        do_start
        case "$?" in
            0) log_end_msg 0 ;;
            1) log_end_msg 1 ;; # Old process is still running
            *) log_end_msg 1 ;; # Failed to start
        esac
        ;;
      *)
          # Failed to stop
        log_end_msg 1
        ;;
    esac
    ;;
  *)
    #echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
    echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
    exit 3
    ;;
esac

:
```

Make the script executable and configure it to run on boot.

```
sudo chmod a+x /etc/init.d/openhab
sudo update-rc.d /etc/init.d/openhab defaults
```

Now whenever your Linux machine boots openHAB will be automatically started.

## Use cheap bluetooth dongles on remote PCs to detect your phone/watch ##

This is more an idea really but consider that openhab can know your home by picking up on the bluetooth on your phone, using cheap 3 euro bluetooth dongles connected to the various PCs and media centres you may have.

EG On an ubuntu/debian based PC.

1) Install the "bluez" bluetooth stack.
apt-get install bluez

2) Use the bash script below to do a simple scan several times per min.  This script looks for my phone "jon-phone".  Its made no real difference to the battery life on my android 4.x phone.  The script updates a switch in openhab called "zoneOneState".  You could have as many zones as you need/want to cover your house.

```
#!/bin/bash

while true
do
  DEVICES=`hcitool scan`

  if [[ $DEVICES = *jon-phone* ]]
  then
    curl --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "ON" http://192.168.1.121:8080/rest/items/zoneOneState/state
  else
    curl --max-time 2 --connect-timeout 2 --header "Content-Type: text/plain" --request PUT --data "OFF" http://192.168.1.121:8080/rest/items/zoneOneState/state
  fi

  sleep 30
done
```

3) The rule governing the decision process on if your house is occupied can be as complex or simple as you need.  The rule below is clunky but simple and gets the job done.  It uses three zone switches to decide if it should set a switch called "occupiedState" to on or off.  Then using the occupiedState switch you can make your other openhab rules behave in different ways.  Each zone checked reports true if your bluetooth device has been seen within the last 5 mins (helps stop flap caused by the patchy coverage of bluetooth signals).

```
//FIGURE IF HOUSE OCCUPIED EVERY 30 SECONDS
rule "check occupiedState"
when
	Time cron "*/30 * * * * ?"
then
	if (occupiedState.state == ON)
	{
		zoneOneFlag = true
		zoneTwoFlag = true
		zoneThreeFlag = true

		if (zoneOneState.state == OFF)
		{
			if (!zoneOneState.changedSince (now.minusMinutes(5)))
			{
				//sendCommand("occupiedState", "OFF")
				zoneOneFlag = false
			}
		}
		if (zoneTwoState.state == OFF)
		{
			if (!zoneTwoState.changedSince (now.minusMinutes(5)))
			{
				//sendCommand("occupiedState", "OFF")
				zoneTwoFlag = false
			}
		}
		if (zoneThreeState.state == OFF)
		{
			if (!zoneThreeState.changedSince (now.minusMinutes(5)))
			{
				//sendCommand("occupiedState", "OFF")
				zoneThreeFlag = false
			}
		}

		if (zoneOneFlag == false && zoneTwoFlag == false && zoneThreeFlag == false) {
			sendCommand("occupiedState", "OFF")
		}
	} else {
		if (zoneOneState.state == ON || zoneTwoState.state == ON || zoneThreeState.state == ON) {
			sendCommand("occupiedState", "ON")
		}
	}
```

## Get connection status of all network devices from Fritz!Box, eg for presence detection ##
For cases where bluetooth detection described above or network health binding is not suitable.

**Requirements**

  * Fritz!Box with telnet access
  * Linux server (preferably the openHAB server), tested with ReadyNAS
  * Linux knowledge to adopt scripts

**Methodology**

  * Shell script on the Fritz!Box to dump all known network devices with their current connection status
  * This script is called from the Linux server through a cron job periodically (eg every minute), parses the dump and updates openHAB items according to their connection status

**Restrictions**

  * Only tested on ReadyNAS Linux with manual added packages, others might need adoption
  * All parameters like login/password/paths/script names "hard-coded" into the scripts, need adoption
  * Absolutely **no** error-checks, use at own risk
  * Due to these restrictions only applicable for people with at least basic Linux knowledge

**How-To**

### On the Fritz!Box ###

The Fritz!Box shows a reliable status of devices connected to the network using the built-in _ctlmgr\_ctl r landevice_ command (see http://www.wehavemorefun.de/fritzbox/Landevice).

Create the script _landevices.sh_ permanently (should not be deleted at reboot), for example in folder _/var/media/ftp/Fritz!Box/_. Make it executable. Take care not to indent the script as the Fritz!Box does not like that.

```
#!/bin/sh
i=0
count_devices=`ctlmgr_ctl r landevice settings/landevice/count`
while [ $i -lt $count_devices ]
do
device="landevice="$i
#for command in name ip mac manu_name dhcp static_dhcp wlan ethernet active online speed deleteable wakeup source neighbour_name is_double_neighbour_name ipv6addrs ipv6_ifid
for command in name active
do
output="`ctlmgr_ctl r landevice settings/landevice$i/$command`"
device=$device" "$command"="$output
done
echo $device
i=`expr $i + 1`
done
```

The commented out for loop shows all possible subcommands, so it is also possible to output eg the speed or mac adress of all landevices. However this would also require a change on the calling script below. Once created on the Fritz!Box and executed it should provide you all known network devices with their current status:

```
# ./landevices.sh
landevice=0 name=BoyLaptop active=0
landevice=1 name=BoyPhone active=0
landevice=2 name=BoyiPod active=0
landevice=3 name=DGOfficePrinter active=1
landevice=4 name=DGTVRasPlex active=1
landevice=5 name=DGTVWii active=0
landevice=6 name=DadLaptop active=0
landevice=7 name=DadLaptopWork active=0
landevice=8 name=DadLaptopWork active=0
landevice=9 name=DadPC active=1
landevice=10 name=DadPhone active=1
landevice=11 name=DadPhoneWork active=1
landevice=12 name=EGEntranceAP active=1
landevice=13 name=EGTVRasPlex active=1
landevice=14 name=EGWZAP active=0
landevice=15 name=GirlLaptop active=0
landevice=16 name=GirlPhone active=1
landevice=17 name=GuestLaptop active=0
landevice=18 name=KGOfficePrinter active=1
landevice=19 name=KGOfficeReadyNAS active=1
landevice=20 name=KGTEC1AP active=1
landevice=21 name=KGTEC2AP active=1
landevice=22 name=MomLaptop active=0
landevice=23 name=MomLaptop active=0
landevice=24 name=MomLaptopWork active=0
landevice=25 name=MomPhone active=0
```

If required, use the Fritz!Box GUI to give the devices some reasonable names in order to easily assign phones/laptops etc to people or media devices to floors/rooms. Using the _ctlmgr\_ctl_ command it would also be possible to only query for specific devices (go through the full list and look for either name or mac or both), but any change in the output of this script requires changes on the other end as well.

### On the Linux server ###

Create a _expect_ script in a folder on your Linux Server and make it executable. The only purpose of this script is only to connect to your Fritz!Box through a telnet session and call the script created there.

```
#!/usr/bin/expect
spawn telnet <<<your fritzbox ip here>>>
expect "user: "
send "<<<your fritzbox login here>>>\r"
expect "password: "
send "<<<your fritzbox password here>>>\r"
expect "telnet"
expect "#"
send "\r"
expect "#"
send "/var/media/ftp/Fritz!Box/landevices.sh\r"
expect "#"
send "exit 0\r"
```

Here the IP of the Fritz!Box, your credentials and the path to the script need to be adopted. Furthermore the above script is for the newest version of the Fritz!box firmware where multiple user accounts can be created. In case this is not your case, you have to omit the "user:" and "login" _expect/send_ statements in the script above, as in that case the Fritz!Box telnet session only requires the password, not the user. Again, to validate the functionality just execute the script:

```
ReadyNAS:~# ./fritzbox_devices.expect
spawn telnet 192.168.178.1

Entering character mode
Escape character is '^]'.

Fritz!Box user: admin
password:


BusyBox v1.20.2 (2013-05-13 12:53:07 CEST) built-in shell (ash)
Enter 'help' for a list of built-in commands.

ermittle die aktuelle TTY
tty is "/dev/pts/1"
weitere telnet Verbindung aufgebaut
#
# /var/media/ftp/Fritz!Box/landevices.sh
landevice=0 name=BoyLaptop active=0
landevice=1 name=BoyPhone active=0
landevice=2 name=BoyiPod active=0
landevice=3 name=DGOfficePrinter active=1
landevice=4 name=DGTVRasPlex active=1
landevice=5 name=DGTVWii active=0
landevice=6 name=DadLaptop active=0
landevice=7 name=DadLaptopWork active=0
landevice=8 name=DadLaptopWork active=0
landevice=9 name=DadPC active=1
landevice=10 name=DadPhone active=1
landevice=11 name=DadPhoneWork active=1
landevice=12 name=EGEntranceAP active=1
landevice=13 name=EGTVRasPlex active=1
landevice=14 name=EGWZAP active=0
landevice=15 name=GirlLaptop active=0
landevice=16 name=GirlPhone active=1
landevice=17 name=GuestLaptop active=0
landevice=18 name=KGOfficePrinter active=1
landevice=19 name=KGOfficeReadyNAS active=1
landevice=20 name=KGTEC1AP active=1
landevice=21 name=KGTEC2AP active=1
landevice=22 name=MomLaptop active=0
landevice=23 name=MomLaptop active=0
landevice=24 name=MomLaptopWork active=0
landevice=25 name=MomPhone active=0
# ReadyNAS:~#
```

As you can see here we have the original output of the script plus some "overhead" for the telnet session.

Now create a _sed_ script _fritzbox\_devices.sed_ that will create _curl_ commands to interface with the openHAB server:

```
#!/bin/sed -f
s#\r$##
s#landevice=[0-9]* name=#curl --silent -H \"Content-Type: text/plain\" http://localhost:8080/rest/items/landevice_#g
s#active=0#-d \"OFF\"#g
s#active=1#-d \"ON\"#g
```

The first line here is just to eliminate "DOS-like" \r at the end of each line which show up with a ReadyNAS as server and might not be required in your setup, but you can still leave them. The second line replaces the "landevice=....name=...... with a _curl_ command. In order to have reasonable variable names again in openHAB the landevices there start with "landevice_", so "DadPhone" on the Fritz!Box gets landevice\_DadPhone in openHAB. Furthermore_localhost:8080_needs to be changed to match your openHAB server if is not running on the same machine. The last two lines replace the "active=0" or "active=1" with the correct statements for_curl_._

Another validation of the scripts created so far:

```
ReadyNAS:~# ./fritzbox_devices.expect | fgrep "landevice=" | ./fritzbox_devices.sed
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyPhone -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_BoyiPod -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGOfficePrinter -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGTVRasPlex -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DGTVWii -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptopWork -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadLaptopWork -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPC -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPhone -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_DadPhoneWork -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGEntranceAP -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGTVRasPlex -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_EGWZAP -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GirlLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GirlPhone -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_GuestLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGOfficePrinter -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGOfficeReadyNAS -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC1AP -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC2AP -d "ON"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptop -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomLaptopWork -d "OFF"
curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_MomPhone -d "OFF"
ReadyNAS:~#
```

### Configuration in openHAB ###

Now it is time to create the required items in openHAB - One Switch for each landevice that is supposed to be used later. In case you don't configure all landevices as items you will get error messages in openHAB logs, however these can be ignored. Just to have a "clean" configuration you might want to add all devices.

```
Switch landevice_KGOfficeReadyNAS	"ReadyNAS"		<network> (KG_TEC1, gNetworkServer)
Switch landevice_EGEntranceAP		"Airport Erdgeschoss"	<network> (gWZ, gNetworkServer)
Switch landevice_EGWZAP			"Airport Repeater Erdgeschoss"	<network> (gWZ, gNetworkServer)
Switch landevice_KGTEC1AP		"Airport Keller TEC1"	<network> (KG_TEC1, gNetworkServer)		
Switch landevice_KGTEC2AP		"Airport Keller TEC2"	<network> (KG_TEC2, gNetworkServer)
Switch landevice_DGOfficePrinter	"Drucker Dachgeschoss"	<network> (DG_GUEST, gNetworkMedia)
Switch landevice_KGOfficePrinter	"Drucker Keller"	<network> (KG_TEC1, gNetworkMedia)
Switch landevice_DGTVRasPlex		"RasPLEX Dachgeschoss"	<network> (DG_TV, gNetworkMedia)
Switch landevice_EGTVRasPlex		"RasPLEX Erdgeschoss"	<network> (EG_TV, gNetworkMedia)
Switch landevice_DGTVWii		"Wii"			<network> (DG_TV, gNetworkMedia)
Switch landevice_DadPC			"Dad PC"		<network> (gWZ, gNetworkDad)
Switch landevice_DadLaptop		"Dad Laptop"		<network> (gWZ, gNetworkDad)
Switch landevice_DadPhone		"Dad Phone"		<network> (gWZ, gDAD_PRESENT, gNetworkDad, gPhones)
Switch landevice_DadLaptopWork	        "Dad Laptop@work"	<network> (gWZ, gNetworkDad)
Switch landevice_DadPhoneWork	        "Dad Phone@work"	<network> (gWZ, gDAD_PRESENT, gNetworkDad, gPhones)
Switch landevice_MomLaptop		"Mom Laptop"		<network> (gWZ, gNetworkMom)
Switch landevice_MomPhone		"Mom Phone"		<network> (gWZ, gMOM_PRESENT, gNetworkMom, gPhones)
Switch landevice_MomLaptopWork	        "Mom Laptop@work"	<network> (gWZ, gNETWORKMom)
Switch landevice_MomPhoneWork	        "Mom Phone@work"	<network> (gWZ, gMOM_PRESENT, gNetworkMom, gPhones)
Switch landevice_GuestLaptop	        "Guest Laptop"		<network> (DG_GUEST, gNetworkGuest)
Switch landevice_GuestPhone		"Guest Phone"		<network> (DG_GUEST, gGUEST_PRESENT, gNetworkGuest, gPhones)
Switch landevice_GirlLaptop		"Girl Laptop"		<network> (DG_GIRL, gNetworkGirl)
Switch landevice_GirlPhone		"Girl Phone"		<network> (DG_GIRL, gGIRL_PRESENT, gNetworkGirl, gPhones)
Switch landevice_BoyLaptop		"Boy Laptop"		<network> (DG_BOY, gNetworkBoy)
Switch landevice_BoyPhone		"Boy Phone"		<network> (DG_BOY, gBOY_PRESENT, gNetworkBoy, gPhones)
Switch landevice_BoyiPod		"Boy iPod"		<network> (DG_BOY, gNetworkBoy)
```

As you can see the devices are assigned to various groups in the config, the interesting ones are the "_PRESENT" ones for the phones. Based on these with rules the status of each person's presence in the house can be defined. In my case when at least one phone per kid/guest is active in the network this person is seen as "present", for mom and dad both devices have to be present (on the weekends the work phones usually stay connected at home). Adding all devices here also offers some other capabilities... For example lower the download speed of the download server based on the number of active laptops in case the internet connection is not lighting fast._

### Final steps on the Linux server ###

Another test: Now execute one of the _curl_ statements created in the previous step on the Linux server in the shell:

```
ReadyNAS:~# curl --silent -H "Content-Type: text/plain" http://localhost:8080/rest/items/landevice_KGTEC2AP -d "ON"
ReadyNAS:~#
```

If the items were created correctly this should generate no output at all in the Linux shell, but show up promptly in the openHAB event log:

```
ReadyNAS:~# tail -200 /opt/openhab/logs/events.log
...
2013-11-13 22:47:25 - landevice_KGTEC2AP received command ON
...
```

Now create the final script _fritzbox\_devices.sh_ on the Linux server

```
#!/bin/sh
/root/fritzbox_devices.expect | /bin/fgrep "landevice=" | /root/fritzbox_devices.sed > /tmp/commands.txt
. /tmp/commands.txt
rm /tmp/commands.txt
```

and test this script again, this time using _time_ to avoid too many calls by cron:

```
ReadyNAS:~# time ./fritzbox_devices.sh
real    0m13.637s
user    0m0.267s
sys     0m0.202s
ReadyNAS:~#
```

Again, you should see no output in case you defined all items in openHAB. In case you did not errors will show up here and in the openHAB logs. A "clean" openHAB event.log outputs looks as follows:

```
ReadyNAS:~# tail -200f /opt/openhab/logs/events.log
...
2013-11-13 22:58:52 - landevice_BoyLaptop received command OFF
2013-11-13 22:58:52 - landevice_BoyPhone received command ON
2013-11-13 22:58:52 - landevice_BoyiPod received command OFF
2013-11-13 22:58:52 - landevice_DGOfficePrinter received command ON
2013-11-13 22:58:52 - landevice_DGTVRasPlex received command ON
2013-11-13 22:58:52 - landevice_DGTVWii received command OFF
2013-11-13 22:58:52 - landevice_DadLaptop received command OFF
2013-11-13 22:58:52 - landevice_DadLaptopWork received command OFF
2013-11-13 22:58:52 - landevice_DadLaptopWork received command OFF
2013-11-13 22:58:52 - landevice_DadPC received command ON
2013-11-13 22:58:52 - landevice_DadPhone received command ON
2013-11-13 22:58:52 - landevice_DadPhoneWork received command ON
2013-11-13 22:58:52 - landevice_EGEntranceAP received command ON
2013-11-13 22:58:52 - landevice_EGTVRasPlex received command ON
2013-11-13 22:58:52 - landevice_EGWZAP received command OFF
2013-11-13 22:58:52 - landevice_GirlLaptop received command OFF
2013-11-13 22:58:52 - landevice_GirlPhone received command ON
2013-11-13 22:58:52 - landevice_GuestLaptop received command OFF
2013-11-13 22:58:52 - landevice_KGOfficePrinter received command ON
2013-11-13 22:58:52 - landevice_KGOfficeReadyNAS received command ON
2013-11-13 22:58:52 - landevice_KGTEC1AP received command ON
2013-11-13 22:58:52 - landevice_KGTEC2AP received command ON
2013-11-13 22:58:52 - landevice_MomLaptop received command OFF
2013-11-13 22:58:52 - landevice_MomLaptop received command OFF
2013-11-13 22:58:52 - landevice_MomLaptopWork received command OFF
2013-11-13 22:58:52 - landevice_MomPhone received command OFF
...
```

When everything is working fine the cronjob can be set up, but avoid calling the script to frequently (thus the _time_ before). In this case we set it up to run each minute:

```
# m h  dom mon dow   command
* * * * * /root/fritzbox_devices.sh
```

### (Yet) incomplete snipplets with additional functionalities for the Fritz!Box ###

_wlandevices.sh_ is just dumping all devices connected directly to the wlan of the Fritz!Box (not those attached through LAN ports, but including the guest WLAN). For example to show devices connected to the guest WLAN with their remaining time in the openHAB GUI:

```
#!/bin/sh
i=0
count_devices=`ctlmgr_ctl r wlan settings/wlanlist/count`
while [ $i -lt $count_devices ]
do
device="wlandevice="$i
#for command in mac UID state speed rssi quality is_turbo is_guest is_ap ap_state mode wmm_active cipher powersave is_repeater channel freq flags flags_set
for command in mac state speed is_guest
do
output="`ctlmgr_ctl r wlan settings/wlanlist$i/$command`"
device=$device" "$command"="$output
done
echo $device
i=`expr $i + 1`
done
```

The output looks as follows and again can be adopted using the commented out for loop:

```
# ./wlandevices.sh
wlandevice=0 mac=AC:81:12:B0:39:F0 state=0 speed=0 is_guest=0
wlandevice=1 mac=20:02:AF:89:59:29 state=0 speed=0 is_guest=0
wlandevice=2 mac=18:E7:F4:09:A7:E4 state=0 speed=0 is_guest=0
wlandevice=3 mac=00:1F:1F:A2:AE:A6 state=5 speed=97 is_guest=0
wlandevice=4 mac=00:27:09:EA:D3:E9 state=0 speed=0 is_guest=0
wlandevice=5 mac=00:1C:B3:C3:5F:B3 state=0 speed=0 is_guest=0
wlandevice=6 mac=00:27:10:4F:3F:F0 state=0 speed=0 is_guest=0
wlandevice=7 mac=98:FE:94:39:AC:49 state=5 speed=41 is_guest=0
wlandevice=8 mac=54:44:08:A9:C4:F4 state=5 speed=106 is_guest=0
wlandevice=9 mac=80:1F:02:63:D3:32 state=5 speed=113 is_guest=0
wlandevice=10 mac=EC:55:F9:19:40:43 state=0 speed=0 is_guest=0
wlandevice=11 mac=64:66:B3:4B:A5:AE state=5 speed=65 is_guest=0
wlandevice=12 mac=64:66:B3:4B:C3:46 state=5 speed=38 is_guest=0
wlandevice=13 mac=B4:B6:76:AD:22:4E state=0 speed=0 is_guest=0
wlandevice=14 mac=58:1F:AA:AD:A6:E1 state=0 speed=0 is_guest=0
wlandevice=15 mac=00:80:92:97:3E:AB state=5 speed=64 is_guest=0
```

When you want to refer to device names, the mac addresses need to be added to the output of _landevices.sh_ above and the output of both scripts needs to be matched by mac address which should be easily possible.

Furthermore _wlanstatus.sh_ just dumps the status of the wireless networks of the Fritz!Box:

```
#!/bin/sh
for wlan in "" guest_
do
device=$wlan"wlan: "
for command in ap_enabled ssid pskvalue time_remain
do
output="`ctlmgr_ctl r wlan status/$wlan$command`"
device=$device" "$command"="$output
done
echo $device
done
```

The output looks as follows:

```
# ./wlanstatus.sh
wlan: ap_enabled=1 ssid=wlanssid pskvalue=unencryptedpassword time_remain=
guest_wlan: ap_enabled=0 ssid=guestssid pskvalue=unencryptedpassword time_remain=0
```

This script might be used together with another (yet to do) guest wlan on/off script to offer kids the possibility to enable/disable the guest wlan with a randomly generated password (eg  _echo `tr -dc A-Za-z0-9_ < /dev/urandom | head -c 16`_) in order to not have to do this yourself each time.


## How to configure openHAB to connect to device symlinks (on Linux) ##
When connecting serial devices to Linux (i.e. USB) they are assigned arbitrary device names - usually something like /dev/ttyUSB0. There is no way to know exactly what name will be assigned each time you plug in the device however.

Using udev rules you can setup symlinks which are created when the system detects a device is added. The symlink allows you to assign a 'known' name for that device. See this link for a useful tutorial about udev - http://www.reactivated.net/writing_udev_rules.html.

So for example, if you have a ZWave USB dongle you can configure a symlink called /dev/zwave. This makes it much easier to configure connection properties since this name will never change.

However the RXTX serial connection library used in openHAB is unable to 'see' these symlinks. Therefore you need to inform RXTX about your symlinks using a system property.

You can either add the property to the Java command line by adding the following (device names delimited by :);

`-Dgnu.io.rxtx.SerialPorts=/dev/rfxcom:/dev/zwave`

Or add a gnu.io.rxtx.properties file which is accessible in the Java classpath. See http://create-lab-commons.googlecode.com/svn/trunk/java/lib/rxtx/README.txt for more details.

This applies to any openHAB binding that uses the RXTX serial connection library - i.e. RFXCOM, ZWave etc.


## Use URL to manipulate items ##

When you have a device unable to send REST requests (f.e. Webcams), you may use
` http://<openhab-host>:8080/CMD?<item>=<state> `

Example:
` http://<openhab-host>:8080/CMD?Light=ON `

## Extract caller and called number from Fritzbox Call object ##

If you define an item like the following in your site.items config

`Call	Incoming_Call_No  "Caller No. [%2$s]"  (Phone)  { fritzbox="inbound" } `

the Type CallType with its associated methods can be used to extract the caller and called number of the call. The following rule sends an email with both numbers in the subject line.

```
import org.openhab.library.tel.types.CallType

rule "inbound Call"
  when Item Incoming_Call_No received update
then 
  val CallType call = Incoming_Call_No.state as CallType
  val String mailSubject =
  "Anruf von Nummer " + call.origNum + " -> " + call.destNum
  sendMail( "us...@domain.de" , mailSubject , "");
end


```

## Item loops with delay ##

Some integrations don't like to be rushed with a cascade of simultaneous commands. This can be an issue if you, for instance want to implement a rule which loops through a list of group members (items) and send a command to all of them, e. g. OFF. The solution is to add a small delay between each command. The example below executes OFF for all members at a one second interval. Use `now.plusMillis(i*`_n_`)` for _n_ millisecond intervals.

```
rule "Lights out"
when 
	Time cron "0 0 22 ? * MON-THU,SUN *" or
	Time cron "0 59 23 ? * FRI,SAT *"
then 
	gLightOffNight?.members.forEach(item,i|createTimer(now.plusSeconds(i)) [|sendCommand(item, OFF)])
end
```