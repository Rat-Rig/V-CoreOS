# Fysetc Spider v1.1

## Wiring
![Fysetc Spider V1.1 Wiring Diagram](_media/spider-11-wiring.png)

!> If you have been testing your Spider without the stepper drivers plugged in, there is a chance that you'll blow the 3.3V voltage regulator on the board if you do not discharge the capacitors before connecting the drivers. The lesson here is don't power up the Spider without the stepper drivers plugged in. Please read https://github.com/FYSETC/FYSETC-SPIDER/blob/main/Spider%203.3v%20issue.md

!> If you use the Ratrig endstop switches and cables, do **not** blindly plug them in to your Spider as doing this will short the board's 3.3V supply rail.  You will probably have to swap the outer two wires (red and white) on the board end of the cable but double check this.

## Firmware installation

For the first time install of Klipper there are two methods.  Via
SSH/USB or with an SD Card.  Once klipper is installed, future updates
can be installed from V-CoreOS.

### via SSH/USB

Make sure your board is connected to the Pi (USB-C on the Spider, USB-A on the Pi). Connect with SSH (putty) to the Pi (login pi, password raspberry if you did
not change the defaults).

Fysetc provide instructions on installing Klipper here: https://github.com/FYSETC/FYSETC-SPIDER#42-Klipper
but here is the sequence that worked for the author.

Put a jumper between 3.3V and BT0 on the Spider.

![Fysetc Spider V1.1 BT0 Jumper](_media/BTO-jumper.png)

Press the reset button on the Spider.

On the Pi, run the following commands:

	lsusb

You should see a device in DFU mode listed. This is your Spider ready to have the firmware
uploaded.

Then run:

	cd ~klipper
	make menuconfig

(the firmware should be configured as per the instructions provided by Fysetc in
the link above)

Then run:

	make clean
	make
	sudo service klipper stop
	dfu-util -a 0 -s 0x08000000:leave -D ~/klipper/out/klipper.bin

You should see the firmware being written to your Spider.

Now remove the jumper between 3.3V and BT0.  Press the reset button on the Spider.

Run "lsusb" again and you should see a device by the name "OpenMoko, Inc.". This is your Spider running Klipper.

run the command "sudo service klipper start".


### via SD Card

Errrr....... 

Copy the `firmware-spider-11.bin` file from the release page to the SD card that goes into your control board and call it `firmware.bin`, then insert the SD card in to the control board.

?> 
You can verify if the board flashed correctly by checking if the firmware.bin file has been changed to firmware.CUR on the SD card. If you have trouble flashing the motherboard, start unplugging your wires beginning with the endstops, sometimes faulty wiring can cause the board to not boot properly.

?> Once you have verifed the board has been succesfully flashed, you don't have to reinsert the SD card.

If you're going through initial setup please continue in the [installation guide](installation.md#setup)
## Firmware upgrade

Sometimes klipper makes changes to the microcontroller code and thus your MCU need to be reflashed with new firmware. You can do that in 2 ways.

### SD Card
If you're not used to the command line or haven't used SSH before, the easiest way is to download the new firmware file from the github release page. New firmware files will uploaded to the latest release when the klipper firmware changes, this is a manual process though and might not be immediately available. Therefore the recommended method is [flashing via usb](#flashing-via-usb)

?> 
You can verify if the board flashed correctly by checking if the firmware.bin file has been changed to firmware.CUR on the SD card. If you have trouble flashing the motherboard, start unplugging your wires beginning with the endstops, sometimes faulty wiring can cause the board to not boot properly.

?> Once you have verifed the board has been succesfully flashed, you don't have to reinsert the SD card.

### Flashing via USB (Recommended)
Another option is to SSH into the pi using something like PuTTy or `ssh pi@v-coreos.local` via the commandline on OS X and Linux machines. Execute `~/klipper_config/config/boards/fysetc-spider/make-and-flash-mcu.sh` and the Pi will compile the klipper firmware and flash the board for you. This has the benefit that it will always recompile the firmware to match your klipper version, so you are not reliant upon the V-CoreOS developers to upload a new firmware binary for you.

!> Be sure to remove the SD card from the board before attempting to flash, if one is in there.

## Known Problems

If V-CoreOS complains that it can't open "/dev/fysetc-spider" run the following command on your Pi:

	sudo ln -s /home/pi/klipper_config/config/boards/fysetc-spider/*.rules /etc/udev/rules.d/

Then disconnect the USB cable, reconnect it and /dev/fysetc-spider should exist. You only need to do this once.