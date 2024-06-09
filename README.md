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
- Change to `bspc monitor -d          ` in ~/.config/bspwm/bspwmrc
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
- Change to `/usr/bin/dmenu_run -b -i -nb "#222222" -nf "#ffffff" -sb "#9b59b6" -sf "#ffffff"` in ~/.config/sxhkd/sxhkdrc
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

- mkdir ~/.config/polybar && touch ~/.config/polybar/config.ini
- touch ~/.config/polybar/launch.sh && chmod +x ~/.config/polybar/launch.sh
- mkdir -p ~/.config/scripts/target && touch ~/.config/scripts/target/target.sh && touch ~/.config/scripts/target/untarget.sh && chmod +x ~/.config/scripts/target/\*
- mkdir ~/.config/scripts/connection && touch ~/.config/scripts/connection/wlan.sh && touch ~/.config/scripts/connection/eth.sh && touch ~/.config/scripts/connection/htb.sh && chmod +x ~/.config/scripts/connection/\*
- mkdir ~/.config/scripts/battery && touch ~/.config/scripts/battery/battery.sh && chmod +x ~/.config/scripts/battery/\*
- Set

```
#!/bin/bash

polybar-msg cmd quit
echo "---" | tee -a /tmp/polybar1.log
polybar bar1 2>&1 | tee -a /tmp/polybar1.log & disown
echo "Bars launched..."
```

in ~/.config/polybar/launch.sh

- Set

```
[colors]
blue = #3498DB
red = #E74C3C
grey = #7F8C8D
green = #2ECC71
purple = #9B59B6
yellow = #F39C12
background = #00000000

[bar/bar1]
width = 98%
height = 5%
offset-x = 1%
module-margin-left = 2
module-margin-right = 2
background = ${colors.background}
font-0 = Hack Nerd Font
modules-left = wlan eth htb target
modules-right = battery pulseaudio date
modules-center = xworkspaces

[module/xworkspaces]
type = internal/xworkspaces
label-active = %name%
label-active-foreground = ${colors.purple}
label-active-padding = 1
label-occupied = %name%
label-occupied-padding = 1
label-urgent = %name%
label-urgent-background = ${colors.red}
label-urgent-padding = 1
label-empty = %name%
label-empty-foreground = ${colors.grey}
label-empty-padding = 1

[module/wlan]
type = custom/script
exec = $HOME/.config/scripts/connection/wlan.sh
interval = 2

[module/eth]
type = custom/script
exec = $HOME/.config/scripts/connection/eth.sh
interval = 2

[module/htb]
type = custom/script
exec = $HOME/.config/scripts/connection/htb.sh
interval = 2

[module/target]
type = custom/script
exec = echo "%{F#E74C3C}󰓾 %{F-}$(cat $HOME/.config/scripts/target/target.txt)"
interval = 2

[module/pulseaudio]
type = internal/pulseaudio
format-volume-prefix = "%{F#9B59B6}  %{F#FFFFFF}"
label-muted = "%{F#E74C3C} %{F#FFFFFF}"

[module/battery]
type = custom/script
exec = $HOME/.config/scripts/battery/battery.sh
interval = 2

[module/date]
type = internal/date
date = %{F#9B59B6}󰸗 %{F#FFFFFF}%b%e  %{F#9B59B6}󱑎 %{F#FFFFFF}%k:%M
interval = 1
```

in ~/.config/polybar/config.ini

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
formatted_ip="%{F#E74C3C}$1%{F#FFFFFF}"
echo "$formatted_ip" > $HOME/.config/scripts/target/target.txt
```

in ~/.config/scripts/target/target.sh

- Set

```
#!/bin/zsh

echo "%{F#BDC3C7}No target%{F#FFFFFF}" > $HOME/.config/scripts/target/target.txt
```

in ~/.config/scripts/target/untarget.sh

- Set

```
#!/bin/zsh

ip_address=$(/usr/bin/ip addr show {WirelessCard} | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)
if [ -z "$ip_address" ]; then
    echo "%{F#E74C3C}󰖪 %{F#FFFFFF}%{F#BDC3C7}Disconnected%{F#FFFFFF}"
else
    echo "%{F#9B59B6}󰖩 %{F#FFFFFF}$ip_address"
fi
```

in ~/.config/scripts/connection/wlan.sh

- Set

```
#!/bin/zsh

ip_address=$(/usr/bin/ip addr show {EthernetNetworkCard} | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)
if [ -z "$ip_address" ]; then
    echo "%{F#E74C3C}󰈀 %{F#FFFFFF}%{F#BDC3C7}Disconnected%{F#FFFFFF}"
else
    echo "%{F#9B59B6}󰈀 %{F#FFFFFF}$ip_address"
fi
```

in ~/.config/scripts/connection/eth.sh

- Set

```
#!/bin/zsh

if /usr/bin/ip link show {HTBNetworkCard} > /dev/null 2>&1; then
    ip_address=$(/usr/bin/ip addr show {HTBNetworkCard} | grep 'inet ' | awk '{print $2}' | cut -d/ -f1)
else
    ip_address=""
fi
if [ -z "$ip_address" ]; then
    echo "%{F#2ECC71}󰆧 %{F#FFFFFF}%{F#BDC3C7}Disconnected%{F#FFFFFF}"
else
    echo "%{F#2ECC71}󰆧 %{F#FFFFFF}%{F#2ECC71}$ip_address%{F#FFFFFF}"
fi
```

in ~/.config/scripts/connection/htb.sh

- Set

```
#!/bin/zsh

battery_level=$(cat /sys/class/power_supply/{BAT}/capacity)
charging_status=$(cat /sys/class/power_supply/{ACAD}/online)
get_battery_icon() {
    if [ $1 -lt 10 ]; then
        echo "%{F#E74C3C}󰁺%{F#FFFFFF}"
    elif [ $1 -lt 20 ]; then
        echo "%{F#E74C3C}󰁻%{F#FFFFFF}"
    elif [ $1 -lt 30 ]; then
        echo "%{F#E74C3C}󰁼%{F#FFFFFF}"
    elif [ $1 -lt 40 ]; then
        echo "%{F#F39C12}󰁽%{F#FFFFFF}"
    elif [ $1 -lt 50 ]; then
        echo "%{F#F39C12}󰁾%{F#FFFFFF}"
    elif [ $1 -lt 60 ]; then
        echo "%{F#F39C12}󰁿%{F#FFFFFF}"
    elif [ $1 -lt 70 ]; then
        echo "%{F#2ECC71}󰂀%{F#FFFFFF}"
    elif [ $1 -lt 80 ]; then
        echo "%{F#2ECC71}󰂁%{F#FFFFFF}"
    elif [ $1 -lt 90 ]; then
        echo "%{F#2ECC71}󰂂%{F#FFFFFF}"
    else
        echo "%{F#2ECC71}󰁹%{F#FFFFFF}"
    fi
}
get_charging_icon() {
    if [ $1 -eq 1 ]; then
        echo "%{F#9B59B6}󰂄%{F#FFFFFF}"
    else
        echo "$(get_battery_icon $battery_level)"
    fi
}
echo "$(get_charging_icon $charging_status) $battery_level%"
```

in ~/.config/scripts/battery/battery.sh

### Setting Picom

- mkdir ~/.config/picom && cp /usr/share/doc/picom/picom.conf.example ~/.config/picom/picom.conf
- Change to `vsync = false;` in ~/.config/picom/picom.conf
- Change to `shadow = false;` in ~/.config/picom/picom.conf
- Change to `corner-radius = 15` in ~/.config/picom/picom.conf

### Setting Feh

- mkdir ~/.config/wallpapers

### Setting Target

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
- dosfstools - Tools to create and check MS-DOS FAT file systems
- feh - Image viewer and background setter (RECOMMENDED)
- firefox - Web browsing (RECOMMENDED)
- gimp - Image and graphics editing
- i3lock - Screen locking (RECOMMENDED)
- keepass - Password management
- kdenlive - Video editing
- man-db - Manual page utilities
- neofetch - Displaying system information
- openvpn - Virtual Private Network (VPN) solution
- pavucontrol - Audio control
- pulseaudio - Sound server (RECOMMENDED)
- qbittorrent - Torrent file downloading
- scrot - Screenshot tool (RECOMMENDED)
- thunar - File management
- timeshift - System backup and restore
- vlc - Media file playback
- xclip - Clipboard manager (RECOMMENDED)
- xorg-xrandr - Display configuration (RECOMMENDED)

## ADDITIONAL FEATURES

- If the computer's time is changed, this command will probably solve it: `sudo timedatectl set-ntp true`
