# WeatherBird
A weather station based on pygame-ce and a mix of local and NWS data

This README will be filled out more when the code is uploaded. Here are some basics

## Interaction ##
- Touch on the far left goes to the base display if it's not already shown. If the base display is shown, a far left touch toggles between the weather picture and the radar picture.
- Touch on the far right jumps immediately to the forecasts. Additional farl right touches cycle through the forecast days.
- A touch in the middle of the screen advances one display forward.
- A USB keyboard can be used; any key advances one display. Exit the program with "q". (I used a cordless keyboard with a 2.4 Ghz USB dongle.)

## Web interface ##
THe current display, whichever it is, is available via HTTP on port 80 at the root URI. Clicking anywhere on the image advances through a series of displays.

![Example base display](outside.png)

## Execution ##
- The ```wbird.sh``` script is started by the GUI after the user is automatically logged in after boot. That script runs ```wbird.py``` in a loop, ensuring that it automatically restarts if it crashes. There is a minimum time between restarts to avoid thrasing the data sources if it crashes immediately.
- The ```wbird.py``` process will stop and the ```wbird.sh``` script will pause if the file ```STOP``` exists in the working directory (```/home/pi/wbird```).

## Implementation ##
- **Configuration** is in ```wb_config.py```. This is a late addition to the code base, so it's not as integrated into all the modules as it could/should be.
- **Display sources** are listed in the ```altUrls``` list in ```wb_config.py``` . The first entry is blank as it is the default internally-generated display. The others are actually laid over the base display. Those that are less than 480 pixels tall let the wind/warnings and current conditions show through from the base display.
- **Location** is given as latitude and longitude in ```wb_config.py``` for warnings and watches, and with the ```myStationId``` setting which names the National Weather Service weather station for current conditions.
- **Temperature and Humidity**
  - Outside temperature is *not* pulled directly from the NWS by the main processes. A background service pulls that and writes it to /var/log/weather.log
  - Inside temperature typically comes from a separate little server on a Pi Zero W that can be placed somewhere in the house more representative of the ambient home temperature than where the WeatherBird display is located. The sensor used *could* be wired into the GPIO pins of the WeatherBird's Pi itself, but with the caveat that the larger, busier Pi and always-on screen generate some heat.
  - Both inside and outside temperature are requested using the same protocol of the simplest kind; the server sends a single line of data to any TCP connection on port 8281 and closes the connection. It's implemented with ```xinetd```. The local WeatherBird system is configured to serve the outside temperature in this way as well as the separate indoors temperature server.
  - A similar service runs on port 8282 or 8283 to server a time series of data for the charts.
  - The separate indoors temperature server broadcasts a UDP datagram on port 8281 once per minute containing its IP address(es) in a JSON list. A daemon on the WeatherBird Pi listens for this and updates file ```indoors_ips.json``` which is used by the main WeatherBird process.
  - The address and port for inside and outside temperature and humidity are configured in ```wb_config.py```. The inside address/port is overwritten from ```indoors_ips.json``` if it exists.
- The farther you are from the NWS weather station, the less accurate your temperature and other current conditions will be.
- The temperature graph and the forecast pages are generated periodically from cron jobs that create PNG images referenced in ```altUrls```.

## Coding standards ##
Hah! We don't need no stinkin' standards!

The author is a veteran C programmer who started this project before learning a whole lot of Python, and it shows. At some point it was somewhat restructured to use some Object Oriented constructs, but it's still not very "pythonic".
