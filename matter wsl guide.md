{\rtf1\ansi\ansicpg1252\cocoartf2639
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 # Matter setup for Windows (WSL)\
\
This guide is intended to assist Amazon Alexa Smart Home solution architects with enabling matter on an Amazon issued Windows 10 Laptop.  Multiple online resources have been used to compile this guide and are referenced in each section when applicable.  For questions or additions please feel free to use the comment section in this document. \
\
## Install WSL and enable WSL 2\
\
Ensure virtualization is enabled in your laptops bios.\
Open Windows 10  Task Manager (ctrl, shift, esc keys)\
Look under performance tab/cpu\
in bottom right you will see \'93virtualization\'94 if it is enabled you will not need to enable it in your bios \
\
[Image: Image.jpg]\
If virtualization is not enabled you will need to find your specific model and bios information and enable it in the bios there are multiple guides for this here is an example https://helpdeskgeek.com/how-to/how-to-enable-virtualization-in-bios-for-intel-and-amd/\
\
From start menu search open \'93Turn windows features on or off\
Add check mark in \'94windows subsystem for linux\'93\
Add check mark in \'94virtual machine platform\'94\
Click OK button\
[Image: Image.jpg]\
\
\
#### Set default WSL to v2\
\
\
Open powershell as administrator\
set WSL to default to v2\
\
```\
`wsl --set-default-version 2`\
\
```\
\
\
[Image: Image.jpg]\
install wsl kernel update\
\
download update here and install:\
https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi\
\
[wsl_update_x64.msi](https://api.quip-amazon.com/2/blob/YZP9AABscvi/LciqpXlAVrKtKdG79NdfuA?name=wsl_update_x64.msi&oauth_token=WkFPOU1BeFYzbTg%3D%7C1709667663%7CPLl6Xun6cjpip%2FXy1vY5O9MBAIX31kYJybmRUkV8lwU%3D&s=sgbsAejA0utt) \
Reboot Windows 10\
\
Open microsoft store and install ubuntu 22.04.1\
\
\
[Image: Image.jpg]\
\
Run ubuntu from start menu\
Set the username password for ubuntu\
\
\
\
## Clone and activate Matter\
\
\
Original repo MD guide [here](https://github.com/project-chip/connectedhomeip/blob/v1.0-branch/docs/guides/BUILDING.md) - some steps listed in original guide are not applicable and/or incorrect\
\
**Note:** commands in this section will run significantly faster when not attached to a vpn and on higher performance x86 hardware\
\
#### Install ubuntu dependencies\
\
```\
\
sudo apt-get update -y\
\
sudo apt-get install git gcc g++ pkg-config libssl-dev libdbus-1-dev \\\
 libglib2.0-dev libavahi-client-dev ninja-build python3-venv python3-dev \\\
 python3-pip unzip libgirepository1.0-dev libcairo2-dev libreadline-dev generate-ninja cmake\
 \
https://[github.com/project-chip/connectedhomeip/blob/v1.0-branch/docs/guides/BUILDING.md](http://github.com/project-chip/connectedhomeip/blob/v1.0-branch/docs/guides/BUILDING.md)\
\
```\
\
#### Clone matter repo in ubuntu \
\
```\
cd $HOME\
mkdir matter \
git clone -b v1.0-branch --recurse-submodules https://github.com/project-chip/connectedhomeip.git\
\
```\
\
#### Setup and set source for matter in Ubuntu\
\
```\
\
cd connectedhomeip/\
source scripts/activate.sh\
\
\
\
```\
\
#### Build source tree for matter in Ubuntu\
\
```\
gn gen out/host -args='is_debug=false'\
ninja -C out/host\
```\
\
if you want a specific version of matter SDK your partner is using for example v1.0.0 use below steps to checkout and update submodules\
\
\
```\
`sudo git checkout v1.0.0`\
`sudo git submodule update \'97init`\
`source scripts/activate.sh`\
```\
\
### Setup Espressif SDK\
\
From Espressif IDF guide [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/establish-serial-connection.html):\
\
#### Install usb/serial driver on windows 10\
\
This windows driver allows windows to use the USB port as a serial interface and can be downloaded from Silabs [here](https://www.silabs.com/documents/public/software/CP210x_Universal_Windows_Driver.zip)\
\
\
#### Install espressif IDF on ubuntu (version 4.4.3 is current known good branch)\
\
```\
cd $HOME/matter\
$ git clone -b v4.4.3 --recursive https://github.com/espressif/esp-idf.git\
cd esp-idf\
git submodule update --init\
./install.sh\
```\
\
\
In /$HOME/.profile edit file so running \'93get_idf\'94 will source IDF\
\
\
```\
alias get_idf='. $HOME/matter/esp-idf/export.sh'\
export IDF_PATH=$HOME/matter/esp-idf\
export PATH="$IDF_PATH/tools:$PATH"\
```\
\
Exit from the command line and open Ubuntu again\
\
```\
get_idf\
```\
\
#### Enable IDF caching (speeds up builds)\
\
export IDF caching value\
export IDF_CCACHE_ENABLE=1\
\
\
\
### Test Connection to ESP32 in Windows - Port setup\
\
\
Configure serial port in windows device manager\
Plug in your esp32 device using a known good usb to microusb cable that supports data transfer (not a charging only cable)\
\
115200 and xon/xoff and note which port it is using for use in putty\
\
Open Windows10 device manager and expand Ports (COM & LTP) in tree\
[Image: Image.jpg]\
\
Note the com port used by Silcon Labs CP210x USB to UART Bridge (COM?)\
In port properties (right click on port and then click properties) set speed to 115200 and flow control to  xon/xoff\
[Image: Image.jpg][Image: Image.jpg]\
\
#### Test connectivity to ESP32 from Windows -  Putty\
\
Putty is used to test connectivity to the esp32 device.\
[Install putty](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.78-installer.msi) for windows\
\
Open putty\
Configure session to use serial connection set speed to 115200 and serial line to your COM port from earlier step\
[Image: Image.jpg]\
\
Plug in esp32 device to Windows PC usb c or a port (ensure data cable is used and not a charging cable)\
Click Open connection and if all is well you should see a terminal with a curser\
[Image: Image.jpg]\
Reset the esp32 device using reset button (esp32c3)\
logs from your device should display in the putty terminal window (may be different depending on what was originally flashed on your device)\
[Image: Image.jpg]\
\
#### Install  serial port mapping tool on Windows 10\
\
This tool allows for usb/uart port to be passed through to WSL\
\
Install uspid drivers on windows 10\
https://github.com/dorssel/usbipd-win/releases\
\
[usbipd-win_2.4.1.msi](https://api.quip-amazon.com/2/blob/YZP9AABscvi/FbCVTE8uHc7nD5zjf7245w?name=usbipd-win_2.4.1.msi&oauth_token=WkFPOU1BeFYzbTg%3D%7C1709667663%7CPLl6Xun6cjpip%2FXy1vY5O9MBAIX31kYJybmRUkV8lwU%3D&s=sgbsAejA0utt) \
\
#### \
install USB/serial port device references on ubuntu\
\
```\
\
sudo apt install linux-tools-virtual hwdata\
sudo update-alternatives -install /usr/local/bin/usbip usbip 'ls /usr/lib/linux-tools/*/usbip | tail -n1' 20\
\
\
```\
\
If the commands above do not work, try the following:\
\
```\
sudo apt-get update\
sudo update-alternatives --install /usr/local/bin/usbip usbip $(command -v ls /usr/lib/linux-tools/*/usbip | tail -n1) 20\
```\
\
\
\
### Map Windows10 serial port to Ubuntu\
\
In order to map the USB/serial port from windows to Ubuntu the device needs to be  \'93attached\'94 in WSL\
\
In Windows10 powershell (as Admin) run\
\
```\
usbipd wsl list\
```\
\
\
Find the Silicon Labs USB to UART Bridge (COM) BUSID (in this example it is 1-3)\
attach to bus-id of silicon labs uart bridge (example is bus 1-3, yours may be different)\
\
\
```\
usbipd wsl attach --busid 1-3\
```\
\
\
Run usbipid wsl list again and verify it now shows as attached\
usbipd wsl list\
[Image: Image.jpg]\
To detach the device \
\
```\
usbipd wsl attach --busid 1-3\
```\
\
#### \
In Ubuntu make port writable\
\
\
In order to allow idf.py to access the port in ubuntu change permissions on the device folder\
**Note**: you may need to run this command again if/when the device is unplugged or un-attached\
\
```\
\
sudo chmod 0666 /dev/ttyUSB0 \
```\
\
\
where ttys# is the com port example here is for USB0\
\
#### Pre build setup for Expressif  on Ubuntu install cmake\
\
sudo apt install cmake\
\
\
[Follow Play with Matter](https://quip-amazon.com/g0bqAlzCElLz/Play-with-Matter) instructions for building matter and testing\
\
#### Build Example\
\
Example of building and flashing the all-clusters app\
\
in the all-clusters-app/esp32 directory:\
\
```\
\
idf.py set-target esp32c3\
idf.py flash\
```\
\
or to just build:\
\
```\
idf.py set-target esp32c3\
idf.py build\
```\
\
bin files (3) that can be used in flashing will be located in the connectedhomeip/examples/all-clusters-app/esp32/ build, partition_table, bootloader, directories (3)  by default\
\
Once the device is succesfully flashed you can run\
\
```\
idf.py monitor\
```\
\
\
to see the device output on the screen (including it\'92s QR code url)\
\
to exit the monitor press keys ctrl +t then ctrl +x\
\
### Additional tools and QOL add-ons\
\
#### Android development kit\
\
Android development kit can be installed enabling building of chip-tool and other useful applications by following the build guide [here](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/android_building.md#linux).  When building for android ensure you use the device and android version specified in the [build guide](https://github.com/project-chip/connectedhomeip/blob/master/docs/guides/android_building.md#linux).  Not following the exact device and version will result in failed builds.\
\
\
#### Android chip-tools\
\
This tool is useful/convenient as it has an android phone app/gui which interacts directly with the esp32 matter device without the need of a PC.  \
Older APK built on Master branch:\
\
[chip-app.apk](https://api.quip-amazon.com/2/blob/YZP9AABscvi/7yeITRbb9Z8s-2KuoURx_g?name=chip-app.apk&oauth_token=WkFPOU1BeFYzbTg%3D%7C1709667663%7CPLl6Xun6cjpip%2FXy1vY5O9MBAIX31kYJybmRUkV8lwU%3D&s=sgbsAejA0utt) \
\
Newer APK built on version 1.0 branch:\
\
\
[chiptool_v10.apk](https://api.quip-amazon.com/2/blob/YZP9AABscvi/W2fhMSyAWmGPLFvTGWCmnA?name=chiptool_v10.apk&oauth_token=WkFPOU1BeFYzbTg%3D%7C1709667663%7CPLl6Xun6cjpip%2FXy1vY5O9MBAIX31kYJybmRUkV8lwU%3D&s=sgbsAejA0utt) \
\
\
Google branded/modified chiptool\
 https://github.com/google-home/sample-app-for-matter-android/releases\
\
\
#### idf.py menuconfig\
\
\
\'93Graphical\'94 interface to configuring esp32 device.  There are **lots** of options/settings in this component including many which will break your ability to flash until reset.\
\
You can change the QR code of the esp32 device in menuconfig using the following  menu options:\
 \
>Chip Device Layer\
>>Device Identification Options\
>>>Device Vendor Id (increment this by one)\
>>>Device Product Id (increment this by one)\
\
Save using S and after erasing then flashing device it will have a new QR code\
\
#### esptool.py\
\
This tool comes with the Expressif IDF and can be used to merge individual bin files into a consolidated bin file.  This can be useful when building on a device that cannot natively flash but are able to build using vscode remote container development detailed [here](https://github.com/project-chip/connectedhomeip/blob/master/docs/VSCODE_DEVELOPMENT.md) (M1 Macs).  Once merged the online browser based expressif tool can be used or esptool can be used on a supported system.\
\
Example of merging all 3 images into a single bin:\
Merge command:\
\
```\
esptool.py --chip ESP32-S3 merge_bin -o merged-flash.bin --flash_mode dio --flash_size 4MB 0x0 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin 0xf000 build/ota_data_initial.bin 0x20000 build/chip-all-clusters-app.bin\
```\
\
merged bin for esp32-c3 device (built on m1):\
[merged-flash.bin](https://api.quip-amazon.com/2/blob/YZP9AABscvi/XYf5oRix4gzHIUPK1Hn1iw?name=merged-flash.bin&oauth_token=WkFPOU1BeFYzbTg%3D%7C1709667663%7CPLl6Xun6cjpip%2FXy1vY5O9MBAIX31kYJybmRUkV8lwU%3D&s=sgbsAejA0utt) \
\
Example of flashing with esptool:\
\
```\
esptool.py -p /dev/cu.usbserial-110 -b 460800 write_flash 0x00000 merged-flash.bin\
```\
\
#### Expressif Launchpad\
\
Browser based tool which allows for flashing images and monitoring output of ESP32 devices\
https://espressif.github.io/esp-launchpad/\
\
#### WSL tray monitor\
\
Simple tray monitor for windows 10 that allows for starting/stopping/status of WSL instances\
https://apps.microsoft.com/store/detail/wsl-tray-monitor/9P781RW2VM6G?hl=en-us&gl=us&rtc=1\
\
\
#### Expressif Tools and VSCode plugin\
\
\
[ESP-IDF Tools installer](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/windows-setup.html#get-started-windows-tools-installer) - Windows installer that installs and sets up IDF on Windows machines\
[VSCode plugin](https://github.com/espressif/vscode-esp-idf-extension) - Plugin for vscode which enables monitor and runnable tasks in vscode\
\
## Notes and comments\
\
To further streamline this process we have fully configured WSL export file which includes all the above steps including android development kit.  An accelerated guide may be created using this export and the few steps (usbipd and usb/uart windows driver installation) if hosting of 63GB file is available.\
\
There is a WIP[mac m1 guide](https://quip-amazon.com/YI6oASqCFyYo/Matter-device-setup-for-M1-mac)which can be used to build but not natively flash esp32 devices.  Flashing can be achieved via esptool with expressif SDK installed on the M1.  Since the build is running via vscode remote containers (docker x86 containers) on m1 bootstrapping/building takes an extremely long time (18 hours).\
\
\
\
}