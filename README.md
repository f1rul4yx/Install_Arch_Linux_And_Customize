# INSTALL ARCH LINUX AND CUSTOMIZE

## INSTALL ARCH LINUX

### Load Spanish Keyboard Layout

- loadkeys es

### Connect to Wi-Fi

- iwctl
- device list
- station {NetworkCard} scan
- station {NetworkCard} get-networks
- station {NetworkCard} connect {NetworkName}

### Check System Time

- timedatectl status

### Partition the Disk

- fdisk /dev/{Disk}

```
{Partition1}   40G     /
{Partition2}   428,6G  /home
{Partition3}   300M    /boot
{Partition4}   8G      [SWAP]
```

### Format Partitions

- mkfs.ext4 /dev/{Partition1}
- mkfs.ext4 /dev/{Partition2}
- mkfs.fat -F 32 /dev/{Partition3}
- mkswap /dev/{Partition4}

### Mount Partitions

- mount /dev/{Partition1} /mnt
- mkdir /mnt/home
- mkdir /mnt/boot
- mount /dev/{Partition2} /mnt/home
- mount /dev/{Partition3} /mnt/boot
- swapon /dev/{Partition4}

### Install Base System

- pacstrap -K /mnt base linux linux-firmware

### Generate fstab

- genfstab -U /mnt >> /mnt/etc/fstab

### Chroot into the New System

- arch-chroot /mnt

### Install Essential Packages

- pacman -S networkmanager grub efibootmgr nano sudo

### Set Timezone and Clock

- ln -sf /usr/share/zoneinfo/{Continent}/{City} /etc/localtime
- hwclock --systohc

### Configure Locale

- Change to `es_ES.UTF-8` in /etc/locale.gen
- locale-gen
- Set `LANG=es_ES.UTF-8` in /etc/locale.conf
- Set `KEYMAP=es` in /etc/vconsole.conf

### Set Hostname

- Set `DeviceName` in /etc/hostname

### Generate initramfs

- mkinitcpio -P

### Set Root Password

- passwd

### Create a New User

- useradd -m -s /bin/bash {Username}
- passwd {Username}

### Sudoers File Configuration

- Set `{Username} ALL=(ALL:ALL) ALL` in /etc/sudoers

### Install and Configure GRUB

- grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
- grub-mkconfig -o /boot/grub/grub.cfg

## INSTALL GRAPHICAL ENVIRONMENT

### Install Packages

- su {Username}
- cd
- sudo pacman -S git p7zip zsh zsh-syntax-highlighting kitty bspwm sxhkd polybar picom dmenu bat lsd lightdm lightdm-webkit2-greeter

### Setting Lightdm

- sudo systemctl enable lightdm
- Change to `greeter-session=lightdm-webkit2-greeter` in /etc/lightdm/lightdm.conf
- Change to `user-session=bspwm` in /etc/lightdm/lightdm.conf

### Setting BSPWM

- mkdir -p ~/.config/bspwm && cp /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm/
- Change to `bspc monitor -d 1 2 3 4 5 6 7 8 9 10` in ~/.config/bspwm/bspwmrc
- Change to `bspc config border_width 0` in ~/.config/bspwm/bspwmrc
- Set

```
setxkbmap -layout es
$HOME/.config/polybar/launch.sh
picom &
feh --bg-scale $HOME/.config/wallpapers/wallpaper.png
xrandr -s 1920x1080 -r 60
```

in ~/.config/bspwm/bspwmrc

### Setting Sxhkd

- mkdir ~/.config/sxhkd && cp /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd/
- Change to `/usr/bin/kitty` in ~/.config/sxhkd/sxhkdrc
- Change to `super + shift + m` in ~/.config/sxhkd/sxhkdrc
- Change to `/usr/bin/dmenu_run -b -i -nb "#222" -nf "#fff"` in ~/.config/sxhkd/sxhkdrc
- Change to `Left,Down,Up,Right` in ~/.config/sxhkd/sxhkdrc
- Set

```
#
# programs
#

# firefox
super + shift + f
    /usr/bin/firefox

# chromium
super + shift + g
    /usr/bin/chromium

# i3lock
super + shift + x
    /usr/bin/i3lock

# scrot
super + shift + s
    /usr/bin/scrot -s '/tmp/screenshot_%Y-%m-%d_%H-%M-%S.png' -e 'mv $f /tmp/screenshot.png' && xclip -selection clipboard -t image/png -i /tmp/screenshot.png

# pactl +
super + shift + p
    pactl set-sink-volume @DEFAULT_SINK@ +1%

# pactl -
super + shift + o
    pactl set-sink-volume @DEFAULT_SINK@ -1%
```

in ~/.config/sxhkd/sxhkdrc

### Setting Kitty

- mkdir ~/.config/kitty && cp /usr/share/doc/kitty/kitty.conf ~/.config/kitty/
- Change to `background #222222` in ~/.config/kitty/kitty.conf
- Change to `background_opacity 0.85` in ~/.config/kitty/kitty.conf
- Change to `window_padding_width 20` in ~/.config/kitty/kitty.conf
- Change to `cursor_shape beam` in ~/.config/kitty/kitty.conf

### Exit Chroot and Reboot

- exit
- exit
- umount -R /mnt
- reboot

### Enable NetworkManager

- sudo systemctl enable NetworkManager
- sudo systemctl start NetworkManager
- nmcli radio wifi on
- nmcli dev wifi list
- nmcli dev wifi connect {SSID} password {password} ifname {NetworkCard}

### Setting ZSH

- sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
- Set `source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh` in ~/.zshrc
- Change to `ZSH_THEME="powerlevel10k/powerlevel10k"` in ~/.zshrc
- Set

```
# Aliases
alias ls="lsd"
alias cat="bat"
alias target="$HOME/.config/scripts/target/target.sh"
alias untarget="$HOME/.config/scripts/target/untarget.sh"
```

in ~/.zshrc

### Setting Powerlevel10k

- git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
- https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/Hack.zip
- sudo 7z x Hack.zip -o/usr/local/share/fonts
- sudo chown $USER:$USER /usr/local/share/fonts

### Setting Polybar

- mkdir ~/.config/polybar && cp /usr/share/doc/polybar/examples/config.ini ~/.config/polybar/
- touch ~/.config/polybar/launch.sh && chmod +x ~/.config/polybar/launch.sh
- Set

```
#!/usr/bin/env bash

# Terminate already running bar instances
# If all your bars have ipc enabled, you can use
polybar-msg cmd quit
# Otherwise you can use the nuclear option:
# killall -q polybar

# Launch bar1 and bar2
echo "---" | tee -a /tmp/polybar1.log /tmp/polybar2.log
polybar example 2>&1 | tee -a /tmp/polybar1.log & disown
polybar example 2>&1 | tee -a /tmp/polybar2.log & disown

echo "Bars launched..."
```

in ~/.config/polybar/launch.sh

- Set

```
[module/battery]
type = internal/battery
full-at = 99
low-at = 5
battery = BAT1
adapter = ACAD
poll-interval = 5

[module/target_ip]
type = custom/script
exec = echo "%{F#F0C674}Target%{F-} $(cat $HOME/.config/scripts/target/target_ip 2>/dev/null || echo '~')"
interval = 1
signal = 10
```

in ~/.config/polybar/config.ini

- Change to

```
modules-left = xworkspaces
modules-right = filesystem pulseaudio xkeyboard memory cpu battery wlan eth target_ip date
```

in ~/.config/polybar/config.ini

- Change to

```
[module/wlan]
inherit = network-base
interface-type = wireless
interface = {NetworkCard}
label-connected = %{F#F0C674}%ifname%%{F-} %local_ip%
```

in ~/.config/polybar/config.ini

### Setting Picom

- mkdir ~/.config/picom && cp /usr/share/doc/picom/picom.conf.example ~/.config/picom/picom.conf
- Change to `vsync = false;` in ~/.config/picom/picom.conf
- Change to `shadow = false;` in ~/.config/picom/picom.conf
- Change to `corner-radius = 15` in ~/.config/picom/picom.conf

### Setting Feh

- mkdir ~/.config/wallpapers

### Setting Target

- mkdir -p ~/.config/scripts/target && touch ~/.config/scripts/target/target.sh && touch ~/.config/scripts/target/untarget.sh && chmod +x ~/.config/scripts/target/\*
- Set

```
#!/bin/zsh

if [ -z "$1" ]; then
  echo "Uso: target {ip}"
  return 1
fi

if [[ ! "$1" =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
  echo "Error: La IP debe ser una dirección IPv4 válida."
  return 1
fi

echo "$1" > $HOME/.config/scripts/target/target_ip
```

in ~/.config/scripts/target/target.sh

- Set

```
#!/bin/zsh

echo "~" > $HOME/.config/scripts/target/target_ip
```

in ~/.config/scripts/target/untarget.sh

### Setting Grub

- https://www.gnome-look.org/p/1009236
- 7z x Vimix-1080p.tar.xz && 7z x Vimix-1080p.tar && sudo mv Vimix-1080p/Vimix /boot/grub/themes/
- Change to `GRUB_THEME="/boot/grub/themes/Vimix/theme.txt"` in /etc/default/grub
- Change to `GRUB_GFXMODE=1920x1080` in /etc/default/grub
- sudo grub-mkconfig -o /boot/grub/grub.cfg

## INSTALL THE GRAPHIC ENVIRONMENT WITH SUDO

### Setting ZSH

- sudo su
- cd
- sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
- Set `source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh` in ~/.zshrc
- Change to `ZSH_THEME="powerlevel10k/powerlevel10k"` in ~/.zshrc
- Set

```
# Aliases
alias ls="lsd"
alias cat="bat"
```

in ~/.zshrc

### Setting Powerlevel10k

- git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

## OTHER SOFTWARE

- base-devel - Development tools and libraries
- chromium - Web browsing (RECOMMENDED)
- code - Source code editing
- feh - Image viewer and background setter (RECOMMENDED)
- firefox - Web browsing (RECOMMENDED)
- gimp - Image and graphics editing
- i3lock - Screen locking (RECOMMENDED)
- keepass - Password management
- kdenlive - Video editing
- neofetch - Displaying system information
- pavucontrol - Audio control
- pulseaudio - Sound server (RECOMMENDED)
- qbittorrent - Torrent file downloading
- scrot - Screenshot tool (RECOMMENDED)
- thunar - File management
- timeshift - System backup and restore
- vlc - Media file playback
- xclip - Clipboard manager (RECOMMENDED)
- xorg-xrandr - Display configuration (RECOMMENDED)
