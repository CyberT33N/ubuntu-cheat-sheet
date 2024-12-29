# ubuntu-cheat-sheet
Ubuntu Cheat Sheet with the most needed stuff..













<br><br>
____________________________________________
____________________________________________
<br><br>

# Nvidia

<br><br>


## Driver
- You can find a list of nvidia driver here and check there if your graphiccard is supported
  - https://wiki.ubuntuusers.de/Grafikkarten/Nvidia/nvidia/


<br><br>

### Install / Update

<br><br>

#### Method 1
- You should install nvidia driver via the GUI
  - https://www.linuxbabe.com/ubuntu/install-nvidia-driver-ubuntu

<br><br>

#### Method 2 (recommended)
- https://ubuntu.com/server/docs/nvidia-drivers-installation
  
```shell
sudo ubuntu-drivers list
sudo ubuntu-drivers install nvidia:550
sudo reboot
```
- You can do the same for updating :) 




<br><br>

#### Check if new driver version is available
```shell
# Get current driver
nvidia-smi

# Check if driver update is available
ubuntu-drivers devices
```












<br><br>
<br><br>

### Suspend not working anymore
- If you suspend and it directly wake up after it then try:
```
sudo systemctl stop nvidia-suspend.service
sudo systemctl stop nvidia-hibernate.service
sudo systemctl stop nvidia-resume.service

sudo systemctl disable nvidia-suspend.service
sudo systemctl disable nvidia-hibernate.service
sudo systemctl disable nvidia-resume.service

sudo rm /lib/systemd/system-sleep/nvidia
```








## Cuda & cuDNN

### Install

#### Ubuntu 23.04
- Install first latest nvidia driver (https://github.com/CyberT33N/linux-cheat-sheet/blob/main/README.md#nvidia)
```shell
sudo apt update
sudo apt install build-essential

# check if it worked
gcc --version
g++ --version

sudo apt install nvidia-cuda-toolkit nvidia-cuda-toolkit-gcc nvidia-cudnn

# Check if it worked
nvcc --version
```































<br><br>
_______________
_______________
<br><br>


## Kernel

### ubuntu 24.04

### Install Kernel 6.10.3



```shell
uname -m
```
- Ergebnisse:
    - x86_64: Dein System ist AMD64 (64-Bit Intel/AMD).
    - aarch64: Dein System ist ARM64 (64-Bit ARM).
    - i686 oder i386: Dein System ist 32-Bit Intel/AMD.

If you have x86_64 then download and install these links in following order:
- https://kernel.ubuntu.com/mainline/v6.10.3/amd64/linux-headers-6.10.3-061003_6.10.3-061003.202408281533_all.deb
- https://kernel.ubuntu.com/mainline/v6.10.3/amd64/linux-headers-6.10.3-061003-generic_6.10.3-061003.202408281533_amd64.deb
- https://kernel.ubuntu.com/mainline/v6.10.3/amd64/linux-image-unsigned-6.10.3-061003-generic_6.10.3-061003.202408281533_amd64.deb
- https://kernel.ubuntu.com/mainline/v6.10.3/amd64/linux-modules-6.10.3-061003-generic_6.10.3-061003.202408281533_amd64.deb

In the terminal run:
```shell
sudo update-grub
```

Reboot and select the kernel from the bootloader menu














<br><br>
<br><br>

## Update

<br><br>

### Update after Ubuntu version is not supported anymore
- https://askubuntu.com/questions/91815/how-to-install-software-or-upgrade-from-an-old-unsupported-release/91821#91821
```shell
sudo sed -i -re 's/([a-z]{2}\.)?archive.ubuntu.com|security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo apt-get update && sudo apt-get dist-upgrade
```








<br><br>
<br><br>

## Upgrade
```
sudo apt update
sudo apt upgrade
```

<br><br>

### Dist Upgrade
```
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
sudo do-release-upgrade
```


#### Upgrade 22 to 23
- Try first the default dist upgrade method explained above and if it not working like you get an error hat upgrade is not supported then try this.
- Works aswell for 23 to 24
- Make sure that while the install process when it will ask you to update grub that you keep the local version instead by replacing it with a new one. This is necesarry when you have custom boot options with grub like e.g. disk encrytption. Everything else is straight forward and can be just be confirmed all the time with yes.
```
Modify your current /etc/apt/sources.list and replace the urls starting with http://archive... by http://old-releases
Once done, execute sudo apt update then sudo apt upgrade and sudo apt dist-upgrade
Then reboot
Modify the /etc/apt/sources.list and put back archive instead of old-releases
Modify the /etc/apt/sources.list and replace all occurences of kinetic by lunar
Then perform sudo apt update, sudo apt upgrade and sudo apt dist-upgrade. This will ensure that you upraded to 23.04
Then reboot
Then do a sudo do-release-upgrade to upgrade to 23.10
```






<br><br>
<br><br>

## benchmark
- https://www.youtube.com/watch?v=dD4-8NVonVM

<br><br>
<br><br>

## Themes
- https://itsfoss.com/install-themes-ubuntu/
- https://www.gnome-look.org
- https://www.pling.com/p/1238824/

```
mkdir ~/.themes
mkdir ~/.icons

sudo apt install gnome-tweaks gnome-shell-extensions

reboot
```

<br><br>



## Known Problems

<br><br>

#### Random freezes
- system settings/ Display and Monitor/ Compositor switching from OpenGL to XRender seems to work.

<br><br>

#### Black screen after installation (https://askubuntu.com/questions/1085807/black-screen-after-installation-of-ubuntu-18-04)
- mediately after the BIOS/UEFI splash screen during boot, with BIOS, quickly press and hold the Shift key, which will bring up a GNU GRUB menu screen. With UEFI press (perhaps several times) the Esc key to get to the GNU GRUB menu screen. Sometimes the manufacturer's splash screen is a part of the Windows bootloader, so when you power up the machine it goes straight to the GNU GRUB menu screen, and then pressing Shift is unnecessary.
  - Press e to enter editing mode
    - Immediately after this string replace ro quiet splash by nomodeset quiet splash. This change is only temporary — it will just be used once and GRUB won't remember it in the future. Press Ctrl+X or F10 to boot with the nomodeset option that was added. If you make a mistake, press Esc to go back to the previous screen.
      
      - How to permanently set kernel boot options on an installed OS?
      - https://askubuntu.com/questions/207175/what-does-nomodeset-do
        - You can easily add this to permanent because it does not affect your gpu performance on your host later**
        ```bash
        sudo gedit /etc/default/grub
        GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
        sudo update-grub
        ```







<br><br><br><br>


## Install .deb files
```bash
# will also install all needed dependencies
sudo apt install *****.deb

# will only install the file
sudo dpkg -i ****.deb
```

<br><br>

## Install .run files
```bash
sudo chmod +x /path/to/file.run
sudo /path/to/file.run
```

















<br><br>

## PPA (Personal Package Archive)

<br><br>

#### remove PPA
```bash
sudo add-apt-repository --remove ppa:whatever/ppa

# As example
sudo add-apt-repository --remove http://ppa.launchpad.net/michael-gruz/canon-trunk/ubuntu
```


<br />
<br />


 _____________________________________________________
 _____________________________________________________


<br />
<br />



## Convert all files with windows line breaks to linux line breaks
```bash
find . -type f -exec dos2unix {} \;
```


<br />
<br />


 _____________________________________________________
 _____________________________________________________


<br />
<br />
