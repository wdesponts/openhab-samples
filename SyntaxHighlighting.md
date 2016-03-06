# Syntax highlighting for external editors #

OpenHAB has a great development environment with the "openHAB Designer".
But in some cases you want to use another editor to make changes to the configuration files of openHAB.
To make this more effectiv there are some files to enable syntax highlighting for openHAB-files in these editors.



## mcedit ##
mcedit is an editor which comes with mc (Midnight Commander).


### Installing the syntax-files ###


  * download copy the syntax-files to _/usr/share/mc/syntax/_
    * https://code.google.com/p/openhab-samples/source/browse/syntaxhl/mc/?repo=wiki

  * insert the following lines to the file _Syntax_ in _/usr/share/mc/syntax/_
```
file ..\*\\.(items)$ openHAB\sItems 
include openhab-items.syntax  
 
file ..\*\\.(sitemap)$ openHAB\sSitemap 
include openhab-sitemap.syntax
 
file ..\*\\.(persist)$ openHAB\sPersistence
include openhab-persist.syntax
 
file ..\*\\.(rules)$ openHAB\sRules
include openhab-rules.syntax 
```
  * edit the Debian-line from
```
file (rules|rocks)$ Debian\srules
```
> to
```
file (rocks)$ Debian\srules
```
> because it interferes with openHABs rules-files.

### Screenshots ###

![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_items.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_items.png)
![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_rules.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_rules.png)
![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_sitemap.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_sitemap.png)

## Notepad++ ##

Notepad++ Version 6.2 or above is required to support UDL2 (User Defined Language v2).

http://notepad-plus-plus.org/news/notepad-6.2-release-udl2.html

### How to import UDL2-files? ###

  * Download the UDL2-Files (openHAB-`*`.xml)
    * https://code.google.com/p/openhab-samples/source/browse/syntaxhl/npp/?repo=wiki
  * Install or update Notepad++ if necessary
    * http://notepad-plus-plus.org/download/
  * Open Notepad++
  * Click "Language" (1)
  * Click "Define your language.." (2)
  * Click "Import..." (3)
  * Select one of the downloaded XML-files
  * Done.

![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_import_udl2.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_import_udl2.png)


### Screenshots ###

![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_items.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_items.png)
![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_rules.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_rules.png)
![http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_sitemap.png](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_sitemap.png)

## vi ##

feel free to contribute..