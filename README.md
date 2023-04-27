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



