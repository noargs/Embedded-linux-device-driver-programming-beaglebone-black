# Embedded linux device driver programming   

## Host (Ubuntu) preparation   
```bash
$ sudo apt-get update 

$ sudo apt-get install build-essential lzop u-boot-tools net-tools bison flex
libssl-dev libncurses5-dev libncursesw5-dev unzip chrpath xz-utils
minicom
```       
    
To cross-compile Linux kernel, Linux application, and kernel
modules to ARM Cortex Ax architecture, we need a cross compiler.    
     
The SOC AM335x from TI, is based on ARM Cortex A8 processor of
ARMv7 architecture     
     
Download Linaro [cross compiler and toolchain](https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabihf/) for 64 bit host Ubuntu `gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz` OR simply run following commands to download and extract into your **workspace** directory.     
     
```bash
$ wget https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz   

$ mkdir -p ~/workspace && mv gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz ~/workspace && cd ~/workspace

workspace$ tar Jxf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz     
```    

Add an entry into ~/.bashrc file to export the toolchain and to make it available in the terminal.    
`export PATH=$PATH:<path_to_toolchain_binaries OR ~/workspace/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin>`    
    
### Getting your board ready    
We will create two partition on the SD Card as follows (images for `MLO`, `u-boot.img` and kernel image `uImage`, device tree `am335-boneblack.dtb` as well as environment variable file `uEnv.txt` are placed in the repo under [pre-built-images](./pre-built-images/) directory) download and place it into your **workspace** directory       
     
![Download boot images and Root filesystem](./images/boot-images-root-filesystem.png)    

Download latest Debian release [Beaglebone black SD card image](https://www.beagleboard.org/distros/am335x-12-2-2023-10-07-4gb-microsd-iot) and extract it into your **workspace** folder      
This release comes with:   
- Kernel: 5.10.168-ti-r72
- U-Boot: v2022.04
- default username:password is [debian:temppwd]   
    
```bash
workspace$ wget https://files.beagle.cc/file/beagleboard-public-2021/images/am335x-debian-12.2-iot-armhf-2023-10-07-4gb.img.xz  
workspace$ tar Jxf am335x-debian-12.2-iot-armhf-2023-10-07-4gb.img.xz
```  

> Now your workspace directory should contain **pre-buil-images**, Debian release **am335x-debian-12.2-iot-armhf-2023-10-07-4gb** as well as cross compiler tool chain from Linaro **gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf**     

Download **GParted Partition Editor** from Ubuntu *App Center* to format your SD Card      

## Target preparetion (Beaglebone black)      

Official website for [Beaglebone black](https://www.beagleboard.org/boards/beaglebone-black).  
    
[Design](https://git.beagleboard.org/beagleboard/beaglebone-black) contain Beaglebone black **Schematic** and **Reference Manual** in *Docs* folder of this repository.  

Go to [wiki](https://elinux.org/Main_Page) and find your board BeagleBone Black for ton of information)   

![Board components](./images/board_components.png)             
    
 
    




