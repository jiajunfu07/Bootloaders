# Bootloaders
I added u-boot to rpi2. For a normal rpi image from their website, it has bootloader bootcode.bin(2nd stage) and start.elf(3rd stage) already. I don't replay the start.elf with u-boot. Instead, I use the start.elf to boot the u-boot. After the u-boot is boot up, I boot the kernel in the CLI of u-boot. \
The reason I used rpi2 is I use UART to show the CLI and the UART of rpi3 is connected to BT.

## Toolchain 
Get a prebuilt toolchain from a repository that the RPi Foundation makes available
```
$ sudo apt install build-essential git
$ git clone https://github.com/raspberrypi/tools.git
$ vi ~/.bashrc
$ tail -l ~/.bashrc
export PATH=$PATH:~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin
$ source ~/.bashrc
$ echo $PATH
$ arm-linux-gnueabihf-gcc --version
```
I can see the gcc version is 4.9

## Build U-Boot
```
$ git clone git://git.denx.de/u-boot.git
$ cd u-boot
$ git checkout v2016.07
```
I tried the latest release v2018.01-rc3, but it requires gcc version above 6.0
```
$ make CROSS_COMPILE=arm-linux-gnueabihf- rpi_2_defconfig
$ make CROSS_COMPILE=arm-linux-gnueabihf-
$ ls -l u-boot*
-rwxrwxr-x 1 jiajun jiajun 2003711 4月  10 21:35 u-boot
-rw-rw-r-- 1 jiajun jiajun  338704 4月  10 21:35 u-boot.bin
-rw-rw-r-- 1 jiajun jiajun   27399 4月  10 21:35 u-boot.cfg
-rw-rw-r-- 1 jiajun jiajun    1673 4月  10 21:35 u-boot.lds
-rw-rw-r-- 1 jiajun jiajun  376492 4月  10 21:35 u-boot.map
-rw-rw-r-- 1 jiajun jiajun  338704 4月  10 21:35 u-boot-nodtb.bin
-rw-rw-r-- 1 jiajun jiajun  973864 4月  10 21:35 u-boot.srec
-rw-rw-r-- 1 jiajun jiajun  101182 4月  10 21:35 u-boot.sym
```
## Install U-Boot
Insert a SD card. Install official image from RPi website to the SD card.\
Back up my original config.txt.
```
$ sudo mount /dev/sdb1 /mnt/tmp
$ mv /mnt/tmp/config.txt /mnt/tmp/config.txt.pre-uboot
```
copy my U-Boot binary to the SD card:
```
$ cp u-boot.bin /mnt/tmp/
$ echo 'kernel=u-boot.bin' > /mnt/tmp/config.txt
$ sudo umount /mnt/tmp
```
Insert the SD card to rpi and power up the board. the U-Boot prompt should show up on the serial console.
```
U-Boot>
```

## Boot the kernel
```
U-Boot> mmc dev 0
U-Boot> ls mmc 0:1
```
I can find the kernel image is called kernel7.img and device tree is bcm2709-rpi-2-b.dtb \
Check the environment variable fdtfile, fdt_addr_r, kernel_addr_r
```
U-Boot> printenv fdtfile
fdtfile=bcm2836-rpi-2-b.dtb
U-Boot> printenv fdt_addr_r
fdt_addr_r=0x00000100
U-Boot> printenv kernel_addr_r
kernel_addr_r=0x01000000
```
Modify the variable fdtfile
```
U-Boot> setenv fdtfile bcm2709-rpi-2-b.dtb
```
Boot Linux
```
U-Boot> fatload mmc 0:1 ${kernel_addr_r} kernel7.img
reading kernel7.img
4579632 bytes read in 464 ms (9.4 MiB/s)
U-Boot> fatload mmc 0:1 ${fdt_addr_r} ${fdtfile}
reading bcm2709-rpi-2-b.dtb
16693 bytes read in 23 ms (708 KiB/s)
U-Boot> bootz ${kernel_addr_r} - ${fdt_addr_r}
```
