# Scripts #

Some useful Scripts for openHAB


## Simple WakeUpLight ##

With this script you can simulate a simple WakeUP light. Just call it via a rule at a specified time
```
var Number wakeUpDimmer

wakeUpDimmer=0
while(wakeUpDimmer<100){
	wakeUpDimmer=wakeUpDimmer+5
	sendCommand(TestDimmer1,wakeUpDimmer)
	Thread::sleep(20000)
}
```
Just replace the item TestDimmer1 with your dimmer item. This script will increase the brightness by 5% every 20 seconds.