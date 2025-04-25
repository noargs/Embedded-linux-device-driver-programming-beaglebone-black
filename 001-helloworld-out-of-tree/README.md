### Linux Kernel module (LKM)
- Let's learn how to write a simple hello world kernel module
- Compiling the kernel module using kbuild
- Transferring kernel module to BBB, loading, and unloading     

### Linux Kernel module (LKM)
- Linux supports dynamic insertion and removal of code from the kernel while the system is up and running. The code what we add and remove at run time is called a kernel module    
- Once the LKM is loaded into the Linux kernel, you can start using new features and functionalities exposed by the kernel module without even restarting the device.      
- LKM dynamically extends the functionality of the kernel by introducing new features to the kernel such as security, device drivers, file system drivers, system calls etc. (modular approach)    
- Support for LKM allows your embedded Linux systems to have only minimal base kernel image(less runtime storage) and optional device drivers and other features are supplied on demand via module insertion      
- Example: when a hot-pluggable new device is inserted the device driver which is an LKM gets loaded automatically to the kernel        
    
### Static and dynamic LKMs
- **Static(y)**
  - When you build a Linux kernel, you can make your module statically linked to the kernel image (module becomes part of the final Linux kernel image ). This method increases the size of the final Linux kernel image. Since the module is ‘built-in’ into the Linux kernel image, you can not ‘unload’ the module. It occupies the memory permanently during run time    
     
- **Dynamic(m)**
  - When you build a Linux kernel, these modules are NOT built into the final kernel image, and rather there are compiled and linked separately to produce .ko files. You can dynamically load and unload these modules from the kernel using user space programs such as **insmod**, **modprobe** , **rmmod**.
    
> So, when you’re building the kernel, you can either link modules directly into the kernel or build them as separate modules that can be loaded into the kernel at some other time.      

### User space Vs Kernel space   

<img src="images/u-k-general.png" alt="User space Vs Kernel space">     

<img src="images/u-k-linux.png" alt="Linux User space Vs Kernel space">     

### LKM writing syntax   
```c
#include <linux/module.h>

// module initialisation entry point
static int __int my_kernel_module_init(void)
{
  // kernel's printf
  pr_info("Hello, World!\n");
  return 0;
}

// Module clean-up entry point
static void __exit my_kernel_module_exit(void)
{
  pr_info("Goodbye, World!\n");
}

// This registration of above entry points with kernel
module_init(my_kernel_module_init);
module_exit(my_kernel_module_exit);

// This is descriptive information about the module
MODULE_LICENSE("GPL");
MODULE_AUTHON("Ibn noargs")
MODULE_DESCRIPTION("A kernel module to print some messages");
```        
<img src="images/lkm_structure.png" alt="LKM structure">      
     
> [!IMPORTANT]   
> Every kernel module should include this header `#include <linux/module.h>`    
> Since you write a kernel module that is going to be executed in kernel space, you should be using kernel headers, never include any userspace library headers like C std library header files.    
> No user space library is linked to the kernel module  
> Most of the relevant kernel headers live in **linux_source_base/include/linux/**    
     
### Module initialisation function    
- The module initialization function is module-specific and should never be called from other modules of the kernel. It should not provide any services or functionalities which may be requested by other modules. Hence it makes sense to make this function private using 'static' though it is optional.
- Must return 0 on success and non zero on failure. so the module will not get loaded.   
- It is an entry point to your module and get called during boot time in case of static modules    
- In the case of dynamic modules, this function will get called during module insertion.    
- There should be one module initialisation entry point in the module.     
-  You may do some initialization of devices, there private data structures
- Requesting memory dynamically for various kernel data structures
and services
- Request for allocation of major-minor numbers   
- Device file creation   

### Module initialisation function     
- This is an entry point when the module is removed, Since you can not remove static modules, Hence clean-up function will get called
only in the case of dynamic modules when it is removed using user space command such as `rmmod`   
- If you write a module and you are sure that it will always be statically linked with the kernel, then there is no need to implement this function.   
- Even if your static module has a clean-up function, the kernel build system will remove it during the build process if there is an `__exit` marker.     
- Typically, you must do exact reverse operation what you had done in the module init function. undoing init function.    
- Free memory which are requested in init function   
- De-init the devices or leave the device in the proper state
    
### Building a kernel module (out of tree)    
- Must use `kbuild` system used by kernel, in order to stay compatible with changes, to pick up right GCC flags.  
- You must have a prebuilt kernal source availabe that contains the configuration and header files used in the build. This ensures as the developer changes the kernel configuration, his custom driver is automatically rebuilt with the correct kernel configuration   
    
> You can compile you module against one kernel version and load into another version kernel. **Thumb rule**: “Have a precompiled Linux kernel source tree on your machine and build your module against that      
   
Following command can build an external module   
```bash
$ make -C <path to linux kernel tree> M=<path to your module> [Target]
```    

For **[Target]** in above command you can use following:    
- **modules** :The default target for external modules. It has the same functionality as if no target was specified.    
- **modules_install**: Install the external module(s). The default location is `/lib/modules/<kernel_release>/extra/`, but a prefix may be added with `INSTALL_MOD_PATH`
- **clean** : Remove all generated files in the module directory only.   
- **help**: List the available targets for external modules      

### Creating a local Makefile   
As shown in above command `make -C` will take a path to linux kernel tree to invoke top level make to initialise the system and populate the right flag and values and then `-M` get to your module for local Makefile. In the local makefile you should define a kbuild variable like below
```bash
obj-<X> := <module_name>.o
```   

Here `obj-<X>` is kbuild variable and ‘X’ takes one of the below values   
**X = n** , Do not compile the module   
**X= y**, Compile the module and link with kernel image (**static**)   
**X = m** , Compile as **dynamic**ally loadable kernel module          
                     
The kbuild system will build `main.o` (`obj-m := main.o`) from `main.c` and after linking will result in the kernel module `main.ko`.    
    
### Compiling for Host kernel    
> You can compile same module against your host kernel too (to use driver in the host machine) with `uname -a` to find the current kernel version running on the ubuntu. and running `make -C /lib/modules/<kernel version from uname and append -generic>/build M=$PWD module`. Now you can use userspace program to load the kernel module `insmod main.ko` and now you can check with the `sudo dmesg`. You will see the warning there **loading out-of-true module taints kernel**. Now you can remove it with `sudo rmmod main.ko` and you will see the message with `sudo dmesg`     

### Compiling for Target kernel (BBB)       
Only difference compiling for target is providing the Arc, cross compile compiler name, and linux kernel tree location (here we didn't give **bin** folder location like *Compiling for host kernel* as above)   
```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -C /home/ibn/.bbb/source/linux_bbb_6.1/ M=$PWD modules   
```     
   
> If you face an error that cross compiler not found (Its because we use `sudo` with `make`, which messed up the path environment variable) then add the *compiler path* to `/etc/sudoers` at the end of `secure_path` which colon seperated list   
```bash
Defaults    secure_path="/usr/local/sbin:/home/ibn/.bbb/downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin"
```    
    
Now you can try above make command again 
```bash
sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -C /home/ibn/.bbb/source/linux_bbb_6.1/ M=$PWD modules   
```    

You can inspect the file as follows   
```bash
file main.ko
modinfo main.ko
arm-linux-gnueabihf-objdump -h main.ko
```       
     
### Transfer the Driver file to Beaglebone Black     
```bash
# make a directory in Beaglebone black
debian@BeagleBone:~$ mkdir drivers   

# in your Host pc Ubuntu where driver .ko file is
pc@ubuntu:~/ scp main.ko debian@192.168.7.2:/home/debian/drivers   
    
# inside Beaglebone black go to `drivers` folder
debian@BeagleBone:~$ sudo insmod main.ko   
```   