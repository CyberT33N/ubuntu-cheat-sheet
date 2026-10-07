# ubuntu-cheat-sheet
Ubuntu Cheat Sheet with the most needed stuff..

# USB

## Bootable


Terminal:

```bash
sudo apt update
sudo apt install isoimagewriter
```

Dann:

```bash
isoimagewriter
```

Wenn Checksum Fehler kommen einfach weiter






























# Secure Erase

## NVMe

### Variante: Ubuntu-Live-USB → NVMe löschen

Ubuntu auf USB Stick installieren und dann Try Ubuntu


  # NVMe Secure Erase – Ubuntu Live
   
  > **Ziel:** NVMe-SSD vollständig über den NVMe-Sanitize-Mechanismus löschen. **Beispiel-Platzhalter:** \- Controller: `/dev/nvmeX` \- Namespace: `/dev/nvmeXn1` \- Modell: `MODEL_PLACEHOLDER` \- Seriennummer: `SERIAL_PLACEHOLDER`
   
  ## 0\. ⚠️ WARNUNG
   
  Der Vorgang löscht die Ziel-SSD.
   
  **Vor dem Löschen IMMER Modell + Seriennummer + Device prüfen.**
   
  ---
   
  ## 1\. SSD identifizieren
   
  ```
  sudo nvme list
  ```
   
  Beispiel:
   
  ```
  Node         Generic      SN                 Model
  /dev/nvmeXn1 /dev/ngXn1   SERIAL_PLACEHOLDER  MODEL_PLACEHOLDER
  ```
   
  Merke dir:
   
  ```
  Controller: /dev/nvmeX
  Namespace:  /dev/nvmeXn1
  ```
   
  ---
   
  ## 2\. Controller-Informationen prüfen
   
  ```
  sudo nvme id-ctrl /dev/nvmeX | grep -E 'mn|sn|fr|sanicap'
  ```
   
  Kontrollieren:
   
  ```
  mn        : MODEL_PLACEHOLDER
  sn        : SERIAL_PLACEHOLDER
  sanicap   : ...
  ```
   
  ---
   
  ## 3\. Blockgerät mit Mountpoints prüfen
   
  ```
  lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
  ```
   
  Die Ziel-NVMe darf **nicht** das laufende Ubuntu-Live-System enthalten.
   
  Außerdem sollte die Ziel-SSD keine gemounteten Partitionen haben.
   
  Prüfen:
   
  ```
  mount | grep nvmeX
  ```
   
  Wenn dort Partitionen der Ziel-SSD erscheinen, diese vor dem Sanitize aushängen:
   
  ```
  sudo umount /dev/nvmeXn1p1
  ```
   
  Bei mehreren Partitionen entsprechend wiederholen.
   
  ---
   
  ## 4\. Sanitize-Fähigkeiten prüfen
   
  ```
  sudo nvme id-ctrl /dev/nvmeX | grep -i sanicap
  ```
   
  Beispiel:
   
  ```
  sanicap : 0x60000002
  ```
   
  **Nicht einfach davon ausgehen, dass jede NVMe dieselben Fähigkeiten besitzt.**
   
  Zusätzlich:
   
  ```
  sudo nvme sanitize-log -H /dev/nvmeX
  ```
   
  Vor dem Start sollte typischerweise stehen:
   
  ```
  Sanitize State : 0  Idle state
  ```
   
  und es darf kein laufender Sanitize-Vorgang vorhanden sein.
   
  ---
   
  ## 5\. Unterstützte Sanitize-Methode bestimmen
   
  ```
  sudo nvme id-ctrl /dev/nvmeX | grep -i sanicap
  ```
   
  Die konkreten unterstützten Methoden hängen vom Controller ab.
   
  Für eine SSD, die **Block Erase** unterstützt:
   
  ```
  Block Erase = supported
  ```
   
  ist der entsprechende Sanitize-Befehl:
   
  ```
  sudo nvme sanitize /dev/nvmeX -a 2
  ```
   
  `-a 2` = **Block Erase Sanitize**
   
  ---
   
  # 6\. 🚨 LETZTE KONTROLLE VOR DEM LÖSCHEN
   
  Noch einmal:
   
  ```
  sudo nvme list
  ```
   
  und:
   
  ```
  lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
  ```
   
  Vergleiche:
   
  ```
  MODEL:  MODEL_PLACEHOLDER
  SERIAL: SERIAL_PLACEHOLDER
  ```
   
  **Erst wenn Modell und Seriennummer eindeutig stimmen, fortfahren.**
   
  ---
   
  # 7\. SANITIZE AUSFÜHREN
   
  ```
  sudo nvme sanitize /dev/nvmeX -a 2
  ```
   
  Dabei wird der **Controller** verwendet:
   
  ```
  /dev/nvmeX
  ```
   
  nicht:
   
  ```
  /dev/nvmeXn1
  ```
   
  Der Vorgang kann im Hintergrund laufen.
   
  **Währenddessen:**
   
  - SSD nicht entfernen
  - Rechner nicht ausschalten
  - Vorgang nicht unterbrechen
   
  ---
   
  # 8\. Status prüfen
   
  ```
  sudo nvme sanitize-log -H /dev/nvmeX
  ```
   
  Während des Vorgangs kann der Status auf laufenden Betrieb hinweisen.
   
  Wiederholen:
   
  ```
  sudo nvme sanitize-log -H /dev/nvmeX
  ```
   
  bis der Vorgang abgeschlossen ist.
   
  ---
   
  # 9\. Erfolgreichen Abschluss verifizieren
   
  Erfolgreich ist es, wenn der Status sinngemäß meldet:
   
  ```
  Most Recent Sanitize Command Completed Successfully.
  ```
   
  und:
   
  ```
  Sanitize State : 0  Idle state
  ```
   
  Bei einem erfolgreichen Sanitize sollte außerdem der entsprechende Global-Data-Erased-Status gemäß Controller-Ausgabe geprüft werden.
   
  ---
   
  # 10\. Nachkontrolle
   
  ```
  sudo nvme list
  ```
   
  und:
   
  ```
  lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
  ```
   
  Die alte Partitionierung/Dateisystemstruktur sollte nicht mehr als nutzbare alte Installation vorhanden sein.
   
  ---
   
  # Kurzversion
   
  ```
  # 1. SSD identifizieren
  sudo nvme list
   
  # 2. Controller prüfen
  sudo nvme id-ctrl /dev/nvmeX | grep -E 'mn|sn|fr|sanicap'
   
  # 3. Blockgerät prüfen
  lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
   
  # 4. Mounts prüfen
  mount | grep nvmeX
   
  # 5. Sanitize-Fähigkeiten/Status prüfen
  sudo nvme sanitize-log -H /dev/nvmeX
   
  # 6. NUR wenn Block Erase unterstützt wird:
  sudo nvme sanitize /dev/nvmeX -a 2
   
  # 7. Ergebnis prüfen
  sudo nvme sanitize-log -H /dev/nvmeX
   
  # 8. Abschlusskontrolle
  sudo nvme list
  lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
  ```
   
  ## Wichtig
   
  **`nvmeX`****und****`nvmeXn1`****sind Platzhalter.**
   
  Beispielsweise könnte eine echte SSD sein:
   
  ```
  Controller: /dev/nvme0
  Namespace:  /dev/nvme0n1
  ```
   
  Eine andere Maschine könnte aber haben:
   
  ```
  Controller: /dev/nvme1
  Namespace:  /dev/nvme1n1
  ```
   
  **Niemals die Device-Namen blind aus diesem Cheat Sheet übernehmen. Immer zuerst mit****`nvme list`****und****`lsblk`****identifizieren.** :::
   
  Wenn du möchtest, kann ich dir auch noch eine **ultrakompakte 10-Zeilen-Version** für einen USB-Stick/Notfall-Cheat-Sheet machen.

















<br><br>
<br><br>

# Update
```
sudo apt update
sudo apt full-upgrade
sudo apt autoremove --purge
sudo apt autoclean

sudo reboot
```






<br><br>
____________________________________________
____________________________________________
<br><br>

# Terminal

## Konsole

### Select and copy with mouse click
- Man kann auf Settings gehen, dann auf Edit Current Profile und anschließend auf Maus. Dort kann man "Copy on Select" auswählen.

Dadurch hat man die Möglichkeit, Text einfach mit der linken Maustaste auszuwählen und direkt zu kopieren. Standardmäßig kann man das nicht wie bei Windows machen, also nicht mit der rechten Maustaste kopieren, was theoretisch sogar ein zusätzlicher Schritt wäre.

Einfügen kann man mit der mittleren Maustaste. Ich habe erstmal keine Möglichkeit gefunden, das auf die rechte Maustaste umzustellen.






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

### Uninstall
```shell
sudo apt remove --purge '^nvidia-.*'
sudo apt autoremove --purge
```






<br><br>
<br><br>

### Install / Update
- Wird nicht automatisch geupdated auf breaking changes bei normalen ubuntu update/upgrade. Man muss selber das unten machen

<br><br>

#### Method 1
- You should install nvidia driver via the GUI
  - https://www.linuxbabe.com/ubuntu/install-nvidia-driver-ubuntu

<br><br>

#### Method 2 (recommended)
- https://ubuntu.com/server/docs/nvidia-drivers-installation
  
```shell
sudo ubuntu-drivers list
sudo ubuntu-drivers install nvidia:590
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
sudo apt full-upgrade

sudo reboot

sudo do-release-upgrade
```

#### Upgrade to higher non lts
```shell
sudo gedit /etc/update-manager/release-upgrades

# set
# Prompt=normal
```


#### 24.10 to 25.10

#### 22 to 23
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
