# BTT-Manta-CB1-Development-Documentation

## OS image for BTT:
Github Releases : [link](https://github.com/bigtreetech/CB1/releases/tag/V2.3.2)

## Image Upload & Boot Process:
Follow information [here](https://3dpandme.com/2022/08/14/btt-manta-m4p-cb1-installation-guide-for-voron-0-1/) untill the step you are able to log into the board.

``` NOTE: Add the USB power jumper to the board before powering it on using the USD C cable. ```


Referance links:

https://github.com/bigtreetech/Manta-M4P/blob/master/BIGTREETECH_MANTA_M4P_User_Manual.pdf

https://github.com/bigtreetech/TFT35-SPI

https://github.com/bigtreetech/CB1

Possibly refer this to install X11 and touch screen interface: https://github.com/jordanruthe/KlipperScreen

Steps to install the OS image:

1. If BIGTREETECH CB1 core board is used, You can only download and install the system image provided by BIGTREETECH:

https://github.com/bigtreetech/CB1/releases

In the above link, multiple assets of image files will be found. Make sure the image file having an extension of `img.xz` and a `CB1_Debian_minimal_kernel` with the latest verison is used.

2. Install the official Raspberry Pi Imager: https://www.raspberrypi.com/software/. The system image of CB1 can also be written with this software.

3. Plug the Micro SD card into the computer via a card reader.

4. Select Operating system:

![Screenshot 2023-04-28 121405](https://user-images.githubusercontent.com/80109965/235837774-d66dd72f-0eb9-4351-8005-c8e329e17f57.png)

5. Select "Use Custom", then select a custom.img from your computer.

![Screenshot 2023-04-28 121456](https://user-images.githubusercontent.com/80109965/235837876-8d56757c-1396-4885-b10b-2059465e4b75.png)

6. Select the Micro SD card and click "WRITE" (Writing the image will format the Micro SD card. Be careful not to select the wrong storage device, otherwise, the 
data will be formatted).

![Screenshot 2023-04-28 121718](https://user-images.githubusercontent.com/80109965/235837995-3f3536d0-f8fd-4e07-8446-7a66872cfb26.png)

7. Wait for the writing to finish.

![Screenshot 2023-04-28 122152](https://user-images.githubusercontent.com/80109965/235838019-1081000f-b010-4878-b845-10791be5a364.png)

### WiFi Setup:

Note: This step can be skipped if you are using a network cable connection.

CB1 cannot directly use the Raspberry Pi Imager to set the WiFi name and password like CM4. After the OS image writing is completed, the MicroSD card will have a FAT32 partition recognized by the computer, find "system.cfg"

Open it with Notepad, replace WIFI-SSID with your WiFi name, and PASSWORD with 
your password.

![image](https://user-images.githubusercontent.com/80109965/235838416-f8feccfb-3478-4a32-a9d0-6dded2bc073e.png)


## Process to Install OctoPrint on BTT Manta M4P CB1

Reference for the complete installation process is present in https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspberry-pi-os-debian/2337

### Steps to Install Octoprint using any SSH server (Consider PuTTY)
Find the IP address using any IP scanner software (consider AngryIP Scanner), where the hostname would appear as `BTT-CB1.local`. Note it's IP address to access the board. 

Now using `PuTTY`, enter the  `HostName` as the IP address that was noted, and username and password is to be entered as `biqu`, unlike how 'pi' was used in the case of 'RaspberryPi'. 

After logging in, the terminal will show the username as `biqu@BTT-CB1`, unlike in RasPi, where it showed 'pi@raspberry'.

Make sure the BTT CB1 has `python3 3.7, 3.8, 3.9 or 3.10` running and not just python, and pip should be present. in order to check the current version, check using:

`python3 --version`

Now, we continue the process of installing the Octoprint using a few commands that are to be entered in terminal of the SSH server. Installing OctoPrint should be done within a virtual environment, rather than an OS wide install, to help prevent dependency conflicts. To setup Python, dependencies and the virtual environment, run:

``` 
cd ~
sudo apt update
sudo apt install python3 python3-pip python3-dev python3-setuptools python3-venv git libyaml-dev build-essential libffi-dev libssl-dev
mkdir OctoPrint && cd OctoPrint
python3 -m venv venv
source venv/bin/activate 
```


OctoPrint and it's Python dependencies can then be installed using pip:


```
pip install pip --upgrade
pip install octoprint
```

You may need to add the biqu user to the dialout group and tty so that the user can access the serial ports, before starting OctoPrint:

```
sudo usermod -a -G tty biqu
sudo usermod -a -G dialout biqu
```

### Starting the server for the first time

You should then be able to start the OctoPrint server using the octoprint serve command:

```
biqu@BTT-CB1:~ $ ~/OctoPrint/venv/bin/octoprint serve
```

Try it out! Access the server by heading to `http://<pi's IP>:5000` and you should be greeted with the OctoPrint UI.

### Automatic Startup

Download the init script files from OctoPrint's repository, move them to their respective folders and make the init script executable:

```
wget https://github.com/FracktalWorks/OctoPrint/raw/master/scripts/octoprint.service && sudo mv octoprint.service /etc/systemd/system/octoprint.service
```

Note: The above github link is the forked version of the original Octoprint's octoprint.service raw script where the username 'pi' has been replaced with `biqu`. For reference of the original file, refer https://github.com/OctoPrint/OctoPrint/raw/master/scripts/octoprint.service

Adjust the paths to your `octoprint` binary in `/etc/systemd/system/octoprint.service`. If you set it up in a virtualenv as described above make sure your `/etc/systemd/system/octoprint.service` looks like this:

```
ExecStart=/home/pi/OctoPrint/venv/bin/octoprint
```
Then add the script to autostart using `sudo systemctl enable octoprint.service`.

This will also allow you to start/stop/restart the OctoPrint daemon via

```
sudo service octoprint {start|stop|restart}
```

### Make everything accessible on port 80

Setup on Raspbian is as follows:

```
biqu@BTT-CB1 ~ $ sudo apt install haproxy
```
In `/etc/haproxy/haproxy.cfg` copy down the configuration as per the versions:


for Haproxy 2.x (Debian 11, Bullseye etc.):

```
global
        maxconn 4096
        user haproxy
        group haproxy
        daemon
        log 127.0.0.1 local0 debug

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        retries 3
        option redispatch
        option http-server-close
        option forwardfor
        maxconn 2000
        timeout connect 5s
        timeout client  15min
        timeout server  15min

frontend public
        bind :::80 v4v6
        use_backend webcam if { path_beg /webcam/ }
        default_backend octoprint

backend octoprint
        option forwardfor
        server octoprint1 127.0.0.1:5000

backend webcam
        http-request replace-path /webcam/(.*)   /\1
        server webcam1  127.0.0.1:8080
```

For more reference on the Haproxy 2.x and 1.x configuration, refer here: https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspberry-pi-os-debian/2337#make-everything-accessible-on-port-80-4

https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspberry-pi-os-debian/2337#haproxy-2x-debian-11-bullseye-etc-5

https://community.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspberry-pi-os-debian/2337#haproxy-1x-debian-10-buster-etc-6

This will make OctoPrint accessible under `http://<your Raspi's IP>/` and make mjpg-streamer accessible under `http://<your Raspi's IP>/webcam/`. You'll also need to modify `/etc/default/haproxy` and enable HAProxy by setting `ENABLED` to `1`. 

This can be done by issuing the following command:

```
sudo nano /etc/default/haproxy
```

It redirects to the internal file. In case there is no `ENABLED=` line seen, add it up manually in the last line and enable it by typing in `ENABLED=1`. 

Then `CTRL + X` to exit. It'll ask to save the changes. Then type in `Y`. Then press `Enter` button to continue into the terminal.

After this, to enable the Octorpint on port 80, type in:

```
sudo service haproxy start
```

This will enable to run OctoprintUI using `only the IP Address` and not the other port such as `:5000`

In case of issues persisting, consider restarting the module and retry

### Configuring OctoPrint settings

After creating the user profile for the first time, certain configuration is required for the features in the octoprint. 

1. In the `Octoprint settings (wrench symbol)`, select the `Serial Connection` section, and under the `General` tab, mention the serial port path `/home/biqu/printer_data/comms/klippy.serial` in the `Additional serial ports` box under the `Connection` tab. 

Note: In usual cases of connecting Klipper when connecting with a regular RaspberryPi, the serial port is usually '/tmp/printer' that the klipper documentation suggests, however in the case of BTT CB1, it generates different folders, hence the serial port is to be defined accordingly.

2. Set the Serial Port to either `AUTO` or the `/home/biqu/printer_data/comms/klippy.serial` which has been successfully detected by the port. Set the Baudrate to `250000`. 

3. Select the `Auto-connect to printer on server start` option.

4. Under `Blacklisted serial ports`, enter `/dev/ttyS*` in order to remove those existing serial ports under the serial ports option.

5. In the main settings tab, select `Webcam & Timelapse` tab, under the `Webcam` section, check on the `Enable webcam support`. Below it, in the `Stream URL` box, enter in `/webcam/?action=stream` in order to activate the camera stream, if a Pi-based camera is connected.

## Steps to Install Klipper in BTT-M4P-CB1 using KIAUH

1. Open the SSH server and login into the CB1 and head onto the terminal.
2. In the terminal, enter in:
```
git clone https://github.com/th33xitus/kiauh.git
``` 
This installs `kiauh`, which is the script that assists in installing different firmwares and 3D printer web interfaces.

3. Next in the terminal, enter:
```
./kiauh/kiauh.sh
```

This will redirect to the KIAUH interface. Once you're in the KIAUH interface, the left-hand side indicates the main navigation menu. And on the right side, you'll find the status of all the software available for the system. Currently, we have the Octoprint interface as per the previous section. We'll need to install Klipper on CB1.

![image](https://user-images.githubusercontent.com/80109965/234785061-6ad4087f-eb24-42ee-84c8-dc7a66b93924.png)

4. Input numerical 1 and press enter. It'll take you to the Installation menu.

5. Klipper is present in the Firmware and API section. Input and enter 1. It'll begin the Klipper installation process.

![image](https://user-images.githubusercontent.com/80109965/234785678-9eb4f885-45c1-499b-a3d9-b47e70e20e81.png)

6. Select the Python version as Python 3.x by inputting 1. It's a stable version, and will give you predictable results.

![image](https://user-images.githubusercontent.com/80109965/234786026-400285ca-e3a8-4ec9-832c-d1fe84d2fb3c.png)

7. Select the number of Klipper instances to set up. By default, the value is 1. Basically, this step lets you set up multiple Klipper instances for multiple 3D printers on a single CB1. The more instances you add, the more stress you'll put on your CB1. For this guide, we're keeping it at 1.

![image](https://user-images.githubusercontent.com/80109965/234786252-258142d0-f395-40ae-8967-eba36c52ef6a.png)

8. The Installation process should look like this:

![image](https://user-images.githubusercontent.com/80109965/234786859-a081dc93-1bd2-401f-8b94-4dff81ebfa37.png)

9. After successful installation, it should show up something like this:

![image](https://user-images.githubusercontent.com/80109965/234788018-257337a3-1ce5-4563-93de-7919b0ad88ed.png)

10. Press `B` to go back to the home interface of `KIAUH` and the main menu should show that `Klipper` is installed and in the repo `Klipper3d/klipper`

![image](https://user-images.githubusercontent.com/80109965/234788953-0ba800ac-e98b-4c2b-948e-34f4b614d9d9.png)

11. Press `Q` to quit and return to the terminal.

## Steps to Configure Klipper after Installation

1. Open the SSH server and log into the CB1. In the terminal, type in:

```
cd ~/klipper/
make menuconfig
```

2. Compile with the configuration shown below(if the options below is not available, 
please update you Klipper source code to the newest version):

* [*] Enable extra low-level configuration options 
* Micro-controller Architecture (STMicroelectronics STM32) ---> 
* Processor model (STM32G0B1) ---> 
* Bootloader offset (8KiB bootloader) ---> 
* Clock Reference (8 MHz crystal) ---> 
* Communication interface (USB (on PA11/PA12)) --->

![image](https://user-images.githubusercontent.com/80109965/234809363-9ac87bb1-f130-48cd-a2af-c6dc5cad3371.png)

3. Press `Q` to exit, and `Yes [Y]` when asked to save the configuration

4. Run `make` to compile firmware，”klipper.bin” file will be generated in 
`home/biqu/klipper/out` folder when make is finished, download it onto your 
computer using the ssh application.


![image](https://user-images.githubusercontent.com/80109965/234809164-5a1b3ac7-bd33-49a6-8b44-9457fdbc3e0b.png)

![image](https://user-images.githubusercontent.com/80109965/234810234-4e13aa1e-b5ca-4558-bdd6-997e9167a826.png)

5. After the `klipper.bin` is generated, access the `SFTP` window using any `SSH` client and find the `klipper.bin` file and download it into the system.

6. Then rename the `klipper.bin` to `firmware.bin` in the system. Remove the SD-card from the board and then access it using an SD-card reader in the system, and copy the `firmware.bin` into the SD-card.

7. After the SD card is flashed with the firmware.bin, eject the card and put it back into the motherboard. After this, you can expect certain possibilities, which  are listed below:

#### Possibility 1

1. The board converts the `firmware.bin` file into `firmware.cur` internally on its own. 

2. After that, get its serial ID by typing in the terminal of the SSH server: `ls /dev/serial/by-id/`. Copy the serial-ID that is generated and save it. This ID is later required to type in into the printer cfg file.

3. We have to update the firmware using the DFU method, by typing in `make flash FLASH_DEVICE=/dev/serial/by-id/xxx` which runs the bootloader and flashes the firmware and updates it. 

4. After the `file download successful` appears, ignore the other error messages that occurs below.

#### Possibility 2

1. After the SD-card is inserted into the board after the `firmware.bin` is pasted, there is a possibility that the board does not convert it to `firmware.cur` file. However this will not cause any functional issues.

2. Similarly, generate the serial ID by typing in 'ls /dev/serial/by-id/`, and it will still generate an ID. Copy down the ID for future uses and purposes.

3. The flashing and updating of firmware using the DFU method may not be possible, due to reasons giving error during the `make flash FLASH_DEVICE= /dev/serial/by-id/xxx` process, where it does not recognise the serial ID, but this does not cause any issues in the future.

4. The generated serial ID can still be put in the printer.cfg, which was generated previously itself when klipper was flashed.

## OctoKlipper plug-in installing and setting up

1. Initially, in the terminal of the SSH server terminal, install the wheel package of the Octoprint, for smooth install of the OctoKlipper plug-in, using the command `~/OctoPrint/venv/bin/pip install wheel`. After installation, head back to IP-address of the OctoPrint.

2. In the OctoPrint settings, head to `Plugin Manager` tab, where the present plugins will be displayed.

3. At the top right options, click on `+ Get More`, search `OctoKlipper` in the searchbar, and select `Install`. The plugin will be successfully installed.

4. Under the settings tab, in the `Plugins` subtab, the OctoKlipper would have been updated and present.

5. In the OctoKlipper tab, some of the settings are to be made up. Under the `Basic` section, update the serial port to `/home/biqu/printer_data/comms/klippy.serial` 

6. Deselect the `Replace Connection Panel` in order to not replace the other connection options that are available in the Connection tab.

7. Select the `Show Short Messages 'on NavBar' and 'on SideBar'`.

8. Under the `Klipper Config Directory` box, enter the pathway as `/home/biqu/printer_data/config/`, since in the case of BTT CB1, the `printer.cfg` is by default read from here.

9. In the `Klipper Base Config Filename` write in the filename as `printer.cfg`.

10. In the `Klipper Log File` enter the pathway as `/home/biqu/printer_data/logs/klippy.log`, as the log file is present in this directory. this will access the `printer.cfg` file that is present in this directory.

## Other Plugins required for OctoPrint:

The following plugins can be found in the `Plugin Manager` in the OctoPrint settings tab and on the top right, click on `+ Get More` and then search the following:

1. `Firmware Updater`

2. `Upload Anything`
