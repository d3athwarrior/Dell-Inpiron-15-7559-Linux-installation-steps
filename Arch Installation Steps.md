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
    1. ```mkfs.fat -F32 /dev/sda1``` - this is because that is the EFS
    2. ```mkswap /dev/sda2 && swapon /dev/sda2```
    3. ```mkfs.ext4 /dev/sda3```  
    **Note:** The sda1/sda2/sda3 etc notation might change based on the disk index. Example it could be called sdb4/sdb5 etc based on your choice of installation.
3. Setup internet connection:
    1. ```iwctl - hit enter```
    2. ```station wlan0 connect <Network_Name>``` - hit enter
4. Install the base system and the packages that will be required when arch-chroot into the new installation:
    1. ```pacstrap /mnt base base-devel linux linux-firmware vim nano sudo git efibootmgr grub iwd dialog bash-completion networkmanager man-db man-pages intel-ucode``` (or amd-ucode if running amd CPU)
5. Generate the fstab using the following command:  
    ```genfstab -U /mnt >> /mnt/etc/fstab```
6. chroot into the new system: ```arch-chroot /mnt /bin/bash```
7. Connect to internet using iwctl as seen in step 3.
8. Set locale & time and date:
    1. Check if timedatectl status shows the Universal Time correctly. If not then follow the steps below else skip to next step:
        1. ```timedatectl set-local-rtc false```
        2. ```timedatectl set-ntp true```
        3. ```timedatectl set-timezone <region>/<city>```
    2. Set the localtime using: ```ln -sf /usr/share/zoneinfo/<replace_with_region>/<replace_with_city> /etc/localtime```  
        eg. ```ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime```  
        **NOTE:** You can see the list of available time zones by entering "ln -sf /usr/share/zoneinfo/" and then hitting the tab key for list of all options.
    3. Set the time for the system using: ```hwclock --systohc --utc```
    4. Check if the date is correct using date and timedatectl status. date should show local time and timedatectl should show the Universal time, the timezone and RTC time
9. Set the hostname
    1. ```echo <your_host_name> > /etc/hostname```
    2. ```nano /etc/hosts``` and add the following entries:  
        127.0.0.0       localhost  
        ::1             localhost  
        127.0.1.1       your_host_name  
10. Install grub bootloader:
    1. ```grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi```  
    **NOTE:** Enable microcode updates - intel-ucode or amd-ucode package was added to pacstrap and the below command should generate the required code to update the microcode whenever needed
    2. grub-mkconfig -o /boot/grub/grub.cfg
11. Enable the service for networkmanager & iwd
    ```systemctl enable NetworkManager```  
    ```systemctl enable iwd```
12. Add a root password by typing ```passwd``` in the console.
13. Now we need to exit and see if the system we just installed works fine or not. For that execute the following commands:
    1. ```exit```
    2. ```umount -R /mnt```
    3. ```swapoff -a``` - if you created a swap and used it
    4. ```reboot```
14. To connect to a WiFi adapter use nmcli:
    1. ```nmcli device wifi connect <SSID> password <password>```
    2. Alternatively refer: <https://wiki.archlinux.org/index.php/NetworkManager#Enable_NetworkManager> => nmcli examples
    3. Scan the list of all the connections and then set the priorities in the config file once connected with the help of <https://jlk.fjfi.cvut.cz/arch/manpages/man/nm-settings.5> or use nmcli connections edit to edit a connection that is already created  
    **NOTE:** If there is wpa_supplicant installed by any means, remove it or remove iwd if using wpa_supplicant
15. To add a user, create a password and a home directory for it, run the following commands:
    1. ```useradd -m -g users -G wheel -s /bin/bash \<username>```
    2. ```passwd \<username>```
    3. If  you need sudo access edit the /etc/sudoers file and uncomment the below line:  
    ```%wheel ALL=(ALL) ALL```
    4. Logoff from the root user and login as the new user.  
    **NOTE:** You may want to add the user to more than just the wheel group but that can be done on a need basis later on as well.
16. Install and create default user directories:  
```sudo pacman -Sy xdg-user-dirs```
17. Enable 32-bit application support in pacman  
    1. Edit the pacman config file ```sudo nano /etc/pacman.conf```
    2. Uncomment the multilib section in the above file then save and exit.
    3. Run ```sudo pacman -Sy``` to upgrade the repository index
18. Install intel and nvidia drivers:
    1. Intel Drivers:  
        ```sudo pacman -Sy mesa lib32-mesa xf86-video-intel vulkan-intel```
    2. Nvidia drivers:  
        ```sudo pacman -Sy nvidia nvidia-utils lib32-nvidia-utils nvidia-settings```
19. Install the desktop environment:
    1. Installation of xorg which is a display server:  
        ```sudo pacman -Sy xorg-server xorg-apps```  
        **Note:** ```wayland``` can be used too but then that would mean using GNOME instead of KDE Plasma which I plan to use for this installation
    2. Installation of KDE Plasma Desktop Environment(DE):  
        1. ```sudo pacman -Sy plasma``` - install everything as it is.
        2. As a result of the previous step, sddm will be installed by default. To start to see a GUI you just need to enable the service using the following command:  
        ```sudo systemctl enable sddm.service```
        3. ```sudo reboot```  
        Post reboot you should see the new desktop and you should be able to login with the existing user.
20. Install KDE applications as per your need. You can choose from ```kde-applications``` group:  
    ```sudo pacman -Sy dolphin dolphin-plugins ffmpegthumbs filelight gwenview kcalc kcharselect kcron kdeconnect kdenetwork-filesharing kdialog kfing khelpcenter plasma-pa kolourpaint konqueror konsole ksystemlog print-manager spectacle ark ntfs-3g```
21. Install Google Noto fonts so that most of the unicode characters along with the latest emojis are rendered correctly:  
    ```pacman -Syu  noto-fonts noto-fonts-cjk noto-fonts-emoji```
22. To enable hibernation:
    1. Run the following command and note the UUID of the SWAP partition we created earlier:  
    ```lsblk -f```
    2. ```sudo nano /etc/default/grub```  
    Find the ```GRUB_CMDLINE_LINUX_DEFAULT``` and append ```resume=UUID=<UUID from above step>```
    3. Run ```sudo nano mkinitcpio.conf``` and then search for ```HOOKS```. Add ```resume``` any where after ```udev```
    4. ```sudo mkinitcpico -P``` - This will regenerate the initramfs with the new hook to resume hibernated session.
    5. ```sudo grub-mkconfig -o /boot/grub/grub.cfg``` - This will regenerate the grub config with the new parameter.
23. To enable bluetooth:
    1. ```sudo pacman -Sy bluez bluez-utils```
    2. ```sudo systemctl enable bluetooth.service```
    3. To enable bluetooth either reboot the system or run ```sudo systemctl start bluetooth.service```

## TODO

1. Setup reflector to find the closest and the fastest mirrors for installing updates
2. ~~Install all the required drivers.~~
3. ~~Add a user with sudo access to the system~~
4. Install drivers for switching between intel and nvidia graphics aka either prime or optimus
5. Add configuration to maybe decide how the CPU throttles(?)
6. ~~Add a GUI~~
7. ~~Post all the above configurations, install Windows alongside arch without breaking the arch installation~~
8. ~~Need to install a terminal app and see what application I need from the kde-application group~~

## Issues

1. ~~The splash screen doesn't show~~(Because screw splash screen, it boots under 20s from SSD and splash screen doesn't even make any sense)
2. When the system is being shut down, it throws some error, need to investigate
3. ~~Hibernation is broken~~

## References

1. <https://gist.github.com/seteBR/34e3c1461cd715aaab7ee93dfc6f8316>
2. <https://itsfoss.com/install-arch-linux/>
3. <https://averagelinuxuser.com/a-step-by-step-arch-linux-installation-guide/#install-the-system>
4. [Arch-Wiki](https://wiki.archlinux.org/)
5. <https://averagelinuxuser.com/plasma-5-on-arch-linux-install-and-configure/>
6. <https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate#Hibernation> - To get Hibernation working
7. <https://wiki.archlinux.org/index.php/Mkinitcpio#Manual_generation> - Manual Regeneration of initramf
