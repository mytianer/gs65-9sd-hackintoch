
# MSI GS65 8SE Hackintosh Guide

Hi! This is a guide on how to install macOS on your MSI GS65. I will specifically show how to install on an MSI with Coffeelake series chipset, eg. 8750H, but if any other MSI owners have issues with their hack feel free to create an issue for any help.

**DONT JUST COPY THE EFI FOLDER SINCE I CANNOT GUARANTEE IF ITWILL WORK FOR YOUR SYSTEM. FOLLOW THE GUIDE AND IF YOU'RE HAVING TROUBLE USE MY EFI AS A GUIDE.**
# Table of Contexts
- [MSI GS65 8SE Hackintosh Guide](#msi-gs65-8se-hackintosh-guide)
- [Table of Contexts](#table-of-contexts)
- [Hardware](#hardware)
- [Issues That Can't Be Fixed (as of now)](#issues-that-cant-be-fixed-as-of-now)
- [Issues I Currently Have (that could possibly be fixed)](#issues-i-currently-have-that-could-possibly-be-fixed)
- [Upgrading from Mojave to Catalina](#upgrading-from-mojave-to-catalina)
- [Requirements](#requirements)
- [Pre-Installation](#pre-installation)
    - [Bios Menu Setup](#bios-menu-setup)
    - [USB Setup](#usb-setup)
  - [Installing Clover](#installing-clover)
- [Building the installer:](#building-the-installer)
    - [Mojave](#mojave)
    - [Catalina](#catalina)
- [Installation](#installation)
- [Post-Installation](#post-installation)
    - [Mounting EFI Partition](#mounting-efi-partition)
    - [Keyboard Brightness keys fix](#keyboard-brightness-keys-fix)
    - [Audio Fix](#audio-fix)
    - [Backlight Fix](#backlight-fix)
    - [Disable the Nvidia GPU](#disable-the-nvidia-gpu)
    - [Fix Sleep and prevent DDGPU turning back on after sleep](#fix-sleep-and-prevent-ddgpu-turning-back-on-after-sleep)
    - [USB Ports Improvement and USB Properties](#usb-ports-improvement-and-usb-properties)
    - [Battery Status](#battery-status)
    - [Bluetooth fix](#bluetooth-fix)
    - [Computer loading when inputting password](#computer-loading-when-inputting-password)
    - [Cannot control RGB Keyboard](#cannot-control-rgb-keyboard)
    - [Multi-Gesture touchpad support](#multi-gesture-touchpad-support)
- [Troubleshooting](#troubleshooting)


# Hardware
* I7-8750H
* Intel HD 630 & Nvidia GeForce GT 1060
* 16G RAM (16GBX1)
* 512GB NVMe for Windows boot
* 256GB NVMe for Mac OS Mojave boot
* KILLER E2500 (RJ45)
* Killer(R) Wireless-AC WiFi (Replaced with BCM94352Z)
* 15.6" FHD (1920x1080), 144Hz, IPS-Level
* Thunderbolt 3.0 Type C

# Issues That Can't Be Fixed (as of now)
* RTX 2060 (Apple does not support optimus/nvidia on Mojave)
* HDMI and miniDP (atleast on my computer, they seem to be routed to the 2060)
* Stock WiFi card does not work.

# Issues I Currently Have (that could possibly be fixed)
* Backlight does not save after restarting (probably because of emulated NVRAM)

# Upgrading from Mojave to Catalina
 If you currently have Mojave on your MSI Laptop, make sure you have the latest Kexts installed and also make sure you have "rename H_EC to EC" in your config.plist ACPI->DSDT->Patches.

# Requirements
* 8GB USB Drive
* macOS Mojave downloaded from the App Store (will update with new versions soon)
* Replace Stock WiFi with BCM94352Z if you want WiFi support, else you will need to use Ethernet port.
* Access to a Mac

# Pre-Installation

### Bios Menu Setup 

* Advanced
  *   Sata mode = AHCI (if it's on raid create a new issue on Github and don't continue with this guide)
  *   VT-d = Disabled

* Boot 
  *   Fastboot = disabled
  *   Boot mode = UEFI with CSM

* Security
  *   Secure boot = disabled


### USB Setup

 ***For this step you will need a mac to install Clover Bootloader and the macOS Installer to the USB.***
  * First find what is your USB disk identifier: <br> 
    * In terminal type: <br>
        ``` 
        $ diskutil list 
        ```
        
        In my case this is what it shows:
        ```
        ➜ diskutil list
            /dev/disk0 (internal):
            #:                       TYPE NAME                    SIZE       IDENTIFIER
            0:      GUID_partition_scheme                         512.1 GB   disk0
            1:                        EFI SYSTEM                  314.6 MB   disk0s1
            2:         Microsoft Reserved                         134.2 MB   disk0s2
            3:       Microsoft Basic Data Windows                 322.9 GB   disk0s3
            4:           Windows Recovery                         943.7 MB   disk0s4
            5:       Microsoft Basic Data D1gital Zro             165.6 GB   disk0s5
            6:           Windows Recovery                         22.3 GB    disk0s6

            /dev/disk1 (internal):
            #:                       TYPE NAME                    SIZE       IDENTIFIER
            0:      GUID_partition_scheme                         240.1 GB   disk1
            1:                 Apple_APFS Container disk2         239.8 GB   disk1s1

            /dev/disk2 (synthesized):
            #:                       TYPE NAME                    SIZE       IDENTIFIER
            0:      APFS Container Scheme -                      +239.8 GB   disk2
                                            Physical Store disk1s1
            1:                APFS Volume macOS                   44.6 GB    disk2s1
            2:                APFS Volume Preboot                 44.7 MB    disk2s2
            3:                APFS Volume Recovery                510.4 MB   disk2s3
            4:                APFS Volume VM                      8.6 GB     disk2s4

            /dev/disk3 (external, physical):
            #:                       TYPE NAME                    SIZE       IDENTIFIER
            0:     FDisk_partition_scheme                        *8.0 GB     disk3
            1:             Windows_FAT_32 ESD-USB                 8.0 GB     disk3s1

        ```
        The usb is located at /dev/disk3. ***Please make sure that you are selecting the correct usb as you can format your whole drive if you select the wrong disk.***
  * Format the USB: <br>
    * We have to repartition the USB as GPT in terminal (or you can repartition as MBR)
        ```
        # repartition /dev/disk3 GPT, one partition
        # EFI will be created automatically
        # second partition, "install_osx", HFS+J, remainder
        $ diskutil partitionDisk /dev/disk3 1 GPT HFS+J "install_osx" R
        ```

        You should get something like this: <br>
        ```
        $ diskutil partitionDisk /dev/disk3 1 GPT HFS+J "install_osx" R

        Started partitioning on disk3
        Unmounting disk
        Creating the partition map
        Waiting for partitions to activate
        Formatting disk3s2 as Mac OS Extended (Journaled) with name install_osx
        Initialized /dev/rdisk3s2 as a 7 GB case-insensitive HFS Plus volume with a 8192k journal
        Mounting disk
        Finished partitioning on disk3
        /dev/disk3 (external, physical):
        #:                       TYPE NAME                    SIZE       IDENTIFIER
        0:      GUID_partition_scheme                        *8.0 GB     disk3
        1:                        EFI EFI                     209.7 MB   disk3s1
        2:                  Apple_HFS install_osx             7.7 GB     disk3s2
        ```
##  Installing Clover
  * Download the latest Clover installer from sourceforge: http://sourceforge.net/projects/cloverefiboot/ (version 5070 or higher)

  * Once you have the Clover installer, run it and make sure you select `change install location `  and select `install_osx`.
  * From there, click on customize and select:
    *  Clover for UEFI booting only
    *  Install Clover in the ESP
    *  UEFI Drivers 
       *  Recommended drivers (select all in recommended drivers)
       *  File system drivers
          * APFSDriverLoader 
          * VBoxHFS (Don't forget this or else we wont be able to see the installer)
        * Memory fix drivers
          * OsxAptioFixDrv (make sure you just have this selected in memory fix) 
  * After that's done, click install.
  * Now that the installation is finished, we still ned to configure the clover files.

  * Preparing kexts and config.plist:
    * Remove all but "Others" folder in EFI/CLOVER/kexts/. We don't need the macOS versions.
    * You can install the kexts from my EFI folder to EFI/CLOVER/kext/Other, but for reference I will link the necessary kexts to at least boot to installer
      * [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases "Virtual SMC") (Apple SMC emulation)
      * [VoodooPS2](https://github.com/acidanthera/VoodooPS2/releases "VoodooPS2") (keyboard and mouse)
      * [Lilu](https://github.com/acidanthera/Lilu/releases "Lilu") (allows use of the plugins)
      * [Whatevergreen](https://github.com/acidanthera/WhateverGreen "Whatevergreen") (Injects video graphics)
      * [USBInjectAll](https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/ "USBInjectAll") (usb injection)
      * [AtherosE2200Ethernet](https://github.com/Mieze/AtherosE2200Ethernet/releases "AtherosE2200Ethernet") (ethernet connection)
    * If you already switched your WiFi card with a BCM94352Z, then you can also install:
    
* [AirportBrcmFixp](https://github.com/acidanthera/AirportBrcmFixup/releases "AirportBrcmFixup") (WiFi Fix for BCM94352Z)
      
  
  * Next you will need to install a custom config.plist from Rehabman's config.plist [repository](https://github.com/RehabMan/OS-X-Clover-Laptop-Config) and selecting the right graphics hardware configuration. Or if you know you have a UHD 630 then you can also use my config.plist. More info [here](https://www.tonymacx86.com/threads/guide-booting-the-os-x-installer-on-laptops-with-clover.148093/)
  
* After you selected your clover file, copy the file and paste it in EFI/Clover. ***Make sure the file is renamed to config.plist or else clover will not accept the file.***
  
    * If you got the config file from somewhere else other than the one I provided, then you need to open the config.plist and go to ACPI-> DSDT ->Patches and add the RTC bug fix (applies only to coffeelake)
	    ```
	    <dict>
			<key>Comment</key>
			<string>RTC Fix</string>
			<key>Disabled</key>
			<false/>
			<key>Find</key>
			<data>
			oAqTU1RBUwE=
    		</data>
			<key>Replace</key>
    		<data>
    		oAqRCv8L//8=
			</data>
        </dict>               
        ```

# Building the installer:
### Mojave
* Download macOS Mojave from the AppStore 
* Once that is finished, open Terminal and run: <br>
​```
sudo "/Applications/Install macOS Mojave.app/Contents/Resources/createinstallmedia" --volume  /Volumes/install_osx --nointeraction
​```
* Once it's finished copying the installer to the usb, rename the USB using Terminal:
​```
sudo diskutil rename "Install macOS Mojave" install_osx
​```
* After that's done, eject the USB and you're ready to install macOS.

### Catalina
* Download macOS Catalina from the AppStore 
* Once that is finished, open Terminal and run: <br>
​```
sudo "/Applications/Install macOS Catalina.app/Contents/Resources/createinstallmedia" --volume  /Volumes/install_osx --nointeraction
​```
* Once it's finished copying the installer to the usb, rename the USB using Terminal:
​```
sudo diskutil rename "Install macOS Catalina" install_osx
​```
* After that's done, eject the USB and you're ready to install macOS.

# Installation
  * Plug in the usb to one of the USB ports and turn on the computer.
  * As soon as the MSI Logo shows up, press F11, or whichever key takes you to select the usb partition)
  * Once Clover loads, click on "install_osx" and let it load. ***Note:*** If for some reason it takes longer than 10 minutes to load or if it restart's your computer, before you select "install_osx" press spacebar and select -v for Verbose mode and open an issue ticket and I will help you with the issue.
  * After the installation loads, click on Disk utility and Format your drive where you want macOS as APFS (if you're installing on a SSD)
  * Once the format is done, click "Install macOS Mojave" and follow the steps and it will restart after it's done verifying. 
  * Once it restart's go back to clover bootloader and click again "install_osx". This time it will extract the files to the hard drive. Once the installation finishes, it will restart again.
  * Once the laptop restarts, go to clover bootloader and select "Boot OS X from YOUR PARTITON NAME". Your partiton name will show depending oh what you typed during disk utility

Congratulations! You installed macOS Mojave on your computer!
# Post-Installation
As you may have noticed, Clover Bootloader loads from a USB, but it would be more convenient loading Clover Bootloader from the HDD instad of the USB.

If you would like to know how to do that you can follow this [link](https://www.insanelymac.com/forum/topic/310038-manually-install-clover-and-configure-boot-priority-with-easyuefi-in-windows/)

There are still a few issues that need to be fixed. For instance:
  * Brightness keys not working
  * Audio not working
  * Battery status not woring
  * USB ports not configured properly
  * Bluetooth not working properly (if you have BCM94352Z WiFi card)
  * Sleep/Wake issues
  * Nvidia card running even though it's not supported (thus it wastes battery)
  * Computer loading when inputting password
  * Controlling RGB keyboard


Fortunately I have fixes for these issues:

### Mounting EFI Partition 
  * Open Terminal and type 
    ```
    ~
    ➜ diskutil list
    ```
  * Get the disk destination of the EFI. Once you found it, type
    ```
    // Replace X and Y with the correct number where the EFI is located
    ~
    ➜ sudo diskutil mount /dev/diskXsY
    ```
  * You should be able to access your EFI folder.

### Keyboard Brightness keys fix
* Download [SSDT-KEYS.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-KEYS.aml?raw=true)
*   Place it in /EFI/Clover/ACPI/patched/
*   Restart system and you should have key brightness fixed.


### Audio Fix
  *   Download AppleALC from my repository (since the patch is still not in Acidanthera's AppleALC)
  *   Install AppleALC.kext to /EFI/Clover/kext/other
  *   After moving to the right location, go back to /EFI/Clover and open config.plist.
  *   Navigate to Device -> Properties -> PciRoot(0)/Pci(0x1f,3) and on add add "layout-id". Make sure that value type is on Number. 
  *   If you have audio codec ALC1220, in the "property value type "34" without quotation marks. If you have another audio codec, see the supported layout values [here](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs)
  *   Save the config.plist and restart your mac. 
  *   Once it's booted up go to system preferences->sound->output and select "internal speakers" and you should have audio fixed. 

### Backlight Fix
*   Download [SSDT-PNLF.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-PNLF.aml?raw=true).
*   Place it in /EFI/Clover/ACPI/patched/
*  Save and restart system and you should have brightness fix.

### Disable the Nvidia GPU
* Download [SSDT-DDGPU.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-DDGPU.aml?raw=true)
*   Place it in /EFI/Clover/ACPI/patched/
*   Restart system and you should have your gpu turned off (white light only shows in power button now).

### Fix Sleep and prevent DDGPU turning back on after sleep
* Download [SSDT-PTSWAK.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-PTSWAK.aml?raw=true) and [SSDT-RMCF.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-RMCF.aml?raw=true)
*   Place it in /EFI/Clover/ACPI/patched/
*   Open config.plist and go to ACPI->DSDT->patches and add the following 2 patches:
    ```
    <dict>
      <key>Comment</key>
      <string>change Method(_PTS,1,N) to ZPTS, pair with SSDT-PTSWAK.aml</string>
      <key>Disabled</key>
      <false/>
      <key>Find</key>
      <data>X1BUUwE=</data>
      <key>Replace</key>
      <data>WlBUUwE=</data>
    </dict>

    <dict>
      <key>Comment</key>
      <string>change Method(_WAK,1,N) to ZWAK, pair with SSDT-PTSWAK.aml</string>
      <key>Disabled</key>
      <false/>
      <key>Find</key>
      <data>X1dBSwE=</data>
      <key>Replace</key>
      <data>WldBSwE=</data>
    </dict>
    ```
*   Save and restart system and you should have sleep issue working fine and dgpu will not turn on after wake.

### USB  Ports Improvement and USB  Properties
* Remove USBInjectAll.kext and download USBPorts.kext from my repository and download [SSDT-XOSI.aml](https://github.com/ErrorErrorError/msi-gs65-8SE-hackintosh/blob/master/ACPI/patched/SSDT-XOSI.aml?raw=true)
*   Place USBPorts.kext in /EFI/Clover/kexts/Other/ and place SSDT-XOSI.aml in /EFI/Clover/ACPI/patched/
* Open config.plist with Xcode and go to ACPI->DSDT->Patches and add the folllowing patch: 
    ```
    <dict>
      <key>Comment</key>
      <string>change _OSI to XOSI</string>
      <key>Disabled</key>
      <false/>
      <key>Find</key>
      <data>X09TSQ==</data>
      <key>Replace</key>
      <data>WE9TSQ==</data>
    </dict>
    ```
* Save and restart system and you should have proper USB injection with correct USB power, assuming that you are using SMBios Macbook15,2.

### Battery Status
* Download and grab [SMCBatteryManager.kext](https://github.com/acidanthera/VirtualSMC/releases) from the VirtualSMC folder.
* Place SMCBatteryManager.kext in /EFI/Clover/kexts/other/
* Restart and you should get battery status.

### Bluetooth fix
* ***You must have BCM94352Z installed on your computer***
* If you're on macOS Mojave:
  * Download [BrcmPatchRam](https://bitbucket.org/RehabMan/os-x-brcmpatchram/downloads/)
  * Once it's finished extracting, copy BrcmFirmwareData.kext and BrcmPatchRam2.kext to /EFI/CLOVER/kexts/other
* Else if you're on macOS Catalina: 
  * Either compile the latest build from [Acidanthera's BrcmPatchRam](https://github.com/acidanthera/BrcmPatchRAM) or download my repository.
  * After building/downloading the necessary files, copy BrcmFirmwareData.kext, BrcmPatchRam3.kext, and BrcmBluetoothInjector and paste them to /EFI/Clover/kexts/other
* Restart and you should have bluetooth working

### Computer loading when inputting password
  * Download [NoTouchID.kext](https://github.com/al3xtjames/NoTouchID)
  * Once it's finished downloading, place it in /EFI/Clover/kexts/other/
  * Restart and your mouse will not freeze in Login screen or when you have to input password.

### Cannot control RGB Keyboard
Unfortunately SteelSeries did not create a driver to control RGB keyboard on macOS. Fortunately I was able to create a gui to change your rgb keyboard. It's still in development so be aware of bugs that you might encounter.

The app is [SSKeyboardHue](https://github.com/ErrorErrorError/SSKeyboardHue). 

### Multi-Gesture touchpad support
Coming soon...

# Troubleshooting
If you have any issues or question about your installation or you're coming from anotehr MSI laptop, feel free to open a new issue and I will be happy to help. 

Also if there is any issues with any of the installation steps that I provided or if you'd like to contribute feel free to do so. 

Thanks to Rehabman, acidanthera, and may other people for allowing hackintosh to be possible.

**will try to make it more universal soon.**