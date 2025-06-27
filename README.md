# Prototype
The Project originates in a Schoolproject with limited time. As I won't be able to finish everything I want in Time, a Prototype is needed.

## Prototype Hardware
For the Prototype a **RaspberryPi 5+** will be used alongside the Pre-Build [MagicMirror²](https://magicmirror.builders/) Software. Also a Temperature Sensor will be used to Display the current Room Temperature.


Hardware wise the following things are needed:
- A Monitor
- A Raspberry Pi 5
- A Micro SD card
- Temperatur Sensor

### MicroSD Setup
To Set up the MicroSD card for the RPi3 another computer is needed to flash the SD card. The Computer can be a Mac, Windows or Linux PC. You'll need the  [RPi Imager](https://www.raspberrypi.com/software/). It'll Download and flash the newest Raspbian OS on the SD card.

<img src="../Documentation/0002-Prototype/01-RPi-logo.png" alt="Raspberry Pi Imager logo" height="100">

In the [RPi Imager](https://www.raspberrypi.com/software/) Software make every Selection needed:

<img src="../Documentation/0002-Prototype/02-RPimager-setup.png" alt="Raspberry Pi Imager Setup">

1. Select your Pi (example: Raspberry Pi 5)
2. Select Operating System: (Debian Desktop (64-Bit))
3. Select your SD-kart (Mind that the Drive will be formated)
4. Start Flashing

Rpi Imager will now **Download** the OS then **Flash** it to the SD-card and then Verifies the Data.

<img src="../Documentation/0002-Prototype/03-Write-success.png" alt="Successfull Writing">

### Connecting the Raspberry
- Insert the SD-Card into the slot of the RPi
- Connect a compatible HDMI kable to a monitor
- Connect Mouse and keyboard to the RPi
- Connect to Power

The Pi should startup now

### Setting the Pi up

After the Pi Startsup, you'll be greated by a Welcome Message. Simply click *next*
<img src="../Documentation/0002-Prototype/02-Initial-Setup/01-Welcome-msg.jpeg" alt="RPiOS Welcome MEssage">

Next Select your **Country**, **Language** and **Timezone**. I also Ticked *"Use Englisch Language"*. Then click *Next*

<img src="../Documentation/0002-Prototype/02-Initial-Setup/02-Set-Country.jpeg" alt="Set-Country Window">

The Next Step is to create your Useraccount.
Select a username (all lowercase and without custom characters) and a Password.
For example: *magicmirr* and *magicmirr*.

Then connect to a **Wi-Fi Network**. After That select your **Favourite Browser** (Chromium of course) and opt in to **install the latest software updates**.

The Last Step is to Restart the Pi.

### Activate SSH and VNC and SPI as well as I2C

[SSH](https://de.wikipedia.org/wiki/Secure_Shell) and [VNC](https://de.wikipedia.org/wiki/Virtual_Network_Computing) is used to connect to pi from other Computers in the network to maintain it without a **Keyboard** or **Mouse** connected. SPI and I2C will be needed for the Temperature Sensor

To activate those Protocolls go to the RPI-Menu in the top-left go to **Preferences** and select **Raspberry Pi Configuration**.
<img src="../Documentation/0002-Prototype/03-SSH-VNC-Setup/01-RPi-menu.jpeg" alt="Raspberry Pi Menu with Rasppi Configuration selected">

In the *Raspberry Pi Configuration* Window select the **Interfaces** Tab and toggle on [*SSH*](https://de.wikipedia.org/wiki/Secure_Shell) and [*VNC*](https://de.wikipedia.org/wiki/Virtual_Network_Computing). Of course also activate [*SPI*](https://de.wikipedia.org/wiki/Serial_Peripheral_Interface) and [*I2C*](https://de.wikipedia.org/wiki/I%C2%B2C).
<img src="../Documentation/0002-Prototype/03-SSH-VNC-Setup/02-RPi-Config-Window.jpeg" alt="02 RPi Config Window">

At Last select *OK*. The Configuration will be Updated.
Get the IP Address of the Raspberry and you can connect to it using **SSH** and **VNC**.

### Connect the Sensor to the RPI
Firstly get the Pinout of the Raspberry.
You can do so by using the `pinout`command in the Terminal.
<img src="../Documentation/0002-Prototype/04-Sensor-Connection/pinout.png" alt="pinout">

**Shutdown** the Pi and **Disconnect Power!!!**

Here is the **Sensor Pinout**:

<img src="../Documentation/0002-Prototype/04-Sensor-Connection/sensor-pinout.jpeg" art="DHT22 Pinout">

Connection:
|    RPI-Pin   | Sensor-Pin  |
|--------------|-------------|
| 3V3 (Pin1)   | VCC (Pin1)  |
| GPIO4 (Pin7) | Data (Pin2) |
| GND (Pin6)   | GND (Pin4)  |

<img src="../Documentation/0002-Prototype/04-Sensor-Connection/Realer-Aufbau.jpeg">

<img src="../Documentation/0002-Prototype/04-Sensor-Connection/Fritzing aufbau.png">

**Not guaranteed to working (Information Provided without guarantee) not liable for Fires or else**

Start the Pi Back up

## Prototype Software
### Prerequired Software
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git build-essential python3-pi
```

### Install MagicMirror2
```bash
bash -c  "$(curl -sL https://raw.githubusercontent.com/sdetweil/MagicMirror_scripts/master/raspberry.sh)"
```

After Installation MagicMirror starts automatically.
To stop it type `pm2 stop default` in the Terminal.

To manually start MagicMirror use `pm2 start default`
Magicmirr should start automatically

### Set Up DHT22 Sensor Reading
First Create a new virutal Python environment
```bash
sudo apt install python3-venv
python3 -m venv ~/dht-env
```

Activate the environment
```bash
source ~/dht-env/bin/activate
```

Install needed packages inside the venv
```bash
(dht-env) magicmirr@raspberrypi:~ $ pip install adafruit-circuitpython-dht
```

Then test the sensor with a small testfile
```bash
nano ~/Documents/dht22test.py
````

I used the following python script:
```python
#!/home/magicmirr/dht-env/bin/python
import time
import board
import adafruit_dht

# Use GPIO4 (physical pin 7)
dhtDevice = adafruit_dht.DHT22(board.D4)

while True:
    try:
        temperature_c = dhtDevice.temperature
        humidity = dhtDevice.humidity
        print(f"Temp: {temperature_c:.1f}°C  Humidity: {humidity:.1f}%")
    except Exception as e:
        print("Sensor error:", e)
    time.sleep(2)
```
and run it `python3 ~/Documents/dht22test.py`

### Configure MagicMirror to use DHT-sensor-module
```bash
nano ~/dht22-to-file.py
```

Use Python Script to save read value to file

```python
#!/home/magicmirr/dht-env/bin/python
import time
import board
import adafruit_dht

dhtDevice = adafruit_dht.DHT22(board.D4)

try:
    temperature_c = dhtDevice.temperature
    humidity = dhtDevice.humidity
    with open("/home/magicmirr/dht22.txt", "w") as f:
        f.write(f"Temp: {temperature_c:.1f}°C\nHumidity: {humidity:.1f}%")
except Exception as e:
    with open("/home/magicmirr/dht22.txt", "w") as f:
        f.write("Sensor error")
```

Make it executable
```bash
chmod +x ~/dht22-to-file.py
```

#### Set up a cronjob to update every minute
```bash
crontab -e
```
Enter:
```
* * * * * /home/magicmirr/dht-env/bin/python3 /home/magicmirr/dht22-to-file.py
```

### Custom Module for Temp
As there is no available module to read from a file and Display it I'll need to write it myself

#### Create module folder
```bash
mkdir -p ~/MagicMirror/modules/MMM-DHT
```

#### Create MMM-DHT.js
```bash
nano ~/MagicMirror/modules/MMM-DHT/MMM-DHT.js
```
```javascript
Module.register("MMM-DHT", {
  defaults: { updateInterval: 60*1000, file: "/home/magicmirr/dht22.txt" },

  start() {
    this.getReading();
    setInterval(() => this.getReading(), this.config.updateInterval);
  },

  getReading() {
    this.sendSocketNotification("READ_FILE", this.config.file);
  },

  socketNotificationReceived(notification, payload) {
    if (notification === "READ_RESULT") {
      this.content = payload;
      this.updateDom();
    }
  },

  getDom() {
    const el = document.createElement("pre");
    el.textContent = this.content || "Loading…";
    return el;
  }
});
```

#### Create Node Helper
```bash
nano ~/MagicMirror/modules/MMM-DHT/node_helper.js
```
```javascript
const NodeHelper = require("node_helper");
const fs = require("fs");

module.exports = NodeHelper.create({
  socketNotificationReceived(notification, payload) {
    if (notification === "READ_FILE") {
      fs.readFile(payload, "utf8", (err, data) => {
        this.sendSocketNotification("READ_RESULT", err ? "Error" : data);
      });
    }
  }
});
```

#### Add to config.js
```bash
nano ~/MagicMirror/config/config.js
````
Add
```javascript
{
  module: "MMM-DHT",
  position: "top_right",
  config: {}
},
```

### Additional config.js changes
#### Change US-Calendar to a GER-Calendar (C3D2 Calendar)
```javascript
{
	module: "calendar",
	header: "C3D2 Events",
	position: "top_left",
	config: {
		calendars: [
			{
				fetchInterval: 7 * 24 * 60 * 60 * 1000,
				symbol: "calendar-check",
				url: "https://c3d2.de/ical.ics"
			}
		]
	}
},
```
#### Complimets removed
Thought it is Cringe

#### Set Weather Location
Location for "Volkswagen Gläserne Manufaktur" is 51.044789152150976, 13.755615297791643
```javascript
{
	module: "weather",
	position: "top_right",
	config: {
		weatherProvider: "openmeteo",
		type: "current",
		lat: 51.044789,
		lon: 13.755615
	}
},
{
	module: "weather",
	position: "top_right",
	header: "Weather Forecast",
	config: {
		weatherProvider: "openmeteo",
		type: "forecast",
		lat: 51.044789,
		lon: 13.755615
	}
},
```

#### Reliable News
Changed Newsfeed from The New York Time to Postillon
```Javascript
{
	module: "newsfeed",
	position: "bottom_bar",
	config: {
		feeds: [
			{
				title: "Der Postillon",
				url: "https://follow.it/der-postillon-abo/rss"
			}
		],
		showSourceTitle: true,
		showPublishDate: true,
		broadcastNewsFeeds: true,
		broadcastNewsUpdates: true
	}
},
```

## Config Files
The config file can be found in the [config-files](./config-files/) Folder.

The main Configuration file is the [config.js](./config-files/config.js) also the [modules](./config-files/MagicMirror/modules/) is important to see.

## Working Configuration Video
[<img src="./Showcase.mp4" width="600" height="300"/>](./Showcase.mp4>)

You can see the Temperature Update at **1:25**
