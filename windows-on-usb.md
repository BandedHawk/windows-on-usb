#  Windows 10 on USB Installation Guide
I don't run Windows as my primary OS. There are times I need to use Windows. This is when there is an application that only runs in Windows, or when I need to test something in a Windows environment. However, I don't want a dual-boot machine. So installing Windows 10 on a USB drive is the best option for me. The problem is that the installation process for Windows 10 or Windows 11 does not support installing the operating system to a USB drive. We need to use a few tools to get us there.

There are instructions on-line but I found I couldn't successfully follow them to a working solution. Additionally, I didn't really understand what was the purpose of some of the steps. I'm writing this to improve clarity. As I said, I don't use Windows day-to-day. So in this guide, I'll be creating a bootable Windows USB using a Linux environment.

#### Items
- VirtualBox 7.x
- Windows 10 or Windows 11 installation ISO media
- Rufus
- GParted
- USB drive - preferably SSD

#### Assumptions
You have already installed VirtualBox in your Linux environment. Permissions have been set up for Virtualbox and the account you are using for running VirtualBox that allow Virtualbox to control connected USB drives. The USB drive should be large enough to hold the Windows operating system and otherwise use it as the normal drive for working in the Windows environment. Refer to Microsoft's guidelines on the requirements of the operating system.
### Creating a USB with a bootable Windows 10 or Windows 11
1. Start the graphical VirtualBox Manager. Click on the **Machine** menu, and from the drop-down, select **New** to create a new virtual machine. We are going to create our Windows instance we want on the USB as a virtual machine instance first.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/virtualbox-manager.png "Create Instance")

2. VirtualBox Manager will guide you through the instance creation process. Specify the name of the instance, where it will be stored and the location of the ISO image for the Windows installation media. Check **Skip Unattended Installation** so you can manually configure the setup of the Windows environment during installation. I prefer this level of control over the initial installation of Windows.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/create-instance.png "Define Instance Information")

3. Next, define the memory to be allocated for the instance and the number of CPUs. This will only be for the purpose of running the installation of your Windows installation. When you move it to the USB and boot it properly, Windows will adjust for the actual hardware of the machine you boot it from. Make sure to allocate enough memory and CPU resources so the Windows installation runs adequately. Enable EFI otherwise you will have trouble, particularly for Windows 11 which requires definition for EFI.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/define-hardware.png "Define Hardware")

4. Define the hard drive size for the instance. This will represent the size of the image that will be copied onto the USB drive. Make sure that this size is not bigger than the size of your USB drive. For a 250 GB USB drive, you will want a virtual hard disk of your instance sized for 236 GB. Unfortunately, a 250 GB listed USB drive is misleading. A USB drive of 250 GB usually equates to 250000000 bytes.  The calculation for the Virtualbox instance is 236 x 2^20 (247463936) bytes. This setting will leave a little space for the recovery partition for Windows on the USB drive and the EFI area.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/define-hard-drive.png "Define Hard Drive")

5. After finishing the creation of the instance, boot the instance from VirtualBox Manager and start the installation process for Windows. Configure and setup the Windows instance to your liking. When done and Windows is operating correctly, power down the VirtualBox instance.

6. Locate the VDI for the VirtualBox Windows instance you have just created. At the command line inside a terminal session, run the following VirtualBox command to convert the VDI to a disk image. Adjust the source and target file names for your circumstances. The VDI name will follow the naming of your VirtualBox Windows instance. You will need as much space at the target location as the full size of the virtual hard disk you defined for the VirtualBox Windows instance. This process may take a while. It may sometimes hang as well. I recommend rebooting Linux before running the command. Place the image to a location that can be accessed by the VirtualBox Windows instance. You can define that with the VirtualBox Manager, specifying the Shared Files location. Inside the Windows instance, this will be seen as a network share.

    `VBoxManage clonemedium --format RAW "Windows USB Install.vdi" windows-usb.img`

7. We are going to prepare the USB drive for copying onto it the image we have just created. Plug in the USB drive into your Linux system. Start gparted, targeting that USB drive. Remove any partitions on the drive. We will create a new partition table of type GPT. The GUID Partition Table (GPT) was introduced with EFI. Windows 11 in particular needs a boot disk with GPT. I recommend a fast SSD USB drive and connected via a USB-C/Thunderbolt interface. That will ensure better Windows performance.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/gparted.png "GParted Main Screen")

8. In the GParted GUI, select the **Device** from the toolbar and **Create Partition Table**. Make sure the partition type selected is **gpt**. Create the new partition table by clicking **Apply**. After this, create a new partition on the drive. Close gparted when both are done.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/gparted-partition-table.png "GParted Create Partition Screen")

9. Return to VirtualBox Manager. We are going to use the Windows instance to copy the Windows image to the USB drive. We need to make the USB drive available to the Windows instance. Right click on the instance in VirtualBox Manager and select **Settings** in the pop-up menu. Click on USB. click on the **Add USB filter icon** and add the USB drive from the list that is displayed. Make sure the USB drive is plugged in before you do this.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/virtualbox-usb-settings.png "VirtualBox USB Settings")

10. Start the Windows instance in VirtualBox Manager.

11. Once logged into Windows, download rufus from [rufus.ie](https://rufus.ie/en/). We will use this to write the image onto the USB drive. We are using rufus as it can create a bootable USB drive for Windows 11. Even if you are creating a Windows 10 bootable USB system, you won't be able to upgrade to Windows 11 unless you have GPT, EFI and otherwise meet the Microsoft Secure Boot requirements. The rufus utility can do this. There are probably other utilites but I could not find a Linux utility that meets this need.

12. Once the download is complete, run the rufus executable directly. Select the **List USB Hard Drives**. The USB drive should be selected automatically as the target device after allowing USB drives to be listed, otherwise manually select the correct device. Select the image that you shared with the VirtualBox Windows instance. When the USB drive has been correctly configured, rufus should recognize the device has a GPT partition scheme and the target system is UEFI. Do NOT uncheck **Quick format**. USB drives will take a very long time to full format with rufus. Check that all the settings are similar to the example here. Click **START** to commence the image write process.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/rufus.png "Rufus Settings")

13. If everything goes well, the USB drive will have a working bootable version of Windows. Configure your system to boot from the USB drive, and the Windows environment should start. It may take a few reboots and updates for Windows to reconfigure itself for the actual hardware of the physical system.
