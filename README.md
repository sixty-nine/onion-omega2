# Onion Omega2+

This is meant for an Onion Omega2+ with an Arduino Dock and an SD card.

There isn't much here that is not in the official documentation.

Also this is **my** config. I might not have the same requirements as you.

## Links

 * [Onion Omega2 Documentation](https://docs.onion.io/omega2-docs/index.html)
 * [Onion Omega2 Arduino Dock Starter Kit](https://docs.onion.io/omega2-arduino-dock-starter-kit/index.html)
 * [pySerial Documentation](https://buildmedia.readthedocs.org/media/pdf/pyserial/latest/pyserial.pdf)

## First start-up

 * Connect to the network "Onion-XXXX".
 * Go to `http://192.168.3.1`
 * Configure local network

```
ssh root@192.168.3.1
```

You are not connected to the Omega.

```
# Change root password
passwd

# Change IP address
uci set network.wlan.ipaddr=192.168.69.69
uci commit network
/etc/init.d/network restart

# Create /root/profile
vim ~/.profile

# Copy your ssh keys
vi /etc/dropbear/authorized_keys
```

## Use SD card as main storage

### Format to ext4

```
opkg update
opkg install e2fsprogs
opkg install block-mount

umount /mnt/mmcblk0p1/
mkfs.ext4 /dev/mmcblk0p1
mount /dev/mmcblk0p1 /mnt/mmcblk0p1
tar -C /overlay -cvf - . | tar -C /mnt/mmcblk0p1/ -xf -
umount /mnt/mmcblk0p1/

block detect > /etc/config/fstab
vi /etc/config/fstab
#
# option  target  '/mnt/<device name>' 
# -->
# option target '/overlay'
#
# option  enabled '0'
# -->
# option  enabled '1'
#
reboot
```

## Enable OpenWRT packages

```
vim /etc/opkg/distfeeds.conf
# Uncomment stuff...
opkg update
```

### Stuff to install

```
opkg install mc htop screen wget
opkg install arduino-dock-2
opkg install git git-http ca-bundle
opkg install python3 python3-pip python-pyserial
opkg install node node-npm
```

## PHP

```
opkg install php7 php7-cgi php7-cli
opkg install php7-mod-mbstring php7-mod-zip \
    php7-mod-pdo-sqlite php7-mod-pcntl php7-mod-openssl \
    php7-mod-hash php7-mod-json php7-mod-phar \
    php7-mod-pdo php7-mod-fileinfo

vim /etc/config/uhttpd
# list interpreter ".php=/usr/bin/php-cgi"
# option index_page 'index.php'

/etc/init.d/uhttpd restart
```

You have to set a timezone in `/etc/php.ini`.
Take one of the timezones you have in `/usr/share/zoneinfo/`.

```
date.timezone = GMT+1
```

## TODO

 * Change WiFi password

## Files

### .profile

```
alias la='ls -lah'
alias l='ls -lh'
alias ..='cd ..'
alias df='df -h'
```

## Arduino

### Serial monitor

 * Detach session: ctrl+a d
 * Kill session: ctrl+a k
 * Reatach session: `screen -r`

```

# Receive data
cat /dev/ttyS1

# Monitor
screen /dev/ttyS1 9600

# Send message
echo "my message" > /dev/ttyS1

# Send message without CRLF
echo -ne 'ArduinoDock2' > /dev/ttyS1
```

## Swap

```
opkg install swap-utils block-mount
dd if=/dev/zero of=/home/swap/swap.page bs=1M count=256
mkswap /home/swap/swap.page
swapon /home/swap/swap.page
```
