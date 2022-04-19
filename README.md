**Build a firmware**

1\. First we need to make sure the dependencies are installed (for Debian/Ubuntu):

run "sudo apt install subversion g++ zlib1g-dev build-essential git python python3 python3-distutils libncurses5-dev gawk gettext

unzip file libssl-dev wget libelf-dev ecj fastjar java-propose-classpath"

For Ubuntu 18.04 or later: run "sudo apt install build-essential libncursesw5-dev python unzip"

2\. Get the source code:

run "git clone https://haiphu@bitbucket.org/buiquocbao/linuxsystemforrouter.git"

cd linuxsystemforrouter

make menuconfig

If you want to build images for the "Turris Omnia" Wifi-Router, select:

"Target Profile" ⇒ "Turris Omnia"

for Clearfog Pro, select: "Target Profile" ⇒ "SolidRun ClearFog Pro"

3\. Build the images

In the menu, select Exit and then Yes to save your settings. Now build the images. That may take some time:

run "make"

**Flash U-boot and Openwrt**

* For clearfog use SDCARD

1\. After the build is complete, the image is stored in the / bin directory.

Right-click and select Extract Here to uncompress image

2\. This will extract a single file with the same name. Right-click this new file and select Open With > Disk Image Writer.

3\. Make sure your SD card is plugged in and select it from the list, then click Start Restoring....

4\. this requires root privileges, so type in your password and click Authenticate.

5\. When it is done, click the eject button and remove the SD Card.

6\. Insert a SD Card into your device.

* For clearfog use EMMC

1\. Preparation

Ubuntu with TFTP server

Ehternet cable

U-boot: u-boot-clearfog-pro-uart.kwb will be in the main directory and u-boot-spl.kwb will be in the bin directory.

Openwrt image: you can find openwrt-mvebu-cortexa9-solidrun_clearfog-pro-a1-squashfs-sdcard.img.gz in /bin/targets/mvebu/cortexa9. Then unzip the file.

kwboot

3\. Load U-boot into device's RAM

When the board is not power on, changing SW1 to UART boot mode. Plug in USB cable in Debug port, open your serial terminal and run this command in the directory contain kwboot:

$ sudo ./kwboot -t -B 115200 /dev/ttyUSB0 -b u-boot-clearfog-pro-uart.kwb

Then power on your board and you will have following output:

Sending boot message. Please reboot the target...|

Sending boot image...

0 % [......................................................................]

0 % [......................................................................]

1 % [......................................................................]

............

96 % [......................................................................]

97 % [......................................................................

98 % [......................................................................]

99 % [............................................................]

kwboot will start a terminal after the transfer is finished. You will see U-Boot information and should press Enter to stop the automatic boot progress and come to the next step.

4\. Flash Openwrt image into device's eMMC

Now the U-Boot is running and the hardwares such as Ethernet are initialised. Plugin the Ethernet cable and setup the IP addresses:

=> setenv serverip <server ip>

=> setenv ipaddr <board ip>

=> saveenv

Saving Environment to MMC...

Writing to MMC(0)... done

=>

You could use ping command to check the connection

For example, 192.168.1.14 is Server IP

=> ping $serverip

host 192.168.1.14 is alive

Now you could download openwrt image

=> tftpboot openwrt-mvebu-cortexa9-solidrun_clearfog-pro-a1-squashfs-sdcard.img

ethernet@70000 Waiting for PHY auto negotiation to complete.... done

Using ethernet@70000 device

TFTP from server 192.168.1.14; our IP address is 192.168.1.99

Filename 'openwrt-mvebu-cortexa9-solidrun_clearfog-pro-a1-squashfs-sdcard.img'.

Load address: 0x800000

Loading: ################################################## 27.2 MiB

4.1 MiB/s

done

Bytes transferred = 28514116 (1b31744 hex)

=>

Note the two import numbers: Load address: 0x800000 and Bytes transferred = 28514116. The second one indicates 28514116 / 512 = 55692 (0xD98C) blocks we should copy

Switch back to the main physical partition and start flash:

=> mmc dev 0 0

switch to partitions #0, OK

mmc0(part 0) is current device

=> mmc write 0x800000 0 0xD98C

MMC write: dev # 0, block # 0, count 55692 ... 55692 blocks written: OK

Finally power off your board and switch back to SD/eMMC boot mode.

5\. Flash U-boot and Openwrt into eMMC

After rebooting and appearing OpenWrt logo, you run command to write u-boot into eMMC:

echo 0 > /sys/block/mmcblk0boot0/force_ro

dd if=u-boot-spl.kwb of=/dev/mmcblk0boot0

Finally, U-boot and Openwrt are in eMMC.