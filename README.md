## Docker Homebridge for RaspberryPi

<p align="center">
        <img src="homebridge-lib.png" alt="PNG" height="300px" />
</p>

### Requeriments
- Raspberry Pi
- Raspbian OS
- Service docker running

### Install Docker
    sudo apt update
    sudo apt upgrade -y
    sudo apt install -y libffi-dev libssl-dev python3 python3-pip
    sudo curl -sSL https://get.docker.com | sh
    sudo usermod -aG docker pi
    sudo apt install -y docker-compose

### Add lines in /boot/config.txt (RaspberryPi 3)
    arm_freq=1200
    arm_freq_min=700
    core_freq=400
    core_freq_min=250
    sdram_freq=450
    sdram_freq_min=400

### Running
    docker-compose up -d

### Check
    docker ps -a

### Access webadmin
    http://your_ip:9590
    user: admin
    pass: admin

## Managing Homebridge
To manage Homebridge go to `http://<ip of raspberry pi>:9590` in your browser. For example, `http://192.168.1.20:9590`. From here you can install, remove and update plugins, modify the Homebridge `config.json` and restart Homebridge.

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

## Shell Access
If you require shell access to the container you can run:

    docker exec -it homebridge sh