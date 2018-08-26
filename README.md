# Project Horus - Browser-Based HAB Chase Map

**Note: This is a work-in-progress. Features may be incomplete or non-functional!**

This folder contains code to display payload (and chase car!) position data in a web browser:

![ChaseMapper Screenshot](https://github.com/projecthorus/chasemapper/raw/master/doc/chasemapper.jpg)

The general idea is this application is run on something like a Raspberry Pi (could be the same one that's running [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx)?) and is accessed from a tablet or laptop computer via a web browser.

You also need flask, and flask-socketio, which can be installed using pip:
```
$ sudo pip install flask flask-socketio
```

You can then clone this repository with:
```
$ git clone https://github.com/projecthorus/chasemapper.git
```

## Telemetry Sources
To use the map, you need some kind of data to plot on it! The mapping backend accepts telemetry data in a few formats:
* 'Payload Summary' and 'Chase Car Position' messages, via UDP broadcast in a JSON format [described here](https://github.com/projecthorus/horus_utils/wiki/5.-UDP-Broadcast-Messages#payload-summary-payload_summary). These can be generated by:
 * [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki) - See [here](https://github.com/projecthorus/radiosonde_auto_rx/wiki/Configuration-Settings#payload-summary-output) for configuration info.
 * Various 'bridge' utilities within the [horus_utils](https://github.com/projecthorus/horus_utils/wiki) repository. For example, [FldigiBridge](https://github.com/projecthorus/horus_utils/wiki#fldigibridge-processing-of-data-from-dl-fldigi) or [HabitatBridge](https://github.com/projecthorus/horus_utils/wiki#habitat-bridge)
* 'OziMux' messages, via UDP broadcast in a simple CSV format [described here](https://github.com/projecthorus/oziplotter/wiki/3---Data-Sources#3---oziplotter-data-inputs).
 * [radiosonde_auto_rx](https://github.com/projecthorus/radiosonde_auto_rx/wiki) - See [here](https://github.com/projecthorus/radiosonde_auto_rx/wiki/Configuration-Settings#sending-payload-data-to-chasemapper--oziplotter--ozimux) for configuration info.
 * Pi-in-the-Sky's [lora_gateway](https://github.com/PiInTheSky/lora-gateway) - Using the `OziPort=8942` configuration option.


## Configuration & Startup
Many settings are defined in the [horusmapper.cfg](./horusmapper.cfg.example) configuration file.
Create a copy of the example config file using
```
$ cp horusmapper.cfg.example horusmapper.cfg
```
Edit this file with your preferred text editor. The configuration file is fairly descriptive - you will need to set:
 * At least one telemetry 'profile', which defines where payload and (optionally) car position telemetry data is sourced from.
 * A default latitude and longitude for the map to centre on.

You can then start-up the horusmapper server with:
```
$ python horusmapper.py
```

The server can be stopped with CTRL+C. Sometimes the server doesn't stop cleanly and may the process may need to be killed. (Sorry!)

You should then be able to access the webpage by visiting http://your_ip_here:5001/

## Live Predictions
We can also run live predictions of the flight path. 

To do this you need cusf_predictor_wrapper and it's dependencies installed. Refer to the [documentation on how to install this](https://github.com/darksidelemm/cusf_predictor_wrapper/).

Once compiled and installed, you will need to: 
 * Copy the 'pred' binary into this directory. If using the Windows build, this will be `pred.exe`; under Linux/OSX, just `pred`.
 * Copy the 'get_wind_data.py' script from cusf_predictor_wrapper/apps into this directory.

You will then need to modify the horusmapper.cfg Predictor section setting as necessary to reflect the predictory binary location, the appropriate model_download command, and set `[predictor] predictor_enabled = True`

You can then click 'Download Model' in the web interface's setting tab to trigger a download of the latest GFS model data. Predictions will start automatically once a valid model is available.


## Chase Car Positions
At the moment Chasemapper only supports receiving chase-car positions via Horus UDP messages. These can be generated by the [ChaseTracker](https://github.com/projecthorus/horus_utils/wiki#chasetracker--chasecar_nogui) application from horus_utils. This application can also plot your position onto the tracker.habhub.org map, so others can see when you're out balloon chasing.

Eventually support will be added to get car positions from either GPSD, or from the client's device.


## Offline Mapping via FoxtrotGPS's Tile Cache
(This is a work in progress)
Chasemapper can serve up map tiles from a specified directory to the web client. Of course, for this to be useful, we need map tiles to server! [FoxtrotGPS](https://www.foxtrotgps.org/) can help us with this, as it caches map tiles to `~/Maps/`, with one subdirectory per map layer (i.e. `~/Maps/OSM/`, `~/Maps/opencyclemap/`).

This can be enabled by setting `[offline_maps] tile_server_enabled = True`, and changing `[offline_maps] tile_server_path` to point to your tile cache directory (i.e. `/home/pi/Maps/`). Chasemapper will assume each subdirectory in this folder is a valid map layer and will add them to the map layer list at the top-right of the interface.

### Caching Maps

To grab map tiles to use with this, we're going to use FoxtrotGPS's [Cached Maps](https://www.foxtrotgps.org/doc/foxtrotgps.html#Cached-Maps) feature. 

 * Install FoxtrotGPS (Linux only unfortunately, works OK on a Pi!) either [from source](https://www.foxtrotgps.org/releases/), or via your system package manager (`sudo apt-get install foxtrotgps`). 
 * Load up FoxtrotGPS, and pan around the area you are intersted in caching. Pick the map layer you want, right-click on the map, and choose 'Map download'. You can then select how many zoom levels you want to cache, and start it downloading (this may take a while!)
 * Once you have a set of folders within your `~/Maps` cache directory, you can startup Chasemapper and start using them! Tiles will be served up as they become available.


(If anyone has managed to get ECW support working in GDAL recently, please contact me! I would like to convert some topographic maps in ECW format to tiles for use with Chasemapper.)


## Contacts
* [Mark Jessop](https://github.com/darksidelemm) - vk5qi@rfhead.net

You can often find me in the #highaltitude IRC Channel on [Freenode](https://webchat.freenode.net/).