# Debian on RISC-V

*Work in progress!*

A set of scripts to build a (somewhat) working Debian image for RISC-V. This 
includes (somewhat) working GDB! 

## Setting up host build environment

* Add Debian Unstable (Sid) repositories to your system:

      printf "Package: *\nPin: release a=unstable\nPin-Priority: 10\n" | sudo tee /etc/apt/preferences.d/unstable.pref      
      printf "deb http://ftp.debian.org/debian unstable main\ndeb-src http://ftp.debian.org/debian unstable main\n" | sudo tee /etc/apt/sources.list.d/unstable.list
      apt-get update

* Install QEMU and `mmdebstrap` (req'd to build root filesystem and run installed system):

      apt-get install udo apt-get install mmdebstrap/unstable qemu-user-static/unstable qemu-system-riscv64/unstable binfmt-support/unstable debian-ports-archive-keyring gcc-riscv64-linux-gnu rsync

## Checking out source code

```
git clone https://github.com/janvrany/riscv-debian.git
git -C riscv-debian submodule update --init --recursive
```

## !!! BIG FAT WARNING !!!

Scripts below do use sudo quite a lot. *IF THERE"S A BUG, IT MAY WIPE OUT 
YOUR SYSTEM*. *DO NOT RUN THESE SCRIPTS WITHOUT READING THEM CAREFULLY FIRST*. 

They're provided for convenience. Use at your own risk.

## Building Linux kernel image

* Run:

  ```
  ./debian-mk-kernel.mk
  ```

  This will leave QEMU bootable kernel image (BBL + kernel image) in 
  `bbl-q`. The image for *HiFive Unleashed* is `bbl-u`, *QEMU image
  simply won't boot* !!!

## Building Debian filesystem image

* Create a file containing Debian root filesystem. This is optional,
  you may use directly a device (say `/dev/mmcblk0p2`) or ZFS zvolume (`/dev/zvol/...`). You will need at least 4GB of space, preferably more. To make plain file image: 

  ```
  truncate -s 4G debian.img
  /sbin/mkfs.ext3 debian.img
  ```

* Install Debian into that image:

  ```
  ./debian-mk-rootfs.sh debian.img  

  ```

  Please note, that Rebian repository for RISC-V arch is really
  shaky, at times `apt-get` may fail because unsatisfiable dependencies. In that case, either wait or fiddle about somehow.

* Install GDB:

  ```
  ./debian-mk-gdb.sh debian.img    
  ```  

  Note, that this may (will) take a lot, lot of time when using QEMU. If you 
  intend to use Debian on real hardware, e.g., *HiFive Unleashed*, you may
  want to compile GDB manually there. To do so,  follow the steps in 
  `./debian-mk-gdb.sh` script. 

## Run Debian in QEMU

* Execute: 

  ```
  ./qemu-fire.sh
  ```

## Creating SD Card for HiFive Unleashed

Following steps assumes the SD card is properly partioned. If not,
please follow steps at the bottom of in [freedom-u-sdk/Makefile][5].
Then...

* To install kernel on SD card (say `/dev/mmcblk0`)

  ```
  ./unleashed-install-kernel.sh /dev/mmcblk0p1
  ```

* To install Debian root filesystem on SD card (say `/dev/mmcblk0`)
 
  ```
  ./unleashed-install-rootfs.sh debian.img /dev/mmcblk0p2
  ```
Now take your SD card, insert it into *Unleashed* and hope for the best. 

## References
* [https://wiki.debian.org/RISC-V][1]
* [https://github.com/jim-wilson/riscv-linux-native-gdb/blob/jimw-riscv-linux-gdb/README.md][2]
* [https://groups.google.com/a/groups.riscv.org/forum/#!msg/sw-dev/jTOOXRXyZoY/BibnmSTOAAAJ][3]
* [https://wiki.debian.org/InstallingDebianOn/SiFive/HiFiveUnleashed#Building_a_Kernel][4]
* [https://github.com/sifive/freedom-u-sdk/issues/44][6]
* [https://github.com/rwmjones/fedora-riscv-kernel][7]
* [https://github.com/andreas-schwab/linux][8]
* [https://forums.sifive.com/t/linux-4-20-on-hifive-unleashed/1955][9]

[1]: https://wiki.debian.org/RISC-V
[2]: https://github.com/jim-wilson/riscv-linux-native-gdb/blob/jimw-riscv-linux-gdb/README.md
[3]: https://groups.google.com/a/groups.riscv.org/forum/#!msg/sw-dev/jTOOXRXyZoY/BibnmSTOAAAJ
[4]: https://wiki.debian.org/InstallingDebianOn/SiFive/HiFiveUnleashed#Building_a_Kernel
[5]: https://github.com/sifive/freedom-u-sdk/blob/master/Makefile#L228
[6]: https://github.com/sifive/freedom-u-sdk/issues/44
[7]: https://github.com/rwmjones/fedora-riscv-kernel
[8]: https://github.com/andreas-schwab/linux
[9]: https://forums.sifive.com/t/linux-4-20-on-hifive-unleashed/1955

