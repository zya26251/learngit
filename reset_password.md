#How to change a forgotten or lost root password###
###0 Issue
- The root password was forgotten and the system cannot be logged into
- How to reset a root password
- Unable to gain root access to a system
- The root password changed

#Environment
-  Red Hat Enterprise Linux (All version)

###,Resolution

#Red Hat Enterprise Linux 4, 5, 6
You can change the root password from either single user mode or rescue mode. The method for booting into single user mode depends on your bootloader:

###GRUB - No password protection

Booting into single user mode using GRUB is accomplished by editing the kernel line of the boot configuration. This assumes that either the GRUB boot menu is not password protected or that you have access to the password if it is.

When the system boots up, you will see the GRUB countdown, which is set to 5 seconds by default . Press "Esc" to intercept this countdown and go enter a GRUB menu. Then follow these steps:

- Press 'e' to start editing.
- Scroll down to the "kernel..." line. This line tells GRUB which kernel to boot.
- Press 'e' again to edit this line.
- Move to the end of the line. Add the number "1" to the end after space.
- Once you have finished that change, press Enter to accept the edit.
- Press 'b' to boot using that kernel and boot into runlevel 1 (single user mode).

Change the root password when the "#" prompt appears by using the "passwd" command.

**Note:**The switch to runlevel 1 is not persistent. At next boot, the system will start in default runlevel as specified in the /etc/inittab file.

###Rescue Mode (GRUB is protected, system is unbootable due to a Maintenance mode prompt, or other issues)


If the GRUB boot menu is password protected or the system is unbootable due to other issues and you do not have access to the password, you will need to use a rescue disk to boot the system.

Follow the instructions given by the rescue disk boot process:
	
- Boot the system from boot disc 1. Once the system has successfully booted from the ISO image and the Red Hat Enterprise Linux boot screen appears,  type "linux rescue" without the quotes at the boot prompt and press the enter key.

		[F1-Main] [F2-Options] [F3-General] [F4-Kernel] [F5-Rescue]
		boot: linux rescue
- When prompted for language and keyboard, provide the pertinent information for the system. When prompted to enable the network devices on the system, select "No"
- Select "Continue" when prompted to allow the rescue environment to mount the Red Hat Enterprise Linux installation under the /mnt/sysimage directory. 
- Run the command "chroot /mnt/sysimage" to change root to your system image.
- Use the command "passwd" to change the root password of the system.
- *If the command "passwd" is not found, you will need to mount /usr in order to access usr/bin/passwd*

###LILO
When the system comes to the LILO prompt, type "linux single". When the "#" prompt appears you will need to type "passwd root". This will update the password to a newer one. At this point you can type "exit" and your system should return to the boot sequence. Alternatively, you can reboot your system with the "shutdown -r now" or "reboot" commands. The system should boot up normally. You can now use your new root password to gain root access.

If LILO is configured to not wait at the boot menu (the timeout value in /etc/lilo.conf is set to zero) you can still halt the boot process by pressing any key in the split second before LILO boots the kernel.

#Red Hat Enterprise Linux 7

###0 Issue
- How to reset the root password of a RHEL-7 / systemd ?
- RHEL-7 ask for a root password even after booting in Single user mode.
- Not able to change the root password in single user mode

###Environment
- Red Hat Enterprise Linux 7.

###,Resolution
1) Boot your system and wait until the GRUB2 menu appears. <br>
2) In the boot loader menu, highlight any entry and press e.<br>
3) Find the line beginning with linux. At the end of this line, append the following:

	init=/bin/sh
Or if you face a panic, instead of "ro"change to "rw" to sysroot as example bellow:

	rw init=/sysroot/bin/sh
4) Press F10 or Ctrl+X to boot the system using the options you just edited.<br>
Once the system boots, you will be presented with a shell prompt without having to enter any user name or password:

	sh-4.2#
5) Load the installed SELinux policy:

	sh-4.2# /usr/sbin/load_policy -i
6) Execute the following command to remount your root partition:

	sh4.2# mount -o remount,rw /
7) Reset the root password:

	sh4.2# passwd root
When prompted to, enter your new root password and confirm by pressing the Enter key. Enter the password for the second time to make sure you typed it correctly and confirm with Enter again. If both passwords match, a message informing you of a successful root password change will appear.<br>
8) Remount the root partition again, this time as read-only:

	sh4.2# mount -o remount,ro /
9) Reboot the system. From now on, you will be able to log in as the root user using the new password set up during this procedure.

**Please note that in case you are using a USB keyboard or if the system is a virtual guest, the following instructions need to be followed**

Note that the above mentioned steps may drop you to a prompt without access to a USB keyboard and do not work in a VM like KVM or VirtualBox. To reset the root password in these environments:

1) add rd.break instead of init=/bin/sh to the end of the line that starts with linux in Grub2:<br>
2) when the system boots, run the following command to remount the root filesystem in read-write mode:

	mount -o remount,rw /sysroot
3) then run:
 
	chroot /sysroot
4) run:

	passwd
5) instruct SELinux to relabel all files upon reboot (because the /etc/shadow file was changed outside of its regular SELinux context) -- run:

	touch /.autorelabel
Note that this may take some time during the next boot.

6) type exit to leave the chroot environment.<br>
7) type exit to log out.

The system will reboot, re-apply all SELinux labels, and present you with a regular login prompt.

**Note:** If the system is encrypted, the above method will not work. Please refer to the following article: [Resetting the Root Password of RHEL-7 for encrypted devices](https://access.redhat.com/solutions/1192343)

###References:

[Red Hat Enterprise Linux 7 Installation Guide- Basic System Recovery](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-basic-system-recovery.html#sect-rescue-mode-reset-root-password)


#Access Single User Mode (Reset Root Password)
To reset the root password of your server, you will need to boot into single user mode.

Access the Manage section of your server in the customer portal and follow these steps. The option depends on the bootloader version on the machine:

#CentOS 6
1. Click [View Console] to access the console and click the send CTRL+ALT+DEL button on the top right. Alternatively, you can also click [RESTART] to restart the server. 
2. You will see a GRUB boot prompt telling you to press any key - you have only a few seconds to press a key to stop the automated booting process. (If you miss this prompt you will need to restart the VM again) 
3. At the GRUB prompt, type "a" to append to the boot command. 
4. Add the text "single" and press enter. 
5. System will boot and you will see the root prompt. Type "passwd" to change the root-password and then reboot again

#Debian, Ubuntu, CentOS 7
1. Click [View Console] to access the console and click the send CTRL+ALT+DEL button on the top right. Alternatively, you can also click [RESTART] to restart the server. 
2. As soon as the boot process starts, press ESC to bring up the GRUB boot prompt. You may need to turn the system off from the control panel and then back on to reach the GRUB boot prompt.
3. You will see a GRUB boot prompt - press "e" to edit the first boot option. (If you do not see the GRUB prompt, you may need to press any key to bring it up before the machine boots) 
4. Find the kernel line (starts with "linux /boot/") and add init="/bin/bash" at the end of the line 
5. Press CTRL-X or F10 to boot. 
6. System will boot and you will see the root prompt. Type "mount -rw -o remount /" and then "passwd" to change the root password and then reboot again. 

#FreeBSD
The boot menu has an option to boot into single-user mode. Press the key for single user mode (2). At the root prompt, type "passwd" to change the root password and then reboot again.

#CoreOS
CoreOS by default uses SSH key authentication. On Vultr, a root user and password are created. If an SSH key is selected when creating the VPS, this SSH key can be used to login as user "core".

It is possible to reset the standard root login by executing "sudo passwd" as user "core". Login as "core" using the SSH key first.

If you lost your SSH key, then you can login as the "core" user by editing the grub loader. Follow these steps:

1. Click [View Console] to access the console and click the send CTRL+ALT+DEL button on the top right. Alternatively, you can also click [RESTART] to restart the server. 
2. You will see a GRUB boot prompt - press "e" to edit the first boot option. (If you do not see the GRUB prompt, you may need to press any key to bring it up before the machine boots) 
3. At the end of the line that begins with "linux$" add " coreos.autologin=tty1" (no quotes). 
4. Press CTRL-X or F10 to boot. You will be logged in as "core" when the system boots. 
5. Remember to reboot your server after you have reset your login. 

#Windows Server 2012
setting the administrator password on Windows Server 2012 is not possible on Vultr. If you have recently installed your VPS, we recommend reinstalling the OS.