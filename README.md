# Intel for display Nvidia for computing

### Basic problems:

Use intel for display and Nvidia for computing.

If use prime-select nvidia, then nvidia GPU are both used as display and computing.

If use prime-select intel, then nvidia-smi and deviceQuery will not found the Nvidia GPU in system.



### 0. Platform

- Intel UHD Graphics 770 in 13700K
- Nvidia 3060Ti Founder Edition
- Ubuntu 22.04.1 LTS (Jammy Jellyfish)



### 1. Initial settings

Connect display to mother board, and enable iGPU from BIOS if needed.



### 2. Main Steps

- Blacklist nvidia-drm module. Don’t need to re-install the nvidia-driver. Only add the following to `/etc/modprobe.d/blacklist-nvidia.conf`.

```bash
# /etc/modprobe.d/blacklist-nvidia.conf
blacklist nvidia-drm
alias nvidia-drm off
```



- Run `$ sudo prime-select nvidia`. Do not use `$ sudo prime-select intel` or `on-demand`, otherwise the Nvidia GPU will not be powered.

  

- Add ‘nogpumanager’ kernel parameter. 

  - Go to /etc/default/grub

  - Edit `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"` to 

    `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nogpumanager"`

  

- Create /etc/X11/xorg.conf

```bash
# /etc/X11/xorg.conf file from reference link 1
Section "Device"
    Identifier     "intel"
    Driver         "modesetting"
    BusID          "PCI:0:2:0"
EndSection
```

On Ubuntu 18.04 or lower you may need to set the driver to `intel` instead of `modesetting`.

The key point is the "BusID" option. It indicates the PCI bus id that the Intel graphics connects to. It can be retrieved from `lspci`. For example, on my computer, `lspci` outputs `00:02.0 VGA compatible controller: Intel Corporation Device 3e92`, thus the bus id is `0:2:0`.

Note that the bus id output from `lspci` is hexadecimal but the number filled in `xorg.conf` should be decimal. For example, if the output from `lspci` is `82:00.0 VGA com...`, you need to fill `PCI:130:0:0` in the configuration.

My own `xorg.conf` is also attached.

- reboot



**Reference links:

1. [ubuntu 18.04+headless_390+intel iGPU after prime-select intel lost contact to GeFORCE 1050ti](https://forums.developer.nvidia.com/t/ubuntu-18-04-headless-390-intel-igpu-after-prime-select-intel-lost-contact-to-geforce-1050ti/66698)

2. [Use intel for display nvidia for computing](http://litaotju.github.io/2019/03/13/=Use-intel-for-display-nvidia-for-computing/)





### 3. Additional:

#### 1. Intel UHD Graphics driver

Normally, you don't need to worry about the iGPU driver with ubuntu. For 11th, 12th, and 13th gen Intel CPU, download the proper driver from: [Intel® Arc™ Graphics Driver - Ubuntu*](https://www.intel.com/content/www/us/en/download/747008/intel-arc-graphics-driver-ubuntu.html) -- only supports Ubuntu 22.04.



#### 2. Nvidia driver fast installation

Go to "Software&Updates -> Additional Drivers", select the proper version and "Apply Changes".

* If there is no Nvidia driver available, try this first: 

```bash
$ sudo add-apt-repository ppa:graphics-drivers/ppa
```

NVIDIA PPA should be added and the APT package repository cache should be updated.
