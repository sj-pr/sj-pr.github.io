# HackerApt
**HackerApt** is an Apache web application designed to display on a RaspberryPi3-cobbled hardware setup.
 
HackerApt contains two projects.
## /html
#### A client-side web application **(html, css, js)**
- `/html/index.html` displays (best with Raspbian Chromium) current weather and WMATA train/bus info. 
- Future updates might include news/other interesting junk.

## /src
#### A Visual Studio 2017 console project to generate static data **(C#)**
Run `src/WmataStaticData/Program.cs` to create local JSON data of less-changing objects, (such as WMATA station and bus-stop info), to reduce daily API calls.

Output files generated:
```
/html/api/wmata/
    static_stationinfo.json
    static_busstopinfo.json
    static_busrouteinfo.json
```
*TODO:* set up a cron job to auto-run static data every month or so.

## Hardware (Optional):
- Monitor with HDMI input (preferably an IPS panel!)
- Short HDMI cable (1 foot is ideal) ([Amazon](https://www.amazon.com/dp/B007PESCCC))
- Raspberry Pi computer (plus wifi dongle, if using the Pi version 1 or 2) ([Amazon, with Micro USB charger](https://www.amazon.com/dp/B01C6FFNY4))
- 8GB+ SD/Micro SD card ([Amazon](https://www.amazon.com/dp/B00M55C0VU))
- Power Extension Cord ([Amazon](https://www.amazon.com/dp/B006ZON7Y6))
- Photo Frame Wire ([Amazon](https://www.amazon.com/dp/B008REERTG))

## API Developer Keys (Required):
This website uses several free third-party APIs (personal developer key required) for current data, through synchronous AJAX requests.
### <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/1e/WMATA_Metro_Logo.svg/1200px-WMATA_Metro_Logo.svg.png" height="50px" style="vertical-align:middle"> WMATA
#### <https://developer.wmata.com>
- **Default Tier** request rate limited to 10 calls/second and 50,000 calls per day.
#### Provides:
- [Bus Route and Stop Methods](https://developer.wmata.com/docs/services/54763629281d83086473f231)
- [Incidents](https://developer.wmata.com/docs/services/54763641281d83086473f232)
- [Misc Methods](https://developer.wmata.com/docs/services/5923434c08d33c0f201a600a)
- [Rail Station Information](https://developer.wmata.com/docs/services/5476364f031f590f38092507)
- [Real-Time Bus Predictions](https://developer.wmata.com/docs/services/5476365e031f590f38092508)
- [Real-Time Rail Predictions](https://developer.wmata.com/docs/services/547636a6f9182302184cda78)
- [Train Positions](https://developer.wmata.com/docs/services/5763fa6ff91823096cac1057)

### <img src="https://www.vectorlogo.zone/logos/darksky/darksky-card.png" height="80px" style="vertical-align:middle" alt="DARKSKY">
#### <https://darksky.net/dev/docs>
- First 1000 requests/day are free
- Every API request beyond that costs $0.0001
#### Provides:
- Current weather conditions
- Minute-by-minute forecasts out to one hour
- Hour-by-hour and day-by-day forecasts out to seven days
- Hour-by-hour and day-by-day observations going back decades

## Media Credits: 
### DRIPICONS-WEATHER <img src="https://image.flaticon.com/icons/svg/69/69652.svg" height="20px">
#### <http://demo.amitjakhu.com/dripicons-weather>
- *Included in `/html/svg`*

### FONTAWESOME <img src="https://upload.wikimedia.org/wikipedia/en/thumb/8/83/Font_Awesome_2017_Logo.svg/75px-Font_Awesome_2017_Logo.svg.png" height="20px">
#### <https://fontawesome.com>
- *Included in `/html/webfonts`*

# Raspbian OS Setup
## Update and Configure Raspbian Defaults:
`sudo apt-get update`
- Downloads OS package lists from the repositories for information on the newest versions of packages and their dependencies

`sudo apt-get upgrade`
- Install the newest versions of all packages currently installed on the system from the sources enumerated

`sudo raspi-config`
- Boot to Desktop logged in as pi
- Set timezone
## Raspbian packages to install:
### xscreensaver (to disable screen sleep)
`sudo apt-get install xscreensaver`
### unclutter (hides mouse after a few minutes of non-movement)
`sudo apt-get install unclutter`
### Chromium browser:
`sudo apt-get install -y rpi-chromium-mods`
### Apache web server:
`sudo apt-get install apache2 -y`
### NodeJS & Node Package Manager (NPM):
Create `/home/pi/install_nodejs.sh`:
```
echo -e "\n--- Installing NodeJS & NPM ---\n"

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash > /dev/null 2>&1
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
source ~/.bashrc > /dev/null 2>&1

nvm install node
nvm alias default node

nvm ls
```
Then,
```
sudo chmod 777 install_nodejs.sh
sudo install_nodejs.sh
```
### CORS-ANYWHERE NodeJS Proxy
#### <https://github.com/Rob--W/cors-anywhere> 

HackerApt uses the CORS-ANYWHERE NodeJS proxy server to allow cross-origin API requests on Raspbian Chromium.

- Install NodeJS/NPM packages above first, then install CORS-ANYWHERE:
```
cd /home/pi
npm install cors-anywhere
```
To RUN CORS-ANYWHERE once:
```
node /home/pi/node_modules/cors-anywhere/server.js
```

To start CORS-ANYWHERE server at boot, edit this file:
```
sudo nano /etc/rc.local
```
Add these lines:
```
# Start CORS-ANYWHERE Server
node /home/pi/node_modules/cors-anywhere/server.js < /dev/null &>> /home/pi/node_modules/cors-anywhere/log &
```

## API Settings
Create an `api_keys.txt` file in `/html/settings`.

### Sample `/html/settings/api_keys.txt` file:
```
WMATA KEY: {your WMATA developer key}
DARKSKY KEY: {your DARKSKY developer key}
USEPROXY: yes
```
Set **USEPROXY** to yes if you see "Access-Control-Allow-Origin" errors in your application. This will enable CORS-ANYWHERE proxying to allow cross-origin client-side requests. Ensure that CORS-ANYWHERE server is running on your RaspberryPi.


### `/html/settings/wmata_display_bus_stops.txt`
*Lines preceded with `#` are considered comments and ignored.*
```
#
# Add bus stops by number here, one per line, in order of desired display.
# -------------------------------------------------------------------
# -- 16th St. NW + Irving St. NW (Southbound)
1002872
# -- 14th St. NW + Irving St. NW (Southbound)
1003087
# -- 16th St. NW + M St. NW (Northbound)
#1001325
# -- 14th St. NW + N St. NW (Northbound)
#1001362
```
### `/html/settings/wmata_display_train_stations.txt`
*Lines preceded with `#` are considered comments and ignored.*
```
#
# Add train stations here, one per line, in order of desired display.
# 	- Can use partial name, for example, "Gallery Pl" instead of 
#	  official, "Gallery Pl-Chinatown"
# -------------------------------------------------------------------
Columbia Heights
Farragut North
#L'Enfant
#Farragut West
```

## Raspbian OS Display Configuration
`sudo nano /boot/config.txt`
```
# Display orientation. Landscape = 0, Portrait = 1
display_rotate=1

# Use 24 bit colors
framebuffer_depth=24

# You'll want to check and make sure that disable_overscan=1 is commented out.
# If black pixels border your desktop image, modify these values until the picture fits your monitor perfectly.
overscan_left=-32
overscan_right=-32
overscan_top=-32
overscan_bottom=-32
```
### Edit autostart file for boot options
`sudo nano ~/.config/lxsession/LXDE-pi/autostart`
```
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi

# Disable Screensaver
@xset s off      # don't activate screensaver
@xset -dpms      # disable DPMS (Energy Star) features.
@xset s noblank  # don't blank the video device
#@xscreensaver -no-splash

# Tool to move the mouse pointer to the application menu launch button
@point-rpi

# Hide mouse after a few minutes of non-movement
@unclutter -display :0 -noevents -grab &
#@unclutter -idle 0

# Start HackerApt
@chromium-browser --noerrdialogs --disable-session-crashed-bubble --user-data-dir --kiosk --incognito --start-maximized http://hackerapt.com/index.html
```

## Disable Screensaver / Screen blanking
`sudo nano /etc/lightdm/lightdm.conf`
Add below `[Seat:*]` line:
```
xserver-command=X -s 0 -dpms
```

## Desktop Shortcuts
### Fullscreen Webpage
`sudo nano /home/pi/Desktop/hackerapt_fullscreen.desktop`
```
[Desktop Entry]
Type=Application
Name=HackerApt (Fullscreen)
Comment=Weather, trains, buses, and news
Icon=/home/pi/Pictures/[some picture].png
Exec=chromium-browser --noerrdialogs --disable-session-crashed-bubble --user-data-dir --kiosk --incognito --start-maximized http://hackerapt.com/index.html
```

### Maximized, but not Fullscreen Webpage
`sudo nano /home/pi/Desktop/hackerapt.desktop`
```
[Desktop Entry]
Type=Application
Name=HackerApt
Comment=Weather, trains, buses, and news
Icon=/home/pi/Pictures/[some picture].png
Exec=chromium-browser --noerrdialogs --disable-session-crashed-bubble --user-data-dir http://hackerapt.com/index.html
```

## Raspbian System Startup
### Display **HackerApt** web application on boot.
`sudo nano /etc/rc.local`
```
# Run HackerApt at Startup
# FULLSCREEN:
chromium-browser --noerrdialogs --disable-session-crashed-bubble --user-data-dir --kiosk --start-maximized http://hackerapt.com/index.html
# Or MAX SCREEN:
# chromium-browser --noerrdialogs --disable-session-crashed-bubble --user-data-dir http://hackerapt.com/index.html
```

### Turn HDMI on/off during unused hours to save power.
#### `/home/pi/rpi-hdmi.sh` ([Source](https://gist.github.com/AGWA/9874925))
```
#!/bin/sh

# Enable and disable HDMI output on the Raspberry Pi
# Goes in /home/pi/rpi-hdmi.sh

is_off ()
{
	tvservice -s | grep "TV is off" >/dev/null
}

case $1 in
	off)
		tvservice -o
	;;
	on)
		if is_off
		then
			tvservice -p
			curr_vt=`fgconsole`
			if [ "$curr_vt" = "1" ]
			then
				chvt 2
				chvt 1
			else
				chvt 1
				chvt "$curr_vt"
			fi
		fi
	;;
	status)
		if is_off
		then
			echo off
		else
			echo on
		fi
	;;
	*)
		echo "Usage: $0 on|off|status" >&2
		exit 2
	;;
esac

exit 0
```

### CRON tab Editor to run `rpi-hdmi.sh` on a set schedule
`sudo crontab -e`
```
# RULES:
# ------
# Turn HDMI On (6:00am) M-F
0 6 * * 1-5 /home/pi/rpi-hdmi.sh on 
# Turn HDMI Off (9:00am) M-F
0 9 * * 1-5 /home/pi/rpi-hdmi.sh off
# Turn HDMI On (5:45pm) M-F
45 17 * * 1-5 /home/pi/rpi-hdmi.sh on 
# Turn HDMI On (8:00am) SS
0 8 * * 6-7 /home/pi/rpi-hdmi.sh on
# Turn HDMI Off (11:59pm) EVERY DAY
59 23 * * * /home/pi/rpi-hdmi.sh off 
```

## License

**Copyright (C) 2018 [Steph Stubler](https://www.linkedin.com/in/stephstubler)**

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.