+++
title =  "Your Guide to Arch Linux"
date =  2021-03-17T00:22:09-04:00
menu = "main"
+++

![](../../static/img/arch.png)

Hi everyone! This guide is meant to take you through the process of setting up Arch Linux. It's more of a set of guidelines rather than strict procedure. I'm hoping to give new or potential users an idea of what it's like to dual boot your computer with Arch Linux. 

I started using Arch in 2018 after getting tired of Ubuntu and since then I have dealt with my fair share of problems (corrupted ISO, problematic updates, bluetooth nightmares). However, I can say with confidence that using Arch has given me a better understanding and appreciation for operating systems as a whole. There are some things I used to take for granted - like the wifi menu that comes pre-installed with Windows or Mac OS - that I had to install and configure when it came to Arch Linux.

I decided to create this guide for myself when my first installation went south and I lost everything because I didn't have my dotfiles backed up. So the second time around I created [this repo](https://github.com/gucci-ninja/.dotfiles.git)

# Step 0 - Preparation
If you are dual-booting you are essentially telling your current operating system to stop hogging all the space on your computer and make room for a new OS. So figure out what disk management tool your current OS uses and make a `partition` for your new OS. I went with 100GB, which should be fairly sufficient unless you know you're going to be storing a lot of stuff.
# Step 1 - Securing the ISO
When you decide to make the switch to Arch you are going to need an image of the operating system. This comes in the form of a `.iso` file. You are going to have to flash this image file onto a good USB (or a CD if that's your jam and you still have a CD player..). If you are on Windows you can use a software called `Rufus` which lets you select your USB and flash an ISO image onto it. 

# Step 2 - Booting the ISO
Once you have a fresh image of Arch on your ISO, you want to plug the USB or whatever you're using into your computer of choice. Then, depending on what computer you're working with, you need to press some key(s) at startup so you're transported to the boot menu. This will prevent your default OS from loading up. For me, I spammed F2/F3 repeatedly until I saw the boot menu. Then, I selected the option to boot into the USB. Remember, if you mess up, it's not the end of the world, just possibly the end of your PC :D JK
# Step 3 - Mount your system
This is actually a bunch of steps but I *usually* do them all at once. It's kind of intimidating if it's your first time because you just see a black screen with a weird font of white text. 

```
# To see partitions (like the one you made in Step 0)
fdisk -l
```
My EFI disk is sda1 and the Arch one is sda5. I know this because sda5 says exactly 100GB, which is how much I allocated to it.

```
# To format sda5
mkfs ext4 -L "Arch Linux" /dev/sda5

# To mount the partition
mount /dev/sda5 /mnt

# Get wifi
wifi-menu

# Install base system
pacstrap /mnt base base-devel vim

# Mount EFI partition
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

# Generate the fstab (I have no clue what this is)
genfstab -p /mnt >> /mnt/etc/fstab
```

# Step 4 - Get out of the USB
At this point you have sucessfully taken the image on your USB and mounted it onto your computer. This step is basically the equivalent of `cd` into your new directory. Still keep the USB plugged in though, we're not done with it yet.
```
# Chroot into arch linux
arch-chroot /mnt
```

# Step 5 - Wifi
Most important step because without wifi you can't browse memes.

```
pacman -S networkmanager

# Automatically start it up
systemctl enable NetworkManager
```

# Step 6 - Bootloader
You need something to load your OS each time you restart your computer so get grub.

```
# Download grub
pacman -S grub efibootmgr

# Install grub into HDD
grub-install /dev/sda
ls -l /boot/efi/EFI/arch

# Generate grub config
grub -mkconfig -o /boot/grub/grub.cfg
```

# Step 7 - Set Up Some Boring Things
Like date, timezone, clock, etc

```
timedatectl set-ntp true

# Set up timezone
ln -sf /usr/share/zoneinfo/Canada/Eastern /etc/localtime

# Set clock
hwclock --systohc

# Also set locale
# First uncomment the Canada locale in /etc/locale.gen
vi /etc/locale.gen
# Generate the locales
locale-gen

# Do this step
mkinitcpio -p linux

# Set the root password
passwd
```

# Step 9 - Unmount

```
umount -R /mnt
reboot
```

Now you can take out the USB.

# Step 10 - Take a break because you've come this far
:sunglasses:

# Step 11 - Users
After rebooting, login as root and create a user.

```
useradd -m -g wheel suhavi
psswd suhavi

# Give sudo ability
vim /etc/sudoers
# Uncomment the following line
%wheel ALl=(ALL) ALL
```

# Step 12 - Xorg
This is a graphical server. So you don't have to look at a blank sceen all the time.
```
pacman -S xorg-server xorg-init
# Type startx to start it
```

# Step 13 - Terminal + Shell + Git + VS Code
At this point the steps are very general. It's important to do your research to see which terminal, shell and editor you want. These are the ones I decided to go with
- terminal: termite
- shell: fish
- editor: VS code
- version control: git

```
pacman -S termite fish git code

# Make fish your default
chsh -l # See all shells
chsh -s fish

# Copy the following to fish config file to make startx run on startup
if status is-login
  if test -z "$DISPLAY" -a $XDG_VTNR = 1
    exec startx -- -keeptty
  end
end

```
Configs are your best friend with Linux. Located under .config at the root level, they contain rules for all the customizations you want to make to your operating system. Take a look at the files in this folder to get an idea of the kinds of changes you can make.
# Step 14 - Window Manager
There are lots of choices for window manager, I personally went with bspwm because I like binary trees. I also got dmenu, which is a search bar that lets you type commands or applications to open. It can be accessed with (cmd/windows key) + spacebar. However, in order for these hotkeys and shortcuts to work you need sxhkd. Lastly, I installed some fonts!

```
pacman -S bspwm dmenu sxhkd

# Fonts
pacman -S noto-fonts
# Fonts can be managed at ~/.config/fontconfig/fonts.conf

# Make x server start bspwm
vim ~/xinitrc
# Add the followng
sxhkd &
exec bspwm

# Make termite your default terminal by going into the sxhkdrc file and changing xterm to termite

# Add custom keybinds to sxhkdrc
```

# Step 15 - Sound
I didn't have sound in the beginning.

```
pacman -S alsa-utils
alsamixer -c 0
```

Use the arrow keys and stuff to toggle all sound things on

# Step 16 - Wallpaper
- find a good wallpaper

```
feh --bg-scale wallpaper.jpg

# Edit the following to xinitrc
~/.fehbg &

# Make sure it's before exec bspwm^
```

# Step 17 - Yay
- this will help download AUR things

```
git clone https://aur.archlinux.org/yay.git

cd yay

makepkg -si
```
# Step 18 - Polybar
I actually hate this part

```
yay -S polybar
install -Dm644 /usr/share/doc/polybar/config $HOME/.config/polybar/config
example polybar

# Make a lot of changes to example config

# I was getting unicode errors so
sudo pacman -S ttf-font-awesome

# In polybar config change fonts
font-1 = "Font Awesome 5 Free:style=Regular:size=10;5"
font-2 = "Font Awesome 5 Free:style=Solid:size=10;5"

# Go through all unicode characters in polybar config and replace them with icons from the FA cheatsheet

# Put this line in bspwmrc
polybar main &

```

# Step 19 - Backing Up Your Dotfiles
I'm not trying to lose all my stuff again.

```
# Initialize a git bare repo
git init --bare $HOME/.dotfiles

# Create an alias
alias dotfiles ="git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME"

# Save alias
funcsave dotfiles

# Ignore files you don't want tracked
dotfiles config --local status.showUntrackedFiles no

# Create a github repo called .dotfiles

# dotfiles remote add origin https://github.com/gucci-ninja/.dotfiles
```

# Step 20 - Make your windows look less ugly
Since I have bspwm I don't have a display manager to add GTK themes to. So I have to get lxappearance >:( I feel lied to

```
yay -S lxappearance
sudo pacman -S gtk-engine-murrine adwaita-icon-theme

```

Step 20 was made possible largely due to this reddit post, https://www.reddit.com/r/unixporn/comments/74z2z6/easily_getting_started_with_bspwm_and_polybar/
and I wish I had found it earlier.

# Step 21 - Neofetch
You don't need to do this, it's mostly fo showing off to people. 

```
pacman -S neofetch

# Add the following to fish.config
neofetch

```

Now eveyr time you create a new terminal session you can show off.

# Step 22 - File Manager
- I'm using thunar

# Step ?? - Aliases

# Step 23 - Connecting to Eduroam
- this is too hard and I failed
- luckily it was easy to connect to Mac-WiFi (note the capitals)
- I'll post my wifi config if someone asks

# Step 24 - Bluetooth
```bash
$ pacman -S bluez bluez-utils
$ pacman -S pulseaudio-bluetooth pulseaudio-alsa pavucontrol

systemctl enable bluetooth.service

$ sudo vim etc/bluetooth/main.conf
# add the following line at the end
# this will ensure your bluetooth headphones auto-connect
AutoEnable=true

# make a directory for pulse (you may already have it)
$ mkdir ~/.config/pulse
# copy sample
$ cp /etc/pulse/* ~/.config/pulse/

$ systemctl restart bluetooth

$ bluetoothctl
# you should be in the bluetooth user now
[bluetooth]~ power on
[bluetooth]~ agent on
[bluetooth]~ default-agent
[bluetooth]~ scan on
# at this point you should put your headphones in pairing mode
# they should show up with a mac address
[bluetooth]~ pair 00:00:00:00:00
[bluetooth]~ connect 00:00:00:00:00
# if this part fails run the following in a new terminal
$ pulseaudio --start
# it should connect now
[bluetooth]~ trust 00:00:00:00:00
[bluetooth]~ scan off
[bluetooth]~ exit

# it should connect your headphoens at this point. if you don't hear sound, restart spotify or whatever and it should work
# if it doesn't, open pavucontrol
$ pavucontrol

# this interface will show you your connected devices, make sure it is not on mute and that under configuration tab it says it's connect to ADP SINK or sm dumb
```

## Step ??
- downloaded nerd-fonts-iosevka
- tty-clock

## polybar improements
- rofi
- https://awesomeopensource.com/project/adi1090x/polybar-themes
- pip3 install pywal


# Help

  I updated my system and something broke.
  
  ```
  $ uname -a
    This is the linux version you're running
  $ cd /var/cache/pacman/pkg
  $ ls
    These are the linux versions you have saved in your cache. Find one you want to downgrade to
  $ sudo pacman -U linux-#.#.#.arch---
  $ reboot
  ```
