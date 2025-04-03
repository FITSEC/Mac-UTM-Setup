# Setting up x86 GDB Debugging for ARM Architecture

## Last Updated: March 6th, 2025

## Setup

Download Ubuntu LTS Server 22.04.5 for AArch64 [here](https://cdimage.ubuntu.com/releases/22.04/release/)  
Download and install the latest version of UTM [here](https://mac.getutm.app)

Install UTM by opening the DMG file and dragging the application into the applications folder  
![](/images/image1.png)  
Once copied, unmount the UTM drive from Finder or your Desktop and move the DMG file to the trash.

## Installation

Open UTM from spotlight search or the applications folder.  
![][/images/image2.png]  
**Select “Create a New Virtual Machine”**  
![][/images/image3.png]  
**Select “Virtualize”**  
**![][/images/image4.png]**  
**Select “Linux”**  
**![][/images/image5.png]**  
**Enable “Use Apple Virtualization”, “Enable Rosetta (x86\_64 Emulation)**  
**Under “Boot ISO Image”, select “Browse..” and locate and select your previously downloaded Ubuntu ISO**

**The rest of the setup options can be left default unless you wish to change them**

**Boot up your virtual machine via the Start/Play button.**

**Continue through the prompts and be sure to select the following when it appears**   
**![][/images/image6.png]**

**Once complete, select reboot.**  
**After rebooting, you may remove the Ubuntu ISO from UTM’s homepage.**

**Log into the virtual machine and run** 

**ifconfig**

**Obtaining the ip address of the virtual machine:**  
**![][/images/image7.png]**  
*Ex: 192.168.64.4*

**Run the following command to logout of the current session**

**logout**

**Minimize the UTM window and open up a Terminal window and run the following command:**

**ssh \<username\>@\<ip\>**

**Enter your password when prompted**  
**![][/images/image8.png]**  
*Ex: ssh pierson@192.168.64.4*

**Once logged-in, run the following commands**

**sudo apt upgrade**  
**dpkg \--add-architecture 'amd64'**  
**sudo apt update**

**Install your text editor of choice\* with the following:**  
**sudo apt install nano**  
**or**  
**sudo apt install vim**  
**or**  
**sudo apt install something-else**

*\*We will be utilizing nano for the rest of this document*

**Run the following commands**  
**sudo apt install gdb-multiarch**  
**sudo apt install binfmt-support**  
**sudo apt install libc6:amd64**  
**sudo apt install gcc:amd64**  
**sudo mkdir /media/rosetta**  
**sudo mount \-t virtiofs rosetta /media/rosetta**

## Mounting the Rosetta drive on boot

We just ran the “**mount**“ command which mounts our drive temporarily  
To make the drive mount on boot, run the following commands  
**sudo su**  
*(Enter your password)*  
**nano /etc/fstab**  
Navigate to the end of the file and add the following line  
**rosetta /media/rosetta virtiofs ro,nofail 0 0**

Press Control+x, y, and then enter to save and exit nano.

Run the following command to logout of root:  
**exit**

## Installation *(cont.)*

From [this](https://docs.getutm.app/advanced/rosetta/) website, find the command listed under the “Enabling Rosetta”   
Copy the command and run it on your virtual machine

![][/images/image9.png]

## Using GDB

Begin by copying and running the following command to enable ptrace  
**sudo sh \<\< END**  
**echo 0 \> /proc/sys/kernel/yama/ptrace\_scope**

**END**

*(You may need to run this command after reboot if gbd is causing issues)*

To run gdb, you will need to open a second MacOS terminal window and ssh into the virtual machine, allowing you to have 2 sessions on the VM

On one terminal, run the following command:

ROSETTA\_DEBUGSERVER\_PORT\=1234 ./program

Where program is the name of the file you want to debug

In the 2nd terminal, run the following:

**gdb-multiarch**

(gdb) set architecture i386:x86-64

(gdb) file ./program

(gdb) target remote localhost:1234

![][/images/image10.png]  
You can begin running the program with C in the gdb terminal, the rest of GDB should function as intended.

## Copying files to the Virtual Machine

Copying files can be done from the MacOS terminal with the following command:  
**scp ./program \<username\>@\<ip\>:\<location\>**  
Example:  
**scp ./jump pierson@192.168.64.4:/home/pierson**

## Configuring gdb for automatically setting architecture

You can have gdb automatically run commands on start via the .gdbinit file, you can create it with the command  
**nano \~/.gdbinit**  
For example, if we want it to set the architecture automatically, we could enter the following:

**set architecture i386:x86-64**

Press Control+x, y, and then enter to save and exit nano.

## Sources:

[https://stenger.io/blog/fast-gdb](https://stenger.io/blog/fast-gdb)  
[https://github.com/pwndbg/pwndbg](https://github.com/pwndbg/pwndbg)
