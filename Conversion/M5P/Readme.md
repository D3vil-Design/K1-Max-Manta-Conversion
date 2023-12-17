<center><H1> Required Items before starting:</h1></center> 



| Items:      | Sites To Source from: |
| ----------- | ----------- |
| RS232 Adapter 2x      | Title       |
| Right Angle USB (See image) 2x of these   | Text        |
| USB Breakout Board 1x (Optional) | Text        |
| FTDI PH female adapter (optional)   | Text        |
| FTDI XH female adapter (optional)   | Text        |
| Manta M5P + CB1 + 3x 2209  | Text        |


## Preparation: 


Remove the bottom Metal cover from your K1 (this is the 4x rubber feet and 2x small silver m3 screws)
CAREFULLY  Remove the screws from the main board and unscrew the terminal plugs for the power/hotbed
Once you can move the mainboard away from the chassis slightly you can start removing the glue on all the plugs. I used a pair of sharp point tweezers to accomplish this job. You need to be very careful during this step 
Start undoing each connector on the Creality Mainboard VERY carefully making sure not to damage any connectors
Once you have all connectors and cables free from the Creality mainboard you can set this aside and start preparing your M5P


## M5P:

### Physical Setup of M5P:

Refer to the image below for jumpers that you need to install on the M5P:



Once you have these jumpers installed you can proceed to screw the M5p into the printed adapter for your K1

When wiring the M5p the connections are as follows: 




Toolhead GND goes to the boards power input GND


Setting up the CB1:
Head over to : https://github.com/bigtreetech/CB1/releases
Download the CB1_Debian11_Klipper_kernel*.***.img.xz (Make sure it is not the one that says minimal)

Use an application such as balena etcher to write this to an SD card
Once this image is written do not forget to setup the system.cfg file to configure your wifi/timezone/klipperscreen rotation before the first bootup.


Insert the SD card into the receptacle on the M5P

Wiring The RS232 Adapters: 
|RS232: <-|->Bed PCB Cable:|
|-------|--------------|
|TX |→ White|
|RX |→ Green|
|GND |→ Gnd (Take note this ground must be shared to the Ground for the power as well as to the RS232 dongle)|


|RS232: <-|->Bed PCB Cable:|
|---------|----------------|
|RX |→ White|
|TX |→ Green|
|GND |→ Gnd (Take note this ground must be shared to the Ground for the power as well as to the RS232 dongle)|

<span style="color:red;"> The Rs232 adapters require a shared ground for the device they are talking to please make sure the grounds they share are appropriate (IE Bed cable ground should be shared at RGB1 header. Toolhead ground should be shared at board main ground) </span>


#### At this point you should be ready to boot up your M5P and let it load for the first time. After approx 5-10 min you should see the M5p on your wifi (as long as you did not forget to use the system.cfg before ejecting the SDCard) 

Once you can see this machine on your network you need to SSH into it in order to setup the klipper patches for K1 and flash the M5P with updated klipper.bin

Run this command in your SSH session:

wget https://github.com/sbtoonz/k1_klipper/raw/master/InstallKlipper.sh && sh InstallKlipper.sh

### Now you can proceed with flashing the M5P:
## Please follow these steps 

In your SSH Session 
```
cd ~/klipper
make menuconfig
```


Use the following options:
```
Enable Extra Low Level Options: X
STM32 architecture
STM32G0B1 SOC
8kib bootloader
8mhz crystal
USB On PA11/PA12
```

Press Q to exit the menu config and save the configuration

Then type:
```
make
```
And press enter.

Once this is complete run:
```
ls /dev/serial/by-id/*
```
Take note of the path it reports it should be something like:
```
/dev/serial/by-id/usb-123-port0
```

Then run:
```
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/the-path-you-found-in-the-last-step
sudo service klipper start
```

Once you have completed these steps again run the Command :
```
ls /dev/serial/by-id/*
```

And take note of the path it returns (it can vary after flashing the firmware)

Use this path in your printer.cfg 
