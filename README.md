<p align="center">
        <img src="res/homebridge-lib.png" alt="PNG" height="300px" />
</p>


## How to setup Homebridge & Docker on a Raspberry Pi

This guide will show you how to run the [oznu/homebridge](https://hub.docker.com/r/oznu/homebridge/) docker image on a Raspberry Pi.

- [Requirements](#requirements)
- [Initial Raspberry Pi Setup](#initial-raspberry-pi-setup)
- [Quick Install](#quick-install)
- [Manual Install](#manual-install)
  - [Step 1: Install Docker](#1-install-docker)
  - [Step 2: Install Docker Compose](#2-install-docker-compose)
  - [Step 3: Create Docker Compose Manifest](#3-create-docker-compose-manifest)
  - [Step 4: Start Homebridge](#4-start-homebridge)
- [Managing Homebridge](#managing-homebridge)
- [Connect iOS / HomeKit](#connect-ios--homekit)
- [Updating Homebridge / Node.js](#updating-homebridge--nodejs)
- [Shell Access](#shell-access)

## Requirements

> :bulb: Homebridge also provides a [Raspberry Pi Image](https://github.com/homebridge/homebridge-raspbian-image#readme) built on Raspbian Lite. If you're just starting out, this is the best option.

> If you are going to use plugins that require access to the Raspberry Pi's GPIO or Bluetooth radio, or require hardware video decoding, you should be aware using Docker to run Homebridge may add an additional layer of complexity to your setup. If this is the case may wish to consider [Setting up Homebridge with Systemd](https://github.com/oznu/homebridge-config-ui-x/wiki/Homebridge-&-Systemd-(Raspbian,-Ubuntu,-Debian)) instead. 

* Raspberry Pi 1
* Raspberry Pi Zero W
* Raspberry Pi 2
* Raspberry Pi 3
* Raspberry Pi 4

## Initial Raspberry Pi Setup

### Download and Install Raspbian

Get the latest copy of **Raspbian Buster Lite** [[raspbian-lite-latest.zip](https://downloads.raspberrypi.org/raspbian_lite_latest)] from the official [Raspberry Pi website](https://www.raspberrypi.org/downloads/raspbian/) and burn this to your SD card using [Etcher](https://www.balena.io/etcher/).

* **Raspbian Buster Lite** is prefered as there is no need to run a GUI desktop.
* [How to install Raspberry Pi Operating System Images](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

### Configure Raspbian for Headless Boot

By default SSH access and WiFi are disabled in Raspbian. You can enable both these services before we boot from the SD card for the first time - this will avoid the need to ever connect a screen to the Pi.

**The following changes to the freshly imaged SD card should be made on your computer before you plug the card into the Raspberry Pi for the first time.**

#### Enable SSH

To enable remote SSH access on first boot create an empty file called `ssh` in the root of the SD card.

#### Enable WiFi (Optional)

If you have a Raspberry Pi with built in WiFi (Pi 3 or Zero W), you can configure WiFi on first boot. To do this create a file named `wpa_supplicant.conf` in the root of the SD card that contains the following (replace the [YOUR_COUNTRY_CODE](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), YOUR_SSID and YOUR_PASSWORD values):

```
country=YOUR_COUNTRY_CODE
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="YOUR_SSID"
    scan_ssid=1
    psk="YOUR_PASSWORD"
    key_mgmt=WPA-PSK
}
```

### Login

Power on your Raspberry Pi and connect to the console using SSH. 

If you're running macOS or have Bonjour for Windows (part of iTunes) installed, you should be able to connect using `raspberrypi.local` as the hostname. If not then you'll need to find out what IP address the Raspberry Pi was assigned and connect using that instead.

```shell
ssh pi@raspberrypi.local
```

The default username is ```pi``` and password ```raspberry```. You should change the default password now using the `passwd` command.

# Quick Install

The quick install script will install Docker, Docker Compose, setup the `docker-compose.yml` file and start the docker container for you.

[Script Source](https://github.com/oznu/docker-homebridge/blob/master/raspbian-installer.sh)

To run the quick install script:

```shell
curl https://raw.githubusercontent.com/oznu/docker-homebridge/master/raspbian-installer.sh?v=2019-12-11 -o get-homebridge.sh
chmod u+x get-homebridge.sh
./get-homebridge.sh
```

Once installed see [Managing Homebridge](#managing-homebridge).

If you don't want to use the quick install script, you can manually run the steps below.

# Manual Install

## 1. Install Docker

Install Docker from the official repository by running these commands:

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
chmod u+x get-docker.sh
sudo ./get-docker.sh
```

Add the ```pi``` user to the docker group:

```shell
sudo usermod -aG docker pi && logout
```
_You will need to logout of the Raspberry Pi and login again in order for your user to pick up the docker group membership._

## 2. Install Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) allows you to easily create a manifest for your Docker containers.

```
sudo apt -y install docker-compose
```

## 3. Create Docker Compose Manifest

Create a new directory to store your homebridge docker-compose manifest and config data in. In this example we will install Homebridge in the ```pi``` user's home directory.

Create a new directory and change into it:

```
mkdir /home/pi/homebridge
cd /home/pi/homebridge
```

Create a new file called ```docker-compose.yml``` using ```nano```:

```
nano docker-compose.yml
```

The contents of this file should be:

```yml
ersion: '2'
services:
  homebridge:
    image: oznu/homebridge:raspberry-pi
    restart: always
    network_mode: host
    container_name: homebridge
    volumes:
      - ./homebridge:/homebridge
      - ./config:/homebridge/config
    environment:
      - TZ=Europe/Madrid
      - PGID=1000
      - PUID=1000
      - HOMEBRIDGE_CONFIG_UI=1
      - HOMEBRIDGE_CONFIG_UI_PORT=9591
```
* The ```restart: always``` line instructs docker to setup the container so that it that will automatically start again if the Raspberry Pi is rebooted, or if the container unexpectedly quits or crashes.
* The ```network_mode: host``` line instructs docker to share the Raspberry Pi's network with the container, allowing your iOS device to find the Homebridge accessory.
* The ```./config:/homebridge``` instructs docker to share the local folder ```config``` with the container. This will allow you to recreate or update the docker container without losing any Homekit settings or Homebridge plugins.
* For an explanation of the `PGID` and `PUID` environment variables please see [User & Group Identifiers](https://github.com/oznu/docker-homebridge#user--group-identifiers).
* The `HOMEBRIDGE_CONFIG_UI` and `HOMEBRIDGE_CONFIG_UI_PORT` enable the [homebridge-config-ui-x](https://github.com/oznu/homebridge-config-ui-x) plugin. You can remove these two options if you don't want to use the UI.

Save and close the file by pressing ```CTRL+X```.

## 4. Start Homebridge

Start the Homebridge Docker container by running:

```
docker-compose up -d
```

* It might take some time to download the initial image which is about 125 MB compressed.
* Docker will now download the latest [oznu/homebridge](https://hub.docker.com/r/oznu/homebridge/) docker image. 
* The ```-d``` flag tells ```docker-compose``` to run the container as a background process.

You'll probably want to view the Homebridge logs to check everything is working and to get the iOS pairing code:

```
docker-compose logs -f
```

![View oznu/homebridge logs](https://github.com/oznu/docker-homebridge/wiki/images/oznu-homebridge-docker-1100-raspberry-pi-logs.png)

Your Homebridge ```config.json```, plugins and all HomeKit data will be stored in the newly created ```config``` directory.

# Managing Homebridge

To manage Homebridge go to `http://<ip of raspberry pi>:8080` in your browser. For example, `http://192.168.1.20:8080`. From here you can install, remove and update plugins, modify the Homebridge `config.json` and restart Homebridge.

The default username is **admin** with password **admin**. Remember you will need to restart Homebridge to apply any changes you make to the `config.json`.

![Homebridge UI](https://github.com/oznu/homebridge-config-ui-x/blob/master/screenshots/homebridge-config-ui-x-plugins.png)

You can restart the container by running:

```
docker-compose restart homebridge
```

## Connect iOS / HomeKit

You should now be able to see the ```Homebridge``` as a new HomeKit accessory in the Apple iOS Home App. You can pair the device using the QR code displayed in the logs (see above) or in browser using the Homebridge UI.

1. Open the Home <img src="https://user-images.githubusercontent.com/3979615/78010622-4ea1d380-738e-11ea-8a17-e6a465eeec35.png" height="16.42px"> app on your device.
2. Tap the Home tab, then tap <img src="https://user-images.githubusercontent.com/3979615/78010869-9aed1380-738e-11ea-9644-9f46b3633026.png" height="16.42px">.
3. Tap *Add Accessory*, then scan the QR code shown in the Homebridge UI or your Homebridge logs.

If the QR code is not displaying correctly and you're using using iOS 11 you will need to follow these steps:

1. Open the Home <img src="https://user-images.githubusercontent.com/3979615/78010622-4ea1d380-738e-11ea-8a17-e6a465eeec35.png" height="16.42px"> app on your device.
2. Tap the Home tab, then tap <img src="https://user-images.githubusercontent.com/3979615/78010869-9aed1380-738e-11ea-9644-9f46b3633026.png" height="16.42px">.
3. Tap *Add Accessory*, and select *I Don't Have a Code or Cannot Scan*.
4. Enter the Homebridge PIN, this can be found under the QR code in Homebridge UI or your Homebridge logs.

## Updating Homebridge / Node.js

To update Homebridge or Node.js to the latest version you just need to pull the latest version of [oznu/homebridge](https://hub.docker.com/r/oznu/homebridge/).

Go to the directory containing your` docker-compose.yml` file:

```
cd /home/pi/homebridge
```

Download the latest [oznu/homebridge](https://hub.docker.com/r/oznu/homebridge/) image:

```
docker-compose pull homebridge
```

If a newer version of the image was downloaded, upgrade the container using the new image by running the ```up``` command again:

```
docker-compose up -d
```

## Shell Access

If you require shell access to the container you can run:

```
docker-compose exec homebridge sh
```
