# Frequently Asked Questions #



## How to use a serial port under linux ##

If you are usiung a usb-to-serial-adapter your serial-interface might be /dev/ttyUSB0.

Check the permissions of this device with
```
user@computer:~$ ls -l /dev/ttyUSB0
crw-rw---- 1 root dialout 188, 0 Aug 11 14:16 /dev/ttyUSB0
```
In order to access this device the user running openHAB needs to be in the group "dailout".

```
sudo adduser <user> dialout
```

Reboot your system.

## openHAB and the Cubieboard, what you need to knwo ##
Running openHAB on a Cubieboard(2) is possible. But there are some kinks to figure out.
First you have to choose a distribution to install. At first I settled for Cubian, since I was already familiar with Raspbian. But Cubian comes only with very few kernel modules for USB to serial bridges. Since many bindings communicate that way, this was a dealbreaker. Currently I run lubuntu-server-13.06-v1.00, which has all needed kernel modules. Simply install this distribution to the NAND memory (see http://cubiebook.org/ for details).
Another problem could be the JDK8. While it was running fine on the Raspberry Pi I had several unexplained crashes of openHAB with the early preview of JDK8. Luckily Oracle released the JDK7 for hard floar arm processors a while ago. Just grab it from here http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html unpack it and use it to run openHAB just like on any other platform.