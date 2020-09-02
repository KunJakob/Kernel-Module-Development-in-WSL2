# Setup for Kernel Module development using WSL2

This setup guide is intended for students at Aarhus University in the Hardware Abstractions course. Certain aspects may not be applicable to general development.

This guide assumes you have already installed WSL2. If you have not, installation guide can be found at https://aka.ms/wslinstall

For the following steps Ubuntu 20.04 LTS (from the Windows Store) is used. Linux commands are subject to change with time.

⚠️ **YOU MUST BE USING WSL2, NOT WSL1, AS WSL1 DOES NOT SHIP WITH A KERNEL** ⚠️

---

## Getting the correct linux kernel files to build against


### Create an appropriate workspace for your work.

```
cd ~

mkdir workspace

cd workspace
```

### Inside the workspace, download the kernel source code for your current kernel version

```
git clone --branch $(uname -r) --depth 1 https://github.com/microsoft/WSL2-Linux-Kernel.git
```

### install dependencies in order to compile the kernel and our kernel modules

```
sudo apt-get install bison build-essential flex libssl-dev libelf-dev bc
```

### copy existing kernel build config to new build config

```
cd ~/workspace/WSL2-Linux-Kernel

zcat /proc/config.gz > .config
```

### Compile the kernel

```
make -j $(nproc)

sudo make -j $(nproc) modules_install
```

---

## Creating a kernel module

For this section, utilizing Visual Studio Code for WSL integration is highly recommended, but is not necessary. Any text editor can be used.

### Create a code repository
This can be wherever you want- even in windows, but utilizing Visual Studio Code with the WSL extension within Linux is recommended.

Here is an example.

```
cd ~/workspace

mkdir hello-world-kernel-module

cd hello-world-kernel-module
```

### Create a .c file for your project

```
touch hello-world-module.c
```

### Edit the file with your favourite text editor/IDE


Using VSCode
```
code hello-world-module.c
```

Using Nano
```
nano hello-world-module.c
```

Using Vim
```
vim hello-world-module.c
```

Open the folder in windows explorer to use windows text editors
```
explorer.exe .
```


### Write a hello-world module

Below is some example code that will write a message to the kernel log when loaded or unloaded

```c
#include <linux/init.h>   /* Needed for module_init */
#include <linux/module.h> /* Needed by all modules */
#include <linux/kernel.h> /* Needed for KERN_INFO */

MODULE_LICENSE("MIT");

static int __init hello_world(void)
{
    printk(KERN_INFO "Hello World!\n");
    return 0;
}

static void __exit goodbye_world(void)
{
    printk(KERN_INFO "Goodbye world!\n");
}

module_init(hello_world);
module_exit(goodbye_world);
```

### Create a makefile to compile your module

```
touch Makefile
```

Open Makefile with your favourite text-editor and write the following into it

```Makefile
obj-m += hello-world-module.o
#Replace hello-world-module with the name of your .c file. You must keep the .o extension

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

### Compiling the module

```bash
make
```
This will compile your kernel module. The output file will be named `hello-world-module.ko` with .ko (Kernel Object).

to clean up your code repository of build products, run make clean instead.

```
make clean
```

### Loading the Kernel Module

```
sudo insmod hello-world-module.ko
```

### Unloading the Kernel Module

For this, do not include the file extension .ko

```
sudo rmmod hello-world-module
```

---

# Reading the kernel log output in WSL2 Ubuntu 20.04

### Starting the system log

```
sudo service rsyslog start
```

### listening to the system log

After running this command, try loading and unloading your kernel module. You should see your printk logging.
```
tails -f /var/log/syslog
```
