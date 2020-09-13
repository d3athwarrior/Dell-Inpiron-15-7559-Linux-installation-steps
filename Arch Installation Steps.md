# Notes for installing Arch Linux on Dell 7559

## System Configuration

ToDo

## NOTE

1. No separate partition for /root and /home it will be in the same space
2. These steps are for a clean installation on a SSD

## Steps to install Arch

1. Use cfdisk /dev/sda to create the following partitions(Always make GPT partition):

    |Description                            |Size           |Type       |
    |---                                    |---            |---        |
    |The EFI System Partition(EFS)          | Base to 512MB | EFI File System  |
    | A swap partition                      | 16G           | swap      |
    | A partition which will have the home and root and all other arch related data                            | 250G          | Linux File System|
    The type can be set using the type option from the menu on the cfdisk command.
2. Formatting the partitions to prepare them for use:
    1. mkfs.fat -F32 /dev/sda1 - this is because that is the EFS
    2. mkswap /dev/sda2 && swapon /dev/sda2
    3. mkfs.ext4 /dev/sda3
3. Setup internet connection:
    1. iwctl - hit enter
    2. station wlan0 connect "Network_Name" - hit enter
4. Install the base system and the packages that will be required when arch-chroot into the new installation:
    1. pacstrap /mnt base base-devel linux linux-firmware vim nano sudo git efibootmgr grub iwd dialog bash-completion networkmanager man-db man-pages intel-ucode (or amd-ucode if running amd CPU)
5. chroot into the new system: arch-chroot /mnt /bin/bash
6. Connect to internet using iwctl as seen in step 3.
7. Set locale & time and date:
    1. Check if timedatectl status shows the Universal Time correctly. If not then follow the steps below else skip to next step:
        1. timedatectl set-local-rtc false
        2. timedatectl set-ntp true
        3. timedatectl set-timezone <region>/<city>
    2. Set the localtime using: ln -sf /usr/share/zoneinfo/<replace_with_region>/<replace_with_city> /etc/localtime  
        eg. ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime  
        **NOTE:** You can see the list of available time zones by entering "ln -sf /usr/share/zoneinfo/" and then hitting the tab key for list of all options.
    3. Set the time for the system using: hwclock --systohc --utc
    4. Check if the date is correct using date and timedatectl status. date should show local time and timedatectl should show the Universal time, the timezone and RTC time
8. Set the hostname
    1. echo your_host_name > /etc/hostname
    2. nano /etc/hosts add the following entries:  
        127.0.0.0       localhost  
        ::1             localhost  
        127.0.1.1       your_host_name  
9. Install grub bootloader:
    1. grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi  
    **NOTE:** Enable microcode updates - intel-ucode or amd-ucode package was added to pacstrap and the below command should generate the required code to update the microcode whenever needed
    2. grub-mkconfig -o /boot/grub/grub.cfg
10. Enable the service for networkmanager & iwd
    systemctl enable NetworkManager
    systemctl enable iwd
11. Add a root password by typing _**passwd**_ in the console.
12. Now we need to exit and see if the system we just installed works fine or not. For that execute the following commands:
    1. exit
    2. umount -R /mnt
    3. swapoff -a if you created a swap and used it
    4. reboot
13. To connect to a WiFi adapter use nmcli:
    1. nmcli device wifi connect \<SSID> password \<password>
    2. Alternatively refer: <https://wiki.archlinux.org/index.php/NetworkManager#Enable_NetworkManager> => nmcli examples
    3. Scan the list of all the connections and then set the priorities in the config file once connected with the help of <https://jlk.fjfi.cvut.cz/arch/manpages/man/nm-settings.5> or use nmcli connections edit to edit a connection that is already created  
    **NOTE:** If there is wpa_supplicant installed by any means, remove it or remove iwd if using wpa_supplicant

## TODO

1. Setup reflector to find the closest and the fastest mirrors for installing updates
2. Install all the required drivers.
3. Add a user with sudo access to the system
4. Install drivers for switching between intel and nvidia graphics
5. Add configuration to maybe decide how the CPU throttles(?)
6. Add a GUI

Post all the above configurations, install Windows alongside arch without breaking the arch installation - Done already.

## References
1. <https://gist.github.com/seteBR/34e3c1461cd715aaab7ee93dfc6f8316>
2. <https://itsfoss.com/install-arch-linux/>
3. <https://averagelinuxuser.com/a-step-by-step-arch-linux-installation-guide/#install-the-system>
4. [Arch-Wiki](https://wiki.archlinux.org/)
5. https://www.google.com/search?q=arch+installation+steps&rlz=1C1CHBD_en-GBUS904US904&oq=arch+installation+steps&aqs=chrome..69i57.13339j0j1&sourceid=chrome&ie=UTF-8 (Yes, I feel I might not remember the search term I used to get a few of the above results)
