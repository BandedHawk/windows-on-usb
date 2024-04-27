#Putting Windows 10 on USB
I don't run Windows as my primary OS. There are times I need to use Windows. This is when there is an application that only runs in Windows, or when I need to test something in a Windows envrionment. However, I don't want a dual-boot machine.So installing Windows 10 on a USB drive is the best option for me. The problem is that the installation process for Windows 10 or Windows 11 does not support installing the operating system to a USB drive. We need to use a few tools to get us there.

There are instructions on-line but I found I couldn't successfully follow them to a working solution. Additonally, I didn't really understand what was the purpose of some of the steps. I'm writing this so it makes it a bit more clear. As I said, I don't use Windows day-to-day. So in this guide, I'll be creating a Windows USB from a Linux environment.

####Items
- VirtualBox 7.x
- Windows 10 or Windows 11 installation iso or media
- Rufus
- GParted
- USB drive - preferably SSD

####Assumptions
You have already installed VirtualBox in your Linux environment. Permissions have been set up for Virtualbox and the account you are using for running VirtualBox allow Virtualbox to control connected USB drives. The USB drive should be large enough to hold the Windows operating system. Refer to Microsoft's guidelines on the rewuirements of the operating system.
###Creating a Virtual Machine With Windows 10 or Windows 11
1. Start the graphical VirtualBox Manager. Click on the **Machine** menu, and from the drop-down, select **New** to create a new virtual machine. We are going to create our Windows instance we wanton the USB.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/virtualbox-manager.png "Create Instance")

2. VirtualBox Manager will guide you through the instance creation process. Specify the name of the instance, where it will be stored and where the iso image for the Windows installation media. Check **Skip Unattended Installation** so you can manually configure the setup of the Windows environment during installation. I prefer this level of control of the initial installation of Windows.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/create-instance.png "Define Instance Information")

3. Next, define the memory to be allocated for the instance and the number of CPUs. This will only be for the purpose of running the installation of your Windows installation. When you move it to the USB and boot it properly, Windows will adjust for the actual hardware of the machine you boot it from. Make sure to allocate enough memory and CPU resources so the Windows installation runs adequately. Enable EFI otherwise you will have trouble, particularly for Windows 11 which requires EFI for security.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/define-hardware.png "Define Hardware")

4. Define the hard drive size for the instance. This will represent the size of the image that will be copied onto the USB drive. Make sure that this size is not bigger than the size of your USB drive. For a 250 GB USB drive, you will want a virtual hard disk of your instance sized for 236 GB. Unfortunately, a 250 GB. A hard drive of 250 GB actually equates to 250000000 bytes.  The calculation for the Virtualbox instance is 236 x 2^20^. This setting will leave a little space for the recovery partition for Windows on the USB drive.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/define-hard-drive.png "Define Hard Drive")

5. After finishing the creation of the instance, boot the instance and start the installation process for Windows. Configure and setup the Windows instance to your liking. When done and Windows is operating correctly, power down the VirtualBox instance.

6. Locate the VDI for the VirtualBox Windows instance you have just created. At the command line inside a terminal session, run the following VirtualBox command to convert the VDI to a disk image. Adjust the source and target file names for your circumstances. The VDI name will follow the naming of your VirtualBox Windows instance. You will need as much space at the target location as the full size of the virtual hard disk you defined for the VirtualBox Windows instance. This process may take a while. It may sometimes hang as well. I reccommend rebooting Linux before running the command. Place the image to a location that can be accessed by the VirtualBox Windows instance. You can define that with the VirtualBox Manager, specifying the Shared Files location. Inside the Windows instance, this will be seen as a network share.

    `VBoxManage clonemedium --format RAW "Windows USB Install.vdi" windows-usb.img`

7. We are going to prepare the USB drive for copying the image we have just created onto it. Plug in the USB drive into your Linux system. Start gparted, targeting that USB drive. Remove any partitions on the drive. We will create a new partition table of type GPT. The GUID Partition Table (GPT) was introduced with EFI. Windows 11 in particular needs a boot disk with GPT. I recommend a fast SSD USB drive. That will ensure better Windows performance.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/gparted.png "GParted Main Screen")

8. In the GParted GUI, select the **Device** from the toolbar and **Create Partition Table**. Make sure the partition type selected is **gpt**. Create the new partition table by clicking **Apply**.After this, create a new partition on the drive. Close gparted when both are done.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/gparted-partition-table.png "GParted Create Partition Screen")

9. Return to VirtualBox Manager. We are going to use the Windows instance to copy the Winows image to the USB drive. We need to make the USB drive available to the Windows instance. Right click on the instance in VirtualBox Manager and select **Settings** in the pop-up menu. Click on USB. click on the **Add USB filter icon** and add the USB drive from the list that is displayed. Make sure the USB drive is plugged in before you do this.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/virtualbox-usb-settings.png "VirtualBox USB Settings")

10. Start the Windows instance in VirtualBox Manager.

11. Once logged into Windows, download rufus from [rufus.ie](https://rufus.ie/en/). We will use this to write the image onto the USB drive.

12. Once the download is complete, run the rufus executable directly. Select the **List USB Hard Drives**. The USB drive should be selected automatically for the de, otherwise mally select the correct device. Select the image that you shared with the VirtualBox Windows instance. f the USB drive has been correctly configured, rufus should recognise the device has a GTP partition scheme and the target system is UEFI. Do NOT uncheck **Quick format**. USB drives will take a very long time to format with rufus. Check that all the settings are similar to the example here. Click start to commence the image write process.

![alt text](https://raw.githubusercontent.com/BandedHawk/windows-on-usb/master/images/rufus.png "Rufus Settings")

13. If everything goes well, the USB drive will have a working bootable version of Windows. Configure your system to boot from the USB drive, and you should have a working Windows environment. It may take a few reboots and updates for Windows to reconfigure itself for the actual hardware of the physical system.
