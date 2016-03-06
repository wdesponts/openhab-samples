

## How to turn on light when motion detected and is dark? ##
Light is turned on when there is motion detected (corMotion) and brightness is below threshold. Every 1min it is checked whether there was motion since the last check. If not: turn light back off.
```
var Number counter = 0
var Number lastCheck = 0

rule "corLightOn"
when   
        Item corMotion changed from OFF to ON or
        Item corFrontDoor changed from CLOSED to OPEN
then   
        counter = counter + 1
        if(corBright.state < 40) {
                sendCommand(corLightCeil, ON)
        }
end

rule "corLightOff"
when   
        Time cron "0 * * * * ?"
then   
        if(lastCheck == counter) {
                counter = 0
                lastCheck = -1;
                sendCommand(corLightCeil, OFF)
                sendCommand(corMotion, OFF)
        } else {
                lastCheck = counter
        }
end

```

## How to create a rule, which only executes some code, if a value does not change for a certain period of time ##
```
var Timer timer

rule "do something if item state is 0 for more than 10 seconds"
when
	Item MyItem changed
then
	if(MyItem.state==0) {
		timer = createTimer(now.plusSeconds(10)) [|
			// do something! 
		]
	} else {
		if(timer!=null) {
			timer.cancel
			timer = null
		}
	}
end
```

## How to calculate the sun position ##

```
import org.openhab.core.library.types.*
import java.lang.Math


// Constants
var Number K = 0.017453

// Change this reflecting your destination
var Number latitude = xx.xxxxxx
var Number longitude = yy.yyyyyy


rule "Set Sun and Dawn States"
when
    Time cron "0 0/5 * * * ?"
then
	var Number tageszahl
	var Number deklination
	var Number zeitgleichung
	var Number stundenwinkel
	var Number x 
	var Number y
	var Number sonnenhoehe
	var Number azimut
	
	var month  = now.getMonthOfYear
	var day    = now.getDayOfMonth
	var hour   = now.getHourOfDay
	var minute = now.getMinuteOfHour
	
	// Source: http://www.jgiesen.de/SME/tk/index.htm
	tageszahl = (month - 1) * 30 + day + hour / 24 
	deklination = -23.45 * Math::cos((K * 360 * (tageszahl + 10) / 365).doubleValue)
	zeitgleichung = 60.0 * (-0.171 * Math::sin((0.0337*tageszahl+0.465).doubleValue) - 0.1299 * Math::sin((0.01787*tageszahl-0.168).doubleValue))
	stundenwinkel = 15.0 * (hour.doubleValue + (minute.doubleValue/60.0) - (15.0-longitude)/15.0 - 12.0 + zeitgleichung/60.0)
	x = Math::sin((K * latitude).doubleValue()) * Math::sin((K * deklination).doubleValue()) + Math::cos((K * latitude).doubleValue()) * Math::cos((K * deklination).doubleValue()) * Math::cos((K * stundenwinkel).doubleValue())
	y = - (Math::sin((K*latitude).doubleValue) * x - Math::sin((K*deklination).doubleValue)) / (Math::cos((K*latitude).doubleValue) * Math::sin(Math::acos(x.doubleValue)))
	sonnenhoehe = Math::asin(x.doubleValue) / K
	
	var break = hour.doubleValue + (minute.doubleValue/60.0) <= 12.0 + (15.0-longitude)/15.0 - zeitgleichung/60.0 
	if (break) {
		azimut = Math::acos(y.doubleValue) / K
	} else {
		azimut = 360.0 - Math::acos(y.doubleValue) / K
	}
	 
	logDebug("Sun.rules", "month: " + month)
	logDebug("Sun.rules", "day: " + day)
	logDebug("Sun.rules", "hour: " + hour)
	logDebug("Sun.rules", "minute: " + minute)
	logDebug("Sun.rules", "tageszahl: " + tageszahl)
	logDebug("Sun.rules", "deklination: " + deklination)
	logDebug("Sun.rules", "zeitgleichung: " + zeitgleichung)
	logDebug("Sun.rules", "stundenwinkel: " + stundenwinkel)
	logDebug("Sun.rules", "x: " + x)
	logDebug("Sun.rules", "y: " + y)
	logDebug("Sun.rules", "sonnenhoehe: " + sonnenhoehe)
	logDebug("Sun.rules", "azimut: " + azimut)
	
	logDebug("Sun.rules", "Calculated new SunHeight angle '" + sonnenhoehe + "°'")
	logDebug("Sun.rules", "Calculated new Azimut angle '" + sonnenhoehe + "°'")
	
	// Send all calculations to the event bus
	
	Sun_Height.postUpdate(sonnenhoehe)
	Sun_Azimut.postUpdate(azimut)
	
	Sun_Dawn_Solar.postUpdate( if (sonnenhoehe < 0) ON else OFF )
	
	postUpdate(Sun_Dawn_Civil, 	  if((sonnenhoehe <   0)  && (sonnenhoehe >=  -6) && (hour < 12)) {ON} else {OFF})
	postUpdate(Sun_Dawn_Nautical,	  if((sonnenhoehe <  -6)  && (sonnenhoehe >= -12) && (hour < 12)) {ON} else {OFF})
	postUpdate(Sun_Dawn_Astronomical, if((sonnenhoehe < -12)  && (sonnenhoehe >= -18) && (hour < 12)) {ON} else {OFF})

	postUpdate(Sun_Dusk_Civil,	  if((sonnenhoehe <   0)  && (sonnenhoehe >=  -6) && (hour > 12)) {ON} else {OFF})
	postUpdate(Sun_Dusk_Nautical,	  if((sonnenhoehe <  -6)  && (sonnenhoehe >= -12) && (hour > 12)) {ON} else {OFF})
	postUpdate(Sun_Dusk_Astronomical, if((sonnenhoehe < -12)  && (sonnenhoehe >= -18) && (hour > 12)) {ON} else {OFF})

end
```

## How to calculate public holidays ##

Items:
```
Switch Holiday					"Public Holiday"
String SpecialDay				"Day [MAP(holidays_de.map):%s]"
```

Transformation (holidays\_de.map):
```
undefined=undefiniert
new_years_day=Neujahr
holy_trinity=Heilige 3 Könige
carnival_monday=Rosenmontag
good_friday=Karfreitag
easter_sunday=Ostersonntag
easter_monday=Ostermontag
labor_day=Tag der Arbeit
ascension_day=Christi Himmelfahrt
whit_sunday=Pfingstsonntag
whit_monday=Pfingstmontag
corpus_christi=Fronleichnahm
assumption_day=Mariä Himmelfahrt
reunification=Tag der deutschen Einheit
reformation_day=Reformationstag
all_saints_day=Allerheiligen
remembrance_day=Volkstrauertag
day_of_repentance=Buß- und Bettag
sunday_in_commemoration_of_the_dead=Totensonntag
christmas_eve=Heiligabend
1st_christmas_day=1. Weihnachtstag
2nd_christmas_day=2. Weihnachtstag
new_years_eve=Silvester
```

Rule:
```
rule "Public Holiday"
when
    Time cron "0 0 0 * * ?" or
    System started
then
    callScript("holiday")
end
```

Script (holiday.script):
```
var int year = now.getYear

var int a = year % 19
var int b = year / 100
var int c = year % 100
var int d = b / 4;
var int e = b % 4;
var int f = (b + 8) / 25;
var int g = (b - f + 1) / 3;
var int h = (19 * a + b - d - g + 15) % 30;
var int i = c / 4;
var int k = c % 4;
var int L = (32 + 2 * e + 2 * i - h - k) % 7;
var int m = (a + 11 * h + 22 * L) / 451;

var int month = (h + L - 7 * m + 114) / 31;
var int day = ((h + L - 7 * m + 114) % 31) + 1;

var boolean holiday = false
var String holidayName = null
var org.joda.time.DateTime easterSunday = parse(year+"-"+month+"-"+day)
var org.joda.time.DateTime stAdvent = parse(year+"-12-25").minusDays(((parse(year+"-12-25").getDayOfWeek) + 21))

var int dayOfYear = now.getDayOfYear
// bundesweiter Feiertag
if (dayOfYear==parse(year+"-01-01").getDayOfYear) {
    holidayName = "new_years_day" // Neujahr
    holiday = true
}
// Baden-Württemberg, Bayern, Sachsen-Anhalt
else if (dayOfYear==parse(year+"-01-06").getDayOfYear) {
    holidayName = "holy_trinity"// Heilige 3 Könige
    holiday = false 
}
// Carnival ;-)
else if (dayOfYear==easterSunday.getDayOfYear-48) {
    holidayName = "carnival_monday" // Rosenmontag
    holiday = false
}
// bundesweiter Feiertag
else if (dayOfYear==easterSunday.getDayOfYear-2) {
    holidayName = "good_friday" // Karfreitag
    holiday = true
}
// Brandenburg
else if (dayOfYear==easterSunday.getDayOfYear) {
    holidayName = "easter_sunday" // Ostersonntag
    holiday = false
}
// bundesweiter Feiertag
else if (dayOfYear==easterSunday.getDayOfYear+1) {
    holidayName = "easter_monday" // Ostermontag
    holiday = true
}
// bundesweiter Feiertag
else if (dayOfYear==parse(year+"-05-01").getDayOfYear) {
    holidayName = "1st_may"// Tag der Arbeit
    holiday = true 
}
// bundesweiter Feiertag
else if (dayOfYear==easterSunday.getDayOfYear+39) {
    holidayName = "ascension_day" // Christi Himmelfahrt
    holiday = true
}
// Brandenburg
else if (dayOfYear==easterSunday.getDayOfYear+49) {
    holidayName = "whit_sunday" // Pfingstsonntag
    holiday = false
}
// bundesweiter Feiertag
else if (dayOfYear==easterSunday.getDayOfYear+50) {
    holidayName = "whit_monday" // Pfingstmontag
    holiday = true
}
// Baden-Württemberg, Bayern, Hessen, NRW, Rheinland-Pfalz, Saarland sowie regional in Sachsen, Thüringen
else if (dayOfYear==easterSunday.getDayOfYear+60) {
    holidayName = "corpus_christi" // Frohnleichnahm
    holiday = true
}
// Saarland sowie regional in Bayern
else if (dayOfYear==parse(year+"-08-15").getDayOfYear) {
    holidayName = "assumption_day" // Mariä Himmelfahrt
    holiday = false
}
// bundesweiter Feiertag
else if (dayOfYear==parse(year+"-10-03").getDayOfYear) {
    holidayName = "reunification" // Tag der deutschen Einheit
    holiday = true
}
// Brandenburg, Mecklenburg-Vorpommern, Sachsen, Sachsen-Anhalt, Thüringen
else if (dayOfYear==parse(year+"-10-31").getDayOfYear) {
    holidayName = "reformation_day" // Reformationstag
    holiday = false
}
// Baden-Württemberg, Bayern, NRW, Rheinland-Pfalz, Saarland
else if (dayOfYear==parse(year+"-11-01").getDayOfYear) {
    holidayName = "all_saints_day" // Allerheiligen
    holiday = true
}
// religiöser Tag
else if (dayOfYear==stAdvent.getDayOfYear-14) {
    holidayName = "remembrance_day" // Volkstrauertag
    holiday = false
}
// religiöser Tag
else if (dayOfYear==stAdvent.getDayOfYear-7) {
    holidayName = "sunday_in_commemoration_of_the_dead" // Totensonntag
    holiday = false
}
// Sachsen
else if (dayOfYear==stAdvent.getDayOfYear-11) {
    holidayName = "day_of_repentance" // Buß- und Bettag
    holiday = false
}
// kann auch der 4te Advent sein
else if (dayOfYear==parse(year+"-12-24").getDayOfYear) {
    holidayName = "christmas_eve" // Heiligabend
    holiday = false
}
// bundesweiter Feiertag
else if (dayOfYear==parse(year+"-12-25").getDayOfYear) {
    holidayName = "1st_christmas_day" // 1. Weihnachtstag
    holiday = true
}
// bundesweiter Feiertag
else if (dayOfYear==parse(year+"-12-26").getDayOfYear) {
    holidayName = "2nd_christmas_day" // 2. Weihnachtstag
    holiday = true
}
// Silvester
else if (dayOfYear==parse(year+"-12-31").getDayOfYear) {
    holidayName = "new_years_eve" // Silvester
    holiday = false
}
if (holidayName!=null) {
    postUpdate(SpecialDay,holidayName)
}
if (holiday) {
    postUpdate(Holiday,ON)
}
else {
    postUpdate(Holiday,OFF)
}
```

## Create a timer for sunset ##
This example gets the sunset from an online api service (in this case www.wunderground.com). The sunset time is parsed into a timer to switch on lights (or any other action)

**Items**:
```
String strSunset "Sunset" <clock> (gWeather) { http="<[http://api.wunderground.com/api/<api_code>/astronomy/q/NL/rijswijk.xml:21600000:XSLT(wunderground_sunset.xsl)]" }
```
Replace 

<api\_code>

 with your own API code (sign up for a developers account). Find a location near your home. The http refresh option can be set to once or twice a day.

**Stylesheet** (wunderground\_sunset.xsl)
```
<?xml version="1.0"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="2.0">
	
	<xsl:output indent="yes" method="xml" encoding="UTF-8" omit-xml-declaration="yes" />

	<xsl:template match="/">
		<!-- format: hh:mm:ss -->
		<xsl:value-of select="//sunset/hour/text()" /><xsl:text>:</xsl:text><xsl:value-of select="//sunset/minute/text()" /><xsl:text>:00</xsl:text>
	</xsl:template>

</xsl:stylesheet>
```

**Rules**
```
import org.joda.time.*

var Timer tIndoorLights

rule "React to sunset"
when 
	Time cron "0 0 16 * * ?"   // Every day 16:00 hours, evaluate sunset
then
    var year   = now.getYear
    var month  = now.getMonthOfYear
    var day    = now.getDayOfMonth
    var datum  = year+"-"+month+"-"+day+" "+strSunset.state
    logInfo("Sunset","datum = " + datum)
    var DateTime sunset = parse(year+"-"+month+"-"+day+"T"+strSunset.state)
    
    /*
     * Indoor Lights
     */
     // Cancel timer to avoid reschedule
    if(tIndoorLights!=null) {
	logInfo("Sunset","Timer tIndoorLights cancelled") 
	tIndoorLights.cancel()
    }
    logInfo("Sunset","Timer tIndoorLights created") 
    tIndoorLights = createTimer(sunset.minusMinutes(15)) [|
	logInfo("Sunset","Timer tIndoorLights executed") 
	gSunset?.members.forEach(Switch|
		sendCommand(Switch, ON)
	)
    ]
end
```

## Send an image from your webcam by e-mail ##

If your webcam offers a still picture in your local LAN e.g. at ![http://192.168.0.2/jpg/image.jpg](http://192.168.0.2/jpg/image.jpg), you can simply send it as an attachment in a rule by e-mail with a line like:
```
sendMail("me@domain.com", "Frontdoor", "There is motion at the frontdoor", "http://192.168.0.2/jpg/image.jpg")
```

## How to display the minimum and maximum values of an item in a given period ##
This example displays the minimum and maximum values and the time they where measured of the item "Temperature\_Garden" for the current day. Other periods can be selected by modifying the parameter of minimumSince() and maximuSince() in the rule below.

**Items**:
```
Number Temperature_Garden "Garden [%.1f °C]" <temperature> ( gGarden) 	{ knx="9.001:1/2/3" }
String Temperature_Garden_Min_Formatted "- Min. Temp [%s]" <temperature>
String Temperature_Garden_Max_Formatted "- Max. Temp [%s]" <temperature>
```

The string items hold the formatted String to be displayed in the UI. The number item holds the current temperature, it must be updated periodically and persistence must be configured for this item. This example assumes, that the rrd4h persistence service is used, for other services the string "rrd4j" in the rule below must be modified accordingly.


**Rules**
```
rule "Update Temperature Min- and Max values"
when
	Item  Temperature_Garden received update
then
	var Number Min
	var Number Max
	var String tmp
	var SimpleDateFormat df = new SimpleDateFormat( "HH:mm" ) 
 	
 	if (Temperature_Garden.state instanceof DecimalType) {
		Min = (Temperature_Garden.minimumSince(now.toDateMidnight, "rrd4j").state as DecimalType)
		tmp = (Math::round(Min.floatValue*10.0)/10.0) + " °C (" + df.format(Temperature_Garden.minimumSince(now.toDateMidnight, "rrd4j").timestamp) + " )"
		postUpdate(Temperature_Garden_Min_Formatted, tmp)
		
		Max = Temperature_Garden.maximumSince(now.toDateMidnight, "rrd4j").state as DecimalType
		df = new SimpleDateFormat( "HH:mm" ) 
		tmp = (Math::round(Max.floatValue*10.0)/10.0) + " °C (" + df.format(Temperature_Garden.maximumSince(now.toDateMidnight, "rrd4j").timestamp) + ")"
		postUpdate(Temperature_Garden_Max_Formatted, tmp)
	}
end
```

**Includes**
```
import org.openhab.core.library.types.*
import org.openhab.model.script.actions.*
import java.lang.Math
import java.util.Calendar
import java.util.Date
import java.util.TimeZone
import java.text.SimpleDateFormat
```

The rules file probably needs the includes listened above.

## How to use Colorpicker widget with KNX/DALI RGB LED STRIPE ##

The openHAB Colorpicker widget was primarily developed for being used with the DMX binding. The following example shows how to make it work with DALI (or any other technology) via KNX.

items.all
```
Dimmer LedR     "LED Red"               <dimmer>        (All) { knx = "11/0/0+11/0/1,   11/0/2,  11/0/3+11/0/4" }
Dimmer LedG     "LED Green"             <dimmer>        (All) { knx = "11/0/5+11/0/6,   11/0/7,  11/0/8+11/0/9" }
Dimmer LedB     "LED Blue"              <dimmer>        (All) { knx = "11/0/10+11/0/11, 11/0/12, 11/0/13+11/0/14" }
Color  RGB      "RGB Light"             <slider>
```
Items `LedR`, `LedG`, `LedB` are mapped in DALI-GW to respective light groups with KNX group addresses.

Explanation of group addresses in above example:
| Group Address | Description |
|:--------------|:------------|
| 11/0/0        | ON/OFF for red channel |
| 11/0/1        | Current status (ON/OFF) for red channel |
| 11/0/2        | Dim value for red channel |
| 11/0/3        | Percentage value for the red channel for update (writing) |
| 11/0/4        | Current percentage value for the red channel (reading) |

The item `RGB` is the object to store the current percentage values for each channel (R, G, B).

sitemap.all
```
sitemap all label="Main Menu"
{
        Frame label="RGB"
        {
                Colorpicker item=RGB
        }
}
```

RGB.rules
```
import org.openhab.core.library.types.*

var HSBType hsbValue
var String  redValue
var String  greenValue
var String  blueValue

rule "Set RGB value"
when
        Item RGB changed
then
        hsbValue = RGB.state as HSBType

        redValue   = hsbValue.red.intValue.toString
        greenValue = hsbValue.green.intValue.toString
        blueValue  = hsbValue.blue.intValue.toString

        sendCommand( LedR, redValue )
        sendCommand( LedG, greenValue )
        sendCommand( LedB, blueValue )
end
```

Any change in value of the RGB Colorpicker item fires this rule. The HSB value of the item is determined and split to percentage values for red, green and blue which then get sent to the individual KNX items LedR, LedG and LedB.

## How to log current timestamp to the openHAB log file ##

mytimestamp.rules
```
import org.openhab.core.library.types.*
import java.util.Date
import java.text.SimpleDateFormat

rule "MyTimeSTamp"
when
        // Any condition
then
        var SimpleDateFormat df = new SimpleDateFormat( "YYYY-MM-dd HH:mm:ss" )
        var String Timestamp = df.format( new Date() )

        logInfo( "FILE", Timestamp )
end
```

or, alternatively, if you rely on String Formatter only (e.g. in Persistence files):

```
import org.openhab.core.library.types.*
import java.util.Date
import java.text.SimpleDateFormat

rule "MyTimeSTamp"
when
        // Any condition
then
        var String Timestamp = String::format( "%1$tY-%1$tm-%1$td %1$tH:%1$tM:%1$tS", new Date() )
        logInfo( "FILE", Timestamp )
end
```

If you wonder, how to use the String::format option:<br />
The syntax of the format method is format( `<String>`pattern, `<object>`obj1, `<object>`obj2 `[,...]`).

In the pattern String, use "%1" for parsing the first object, "%2" for the second object a.s.o.

e.g. pattern "%1$tY" refers to the first object passed into the format method after the pattern string and treats the input as a (t)imestamp extracting the two-digit (Y)ear.

For more information on how to use the format method please see the [Java documentation](http://docs.oracle.com/javase/6/docs/api/java/util/Formatter.html).

## Irrigation controller ##

This assumes you have some way of turning your irrigation zones on/off. I use a 4 way relay board connected to the GPIO pins on a Raspberry Pi running my PiFace binding. Note: by using an external relay board I don't need the PiFace extension board but you could use one if you only had 2 zones (since the PiFace only has two relays).

The items/rules defined below allow you to configure how long each zone is 'ON' for and then apply a scaling factor to all zones. I.e. you might decide each zone needs 30mins, but in winter scale this back by 50%.

I haven't worked out a way to allow the 'start time' to be edited in the openHAB UI yet - anyone got any bright ideas?

The last rule will disable the irrigation system if there is any rain detected in the last 24 hours, or any rain in the forecast.

Items
```
Group 		Irrigation
Group 		Weather

Switch 		Irrigation_Master		"Irrigation Master"			<power>			(Irrigation)
String		Irrigation_StartTime	"Start Time [%s]"			<calendar>		(Irrigation)
Number		Irrigation_ScaleFactor	"Scale Factor [%d %%]"		<water>			(Irrigation)

Number		Irrigation_LawnMins		"Lawn Sprinkler [%d mins]"	<water>			(Irrigation)
Number		Irrigation_VegeMins		"Vege Patch [%d mins]"		<water>			(Irrigation)
Number		Irrigation_BackMins		"Back Garden [%d mins]"		<water>			(Irrigation)
Number		Irrigation_FrontMins	"Front Garden [%d mins]"	<water>			(Irrigation)

Switch 		Irrigation_Lawn			"Lawn Sprinkler"			<water>			(Irrigation)	{ piface="irrigation:OUT:11" }
Switch 		Irrigation_Vege			"Vege Patch"				<water>			(Irrigation)	{ piface="irrigation:OUT:12" }
Switch 		Irrigation_Back			"Back Garden"				<water>			(Irrigation)	{ piface="irrigation:OUT:13" }
Switch 		Irrigation_Front		"Front Garden"				<water>			(Irrigation)	{ piface="irrigation:OUT:15" }

Number		Weather_Rain				"Rainfall [%.1f mm]"			<rain>   		(Weather)
String 		Weather_TodayIcon			"Today [%s]"        			<w>  			(Weather)				{ http="<[http://api.wunderground.com/api/<YOUR_API_KEY>/forecast/q/pws:<YOUR_PWS_ID>.xml:3600000:XSLT(<YOUR_TRANSFORM>.xsl)]" } 
String 		Weather_TomorrowIcon		"Tomorrow [%s]"     			<w>  			(Weather) 				{ http="<[http://api.wunderground.com/api/<YOUR_API_KEY>/forecast/q/pws:<YOUR_PWS_ID>.xml:3600000:XSLT(<YOUR_TRANSFORM>.xsl)]" } 
```

Rules
```
import org.openhab.core.library.types.*
import org.openhab.core.persistence.*
import org.openhab.model.script.actions.*

import org.joda.time.*

rule "Irrigation startup"
when
	System started
then
	postUpdate(Irrigation_StartTime, "03:00")

	sendCommand(Irrigation_Lawn, OFF)
	sendCommand(Irrigation_Vege, OFF)
	sendCommand(Irrigation_Back, OFF)
	sendCommand(Irrigation_Front, OFF)
end

rule "Irrigation run"
when
    Time cron "0 0 0 * * ?"
then
	if (Irrigation_Master.state == ON) {
		// get the scale factor - used to reduce the run times across the board
		var Number scaleFactor = Irrigation_ScaleFactor.state as DecimalType

	    // convert our start time to a joda.time.DateTime for today
    	var DateTime startTime = parse(now.getYear() + "-" + now.getMonthOfYear() + "-" + now.getDayOfMonth() + "T" + Irrigation_StartTime.state + ":00")
    	var DateTime endTime		

		// get the raw run times for each zone (in mins)
		var Number lawnMins = Irrigation_LawnMins.state as DecimalType
		var Number vegeMins = Irrigation_VegeMins.state as DecimalType
		var Number backMins = Irrigation_BackMins.state as DecimalType
		var Number frontMins = Irrigation_FrontMins.state as DecimalType
		
		// convert to the actual run times (by applying the scale factor)
		var int lawnTime = ((lawnMins * scaleFactor) / 100).intValue
		var int vegeTime = ((vegeMins * scaleFactor) / 100).intValue
		var int backTime = ((backMins * scaleFactor) / 100).intValue
		var int frontTime = ((frontMins * scaleFactor) / 100).intValue
		
		// turn on each zone in turn (with a minute gap between each zone activation)
		if (lawnTime > 0) {
			endTime = startTime.plusMinutes(lawnTime)
			createTimer(startTime) [| sendCommand(Irrigation_Lawn, ON) ]
			createTimer(endTime) [| sendCommand(Irrigation_Lawn, OFF) ]
			startTime = endTime.plusMinutes(1)
		}
		
		if (vegeTime > 0) {
			endTime = startTime.plusMinutes(vegeTime)
			createTimer(startTime) [| sendCommand(Irrigation_Vege, ON) ]
			createTimer(endTime) [| sendCommand(Irrigation_Vege, OFF) ]			
			startTime = endTime.plusMinutes(1)
		}
		
		if (backTime > 0) {
			endTime = startTime.plusMinutes(backTime)
			createTimer(startTime) [| sendCommand(Irrigation_Back, ON) ]
			createTimer(endTime) [| sendCommand(Irrigation_Back, OFF) ]
			startTime = endTime.plusMinutes(1)
		}
		
		if (frontTime > 0) {
			endTime = startTime.plusMinutes(frontTime)
			createTimer(startTime) [| sendCommand(Irrigation_Front, ON) ]
			createTimer(endTime) [| sendCommand(Irrigation_Front, OFF) ]
			startTime = endTime.plusMinutes(1)
		}
	}	
end

rule "Disable irrigation if any rain"
when
	Item Weather_Rain changed or
	Item Weather_TodayIcon changed or
	Item Weather_TomorrowIcon changed
then
    // the rainfall threshold where we shutdown off irrigation
    var rainThreshold = 1

	// check for any rain in the last 24 hours
    var rainInLast24Hours = Weather_Rain.maximumSince(now.minusHours(24), "rrd4j")
	
	// default to the current rain value in case there is nothing in our history
	var rain = Weather_Rain.state
	
    if (rainInLast24Hours != null)
    	rain = rainInLast24Hours.state
    	
    // check if any rain is forecast
    var rainToday = Weather_TodayIcon.state == "chanceflurries" ||
    				Weather_TodayIcon.state == "chancerain" ||
    				Weather_TodayIcon.state == "chancesleet" ||
    				Weather_TodayIcon.state == "chancesnow" ||
    				Weather_TodayIcon.state == "chancetstorms" ||
    				Weather_TodayIcon.state == "flurries" ||    				
    				Weather_TodayIcon.state == "rain" || 
    				Weather_TodayIcon.state == "sleet" ||
    				Weather_TodayIcon.state == "snow" ||
    				Weather_TodayIcon.state == "tstorms"
    				
    var rainTomorrow = Weather_TomorrowIcon.state == "chanceflurries" ||
    				   Weather_TomorrowIcon.state == "chancerain" ||
    				   Weather_TomorrowIcon.state == "chancesleet" ||
    				   Weather_TomorrowIcon.state == "chancesnow" ||
    				   Weather_TomorrowIcon.state == "chancetstorms" ||
    				   Weather_TomorrowIcon.state == "flurries" ||
    				   Weather_TomorrowIcon.state == "rain" || 
    				   Weather_TomorrowIcon.state == "sleet" ||
    				   Weather_TomorrowIcon.state == "snow" ||
    				   Weather_TomorrowIcon.state == "tstorms"
       	
	// shutoff irrigation if there has been rain or rain is forecast
    var logMessage = ""
	if (rain > rainThreshold) {
	    logMessage = "Rain in the last 24 hours (" + rain + " mm) is above our threshold (" + rainThreshold + " mm) - irrigation disabled!"
	} else if (rainToday) {
	    logMessage = "Rain is forecast for today - irrigation disabled!"
	} else if (rainTomorrow) {
		logMessage = "Rain is forecast for tomorrow - irrigation disabled!"
	}
	
	if (logMessage != "") {
		if (Irrigation_Master.state == ON) {
        	logInfo("Irrigation", logMessage)
			send("ben.jones12@gmail.com", logMessage)
			postUpdate(Irrigation_Master, OFF)
		}
	} else {
		if (Irrigation_Master.state == OFF) {
	        logInfo("Irrigation", "No rain in the last 24 hours or any rain forecast - irrigation enabled!")
			send("ben.jones12@gmail.com", "No rain in the last 24 hours or any rain forecast - irrigation enabled!")
			postUpdate(Irrigation_Master, ON)
		}
	}
end
```