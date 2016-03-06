# Introduction #

If you are experimenting with the latest openHAB-snapshots, it can be quit annoying to download all nightly-packages, extract them, update all files and addons and copy/replace the configuration files, every time.


To automate this process there are two shell-scripts for Linux available.

## Script 1 ##

the script [openhab\_get\_snapshot.sh](http://code.google.com/p/openhab-samples/source/browse/scripts/openhab_get_snapshot.sh?repo=wiki) is a quick-and-dirty-script to download a new openHAB Snapshot to a new folder.

Usage is `openhab_get_snapshot.sh nnn` , where nnn is the Snapshot-Number

Within the Script there is only one variable to set:

version is the actual Snapshot-Version (with a dash at the end) and has to be updated, if the Version number changes.

This is used to build the name of the subdirectory, to which openHAB will be downloaded.

If unchanged, the script will create a subdirectory under `/srv/openhab/` with the name **versionnnn** (e.g. 1.3.0-461), then download all needed packages to another subdirectory `zips/`, unzip the packages, move addons and link `configurations/`, `etc/` and `webapps/images/` to `/srv/openhab/`**`subdir`**`/`.

At the end, all that has to be done is to restart openHAB from the new path.

All addons will reside in runtime/addons\_inactive if not moved automatically to `runtime/addons/` from the script, so it is easy to activate more addons.

A assumption is, that all user-specific Stuff resides in
```
/srv/openhab/configurations #the configs
/srv/openhab/etc #for persistence-data
/srv/openhab/images #images for the UI
```
which makes it necessary to move or copy the data first (only once).

All delivered `configurations/` are moved to `configurations_old/`, just as `etc/` is moved to `etc_old/` and `webapps/images/` is moved to `webapps/images_old/`, so no data is lost (e.g. new openhab.cfg-entrys)


## Script 2 ##

This script overwrites all files of the defined openhab-folder.
The packages you want to update can be specified in the script. (see filelist, addonlist).
You can also define file names to be excluded from the update process.

https://code.google.com/p/openhab-samples/source/browse/scripts/openhab_get_snapshot_overwrite.sh?repo=wiki

## Script 3 ##

Add-on to Script 1 to install & configure the latest master of [HABmin](https://github.com/cdjackson/HABmin) as well as the rev of openHAB selected.

[openhab\_habmin\_get\_snapshot.sh](http://code.google.com/p/openhab-samples/source/browse/scripts/openhab_habmin_get_snapshot.sh?repo=wiki)