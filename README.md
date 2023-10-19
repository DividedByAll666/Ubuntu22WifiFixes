# Ubuntu22WifiFixes
#Fixes for WiFi on Ubuntu 22#
Fix1) sudo lshw -class network

*Copy the Product Name/Number of the Network Adapter*


cd /etc/pm/sleep.d


sudo touch config


*Add the line below to the "config" file < Save & Exit*

SUSPEND_MODULES="Wireless-AC 9260"



echo "options  "Wireless-AC 9260" fwlps=N" | sudo tee /etc/modprobe.d/"Wireless-AC 9260".conf

*Replace "Product_Number" with the Adapter's Product Number without quotes*

*Lastly reboot for changes to take effect*



Fix2) nmcli device wifi connect "MY_SSID" password "MY_PW"

*Replace whats in quotes with your wifi ssid and wifi password*



Fix3) At bootup hold the Shift Key (If shift doesnt work hold esc key) to enter advanced mode so that you can switch to an older kernel version

You'll want to switch to kernel version 5.13...OR....5.15.0-37-generic*


Fix4) Reinstall the Network Manager

sudo apt-get install --reinstall network-manager


Fix5) Editing Your Configuration file

sudo nano /etc/network/interfaces

*Amend it to read as follows*

auto lo iface lo inet loopback auto wlan0 iface wlan0 inet dhcp wpa-essid myssid wpa-psk mypasscode

		OR
auto lo
iface lo inet loopback

auto wlp2s0
iface wlp2s0 inet dhcp



*Then, restart the interface*

sudo ifconfig wlp2s0 down && sudo ifconfig wlp2s0 up

		OR

/etc/init.d/networking restart = Restarts networking service


5B)*Just connect the network cable and run ifconfig. If you get an IP-address you're good to go. If not, you need to edit the /etc/network/interfaces file*

*Eth0 with dhcp:

# The loopback network interface
auto lo eth0
iface lo inet loopback

# The primary network interface
iface eth0 inet dhcp

*Eth0 with static IP:

# The loopback network interface
auto lo eth0
iface lo inet loopback

# The primary network interface
iface eth0 inet static
    address 192.168.1.99
    netmask 255.255.255.0
    broadcast 192.168.1.255
    network 192.168.1.0
    gateway 192.168.1.1
dns-nameservers 192.168.1.1

*For wlan0 with dhcp and WEP encryption add:

auto wlan0
iface wlan0 inet dhcp
    wireless-essid myssid
    wireless-key a1b2c3d4e5

*For wlan0 with dhcp and WPA add:

auto wlan0
iface wlan0 inet dhcp
wpa-ssid myssid
wpa-psk a1b2c3d4e5

*Obviously you need to set your own IP, gateway, wifi password etc.*


Fix6) Command to fix Networking issues

sudo sed -i 's/3/2/' /etc/NetworkManager/conf.d/*

sudo reboot

*Also try the commands below*

sudo su

sudo sed -i 's/3/2/' /etc/NetworkManager/conf.d/*

sudo dhclient -r


Fix7) sudo lshw -C network
*Locate configuration: & driver= to find out what wifi driver is being used/installed*


*Once you determine the driver you're using, you can use the following command to show more information about it*

modinfo <driver-name>


*To check what wireless drivers you currently have installed, but not necessarily being used by anything, you can do the following command*

find /lib/modules/$(uname -r)/kernel/drivers/net/wireless -name '*.ko'

*The above command will list all drivers you have installed. This will probably be an exhaustive list, because these are preinstalled drivers on your Ubuntu to make it possible for people to use their wireless drivers as soon as they install Ubuntu.*





*Solution 1: For Slow WiFi in Atheros Wireless Network Adapters*

*First, you need to find your wireless network adapter in Linux. You can do so by using the command below in the terminal:

lshw -C network

*If your adapter manufacturer is Atheros, this solution should work for you.

*Open a terminal and use the following commands one by one:

sudo su
echo "options iwlwifi nohwcrypt=8" >> /etc/modprobe.d/iwlwifi.conf

*Here, you’re basically enabling a module to use software-based encryption over hardware encryption for your adapter.
This will add the additional line to the configuration file. Restart your computer and you should be good to go.
If it does not fix or if you don’t have Atheros WiFi adaptor, try other solutions.




*Solution 2: Disable 802.11n (works best if you have older router)*

*The next trick is to force disable the 802.11n protocol. Even after so many years, most of the world runs 802.11a,b and g. While 802.11n provides better data rate, not all the routers support it, especially the older ones. It has been observed that disabling the 802.11 n helps speed up the wireless connection in Ubuntu and other OS. Open the terminal and use the following command:

sudo rmmod iwlwifi
	(OR)
sudo rmmod iwlmvm

*If the output says rmmod: ERROR: Module iwlwifi is in use by: iwlmvm
then use the command(s) below to solve the error message*

sudo rmmod iwlwifi -f = -f stands for Force

sudo modprobe iwlwifi = Loads the driver

sudo modprobe iwlwifi 11n_disable=8

*It’s worth noting that in newer kernels, doing this will also disable the 802.11ac protocol and will limit the device throughput to 54 Mbps as mentioned in Gentoo’s wiki page. If you find no significant increase in the wireless connection speed, restart the computer to revert the changes and forget about this solution. But, if it worked for you and you have a faster WiFi now, you should make the changes permanent by using these commands:

sudo su

echo "options iwlwifi 11n_disable=8" >> /etc/modprobe.d/iwlwifi.conf

sudo modprobe -rfv iwldvm

sudo modprobe -rfv iwlwifi

sudo modprobe -v iwlwifi

sudo modprobe -rfv iwlmvm

*Restart your computer and live your life at full speed.





*Solution 3: Fix the bug in Debian Avahi-daemon*

*The slow WiFi in Ubuntu problem could also be related to a bug in Avahi-daemon of Debian. Ubuntu and many other Linux distribution are based on Debian so this bug propagates to these Linux distributions as well.To fix this bug, you have to edit the nsswitch configuration file. Open a terminal and use the following command:

sudo nano /etc/nsswitch.conf


*This will open the configuration file in gedit so that you could edit it easily in GUI. You can also use nano instead of gedit if you want to use the terminal. In here, look for the following line:

hosts:          files mdns4_minimal [NOTFOUND=return] dns

*If you find this file, replace it with the following line:

hosts:          files dns

*Save it, close it, restart your computer. It should fix the slow wireless connection problem for you. If it doesn’t, check the other solutions.
*OR try this fix below with the same file /etc/nsswitch.conf*

default = hosts:          files mdns4_minimal [NOTFOUND=return] dns

changed = hosts:          dns files mdns4_minimal [NOTFOUND=return]




*Solution 4: Disable IPv6 support*

*Yes, you heard it right. Let’s go back to the previous century and care about IPv4 by ditching IPv6 support. Sometimes, you don’t even need IPv6 support. Moreover, if it improves the Wi-Fi speed in some cases. So, you can surely give it a try if nothing else is working for you. To disable IPv6 support in Ubuntu, use the following commands one by one:

sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1 && sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1 && sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1

*Do note that you’re disabling IPv6 temporarily here. If it doesn’t work, simply reboot your system and you will have IPv6 enabled. But, if it does work, you can type in the following commands to make the change permanent:

sudo su
echo "#disable ipv6" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
echo "net.ipv6.conf.lo.disable_ipv6 = 1" >> /etc/sysctl.conf

*Restart your computer and it should do the magic. If not try the next one.





*Solution 5: Ditch default network manager and embrace Wicd (possibly obsolete)*

*This is only applicable if you’re using Ubuntu 16.04 or lower (which is highly unlikely at the time of updating this article) — but this was a working solution back then. If you have Ubuntu 16.04 for some reason, you can try it out. Slow or inconsistent wireless connection, in some cases, are also due to Ubuntu’s very own default network manager. I am not sure what causes this but I have seen people in Ubuntu Forums talking about this problem in especially in Ubuntu. You can install Wicd, an alternative network manager from Ubuntu Software Centre or from the terminal. For details on how to use Wicd, you can read my other article which I used to find SSID of wireless networks in Ubuntu.





*Solution 6: More power to wireless adaptor (possibly obsolete)*

*This trick should be obsolete and this is why I mentioned it in the end. But, it looks like some of you find it working even with Ubuntu 20.04. Linux Kernel has a power management system which comes in handy. However, for some users with their wireless connection it sent less power to the wireless adapter and thus affecting its performance. As a result, wireless connection would be some times fast and some times dead slow. While this is probably fixed in later Kernels, systems running older Linux Kernel may still face it. Open a terminal and use the following command:

sudo iwconfig

*It will give you the name of your wireless device. Normally it should be wlan0. Now use the following command:

sudo iwconfig wlan0 power off

*This will switch off the special power management system for the wireless adaptor and thus it will get more power and will work more (lame attempt at humour).

*Also try the command below < Then reboot your system*

sudo sed -i 's/3/2/' /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf 


*Solution 7: Editing GRUB with Disabling Active State Power Management (ASPM)
sudo nano /etc/default/grub 

# Edit the following line and add pcie_aspm=off			*Default value=GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" *

GRUB_CMDLINE_LINUX_DEFAULT="splash pcie_aspm=off"

sudo update-grub


*Solution 8:

echo "options iwlwifi 11n_disable=8 swcrypto=1 bt_coex_active=0 power_save=0 " | sudo tee /etc/modprobe.d/iwlwifi.conf

							OR

sudo tee /etc/modprobe.d/iwlwifi-3options.conf

*Hereby you achieve three things:

- With bt_coex_active=0 you disable the Bluetooth feature of the WiFi chipset, which sometimes interferes;

- With swcrypto=1 you shift the signal encryption from the hardware (WiFi chipset) to the software, thus taking some load off the WiFi chipset;

- With 11n_disable=8 you enable antenna aggregation (Tx AMPDU). Don't be confused because of the option name 11n_disable: when its value is set to 8 it does not disable anything, but enables transmission antenna aggregation (Tx AMPDU).*

Then Reboot your pc.


*No improvement? Then undo this hack, by removing the file that contains the toggled options, with this terminal command:

sudo rm -v /etc/modprobe.d/iwlwifi-3options.conf

*Reboot your computer and proceed with item 3 below.*


# Also try adding these to /etc/modprobe.d/iwlwifi.conf

options iwlwifi 11n_disable=8
options iwlwifi swcrypto=1
options iwlwifi bt_coex_active=0
options iwlwifi power_save=0
options iwlwifi nohwcrypt=8
options iwlwifi amsdu_size=3
options iwlmvm power_scheme=3
options cfg80211 cfg80211_disable_40mhz_24ghz=1
options cfg80211 ieee80211_regdom=US
options iwlwifi uapsd_disable=0
options iwlwifi power_save=Y 
options iwlwifi power_level=5
options mac80211 ieee80211_default_rc_algo=minstrel_ht


*Always run the command below after changing any content in /etc/modprobe.d/*

sudo update-initramfs -u

*Backup the iwlwifi.conf file*

sudo mv /etc/modprobe.d/iwlwifi.conf iwlwifi.conf.bak


*Save changes and reboot with the command below*

sudo init 6

*Confirm the module options are in effect with command below*

sudo systool -vm iwlmvm | grep power

sudo systool -vm iwlwifi | grep power



*Solution 9:Turn off NetworkManager Connectivity Checking

sudo nano /etc/NetworkManager/conf.d/20-connectivity.conf

*Add the contents below to '20-connectivity.conf' file*

[connectivity]
uri=http://www.archlinux.org/check_network_status.txt
interval=0

*Now Save & Exit Then Restart NetworkManager with the cmd below*

sudo systemctl restart NetworkManager


*Solution 10: sudo nano /etc/gai.conf 

*Look for the line with precedence ::ffff:0:0/96 100 and removed the # character that preceded it. save changes and reboot. if not working add # back.*


*Solution 11: 
sudo apt install -y dkms bcmwl-kernel-source dkms backport-iwlwifi-dkms

sudo apt-add-repository -p proposed && sudo apt update

sudo apt install -y linux-generic

sudo apt-add-repository -r -p proposed && sudo apt update

sudo apt install -y linux-oem-22.04

*To Remove these use the commands below*

sudo apt remove --purge -y bcmwl-kernel-source linux-generic linux-oem-22.04 dkms backport-iwlwifi-dkms



*Solution 12:
sudo nano /etc/modprobe.d/blacklist.conf

add blacklist acer_wmi as a new line at the bottom of this file 

sudo update-initramfs -u

Then Reboot


*Solution 13:
echo "options hp_wmi wireless=1" | sudo tee /etc/modprobe.d/hp_wmi.conf

sudo update-initramfs -u


*Solution 14:Disable VT-d entry in BIOS (OR) use the GRUB setting
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=off"

sudo update-grub


*Solution 15:
sudo systemctl stop NetworkManager.service

sudo rm /var/lib/NetworkManager/NetworkManager.state

sudo systemctl start NetworkManager.service


*Solution 16:
sudo iw reg get = Shows current country code selected

sudo iw reg set US = Sets Country Code to USA

sudo nano /etc/default/crda

*Contents*

REGDOMAIN=US


*Solution 17:
sudo lsmod | grep iwlwifi

*If output displays iwlwifi in red continue with solution*

echo "options iwlwifi 11n_disable=8" | sudo tee /etc/modprobe.d/iwlwifi-speed.conf

sudo update-initramfs -u

*To undo the changes made with the above command use the command below and then reboot*

sudo rm -rf /etc/modprobe.d/iwlwifi-speed.conf

sudo update-initramfs -u


*Solution 18:
cd /etc/modprobe.d/

sudo nano iwlwifi.conf  (OR) mac80211.conf

*Add the line below*

options mac80211 ieee80211_default_rc_algo=minstrel_ht

sudo update-initramfs -u


*Solution 19:
sudo nano /etc/NetworkManager/dispatcher.d/wifi-powerman-off.sh

*Add contents below to wifi-powerman-off.sh*


#!/usr/bin/env sh

IFACE=$1 STATUS=$2
IW=/usr/bin/iw
WLAN_IFACE=wlan0

if [ ! -x $IW ]; then
	exit 1
fi

if [ "$IFACE" = $WLAN_IFACE ] && [ "$STATUS" = "up" ]; then
	echo "[SCRIPT] Disabling wifi power mgmt"
	$IW dev $IFACE set power_save off
fi


*Save & Exit then chmod u+x wifi-powerman-off.sh*


*Solution 20: cd /etc/modprobe.d/

echo "options ath9k nohwcrypt=1" | sudo tee /etc/modprobe.d/ath9k.conf

echo "options ath5k nohwcrypt=1" | sudo tee /etc/modprobe.d/ath5k.conf

echo "options ath10k_core skip_otp=y" | sudo tee /etc/modprobe.d/ath10k.conf

sudo update-initramfs -u


*Solution 21: 
cd /etc/systemd/system/

sudo nano wifi-resume.service

*Contents*

[Unit]
Description=Restart networkmanager at resume
After=suspend.target
After=hibernate.target
After=hybrid-sleep.target

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart network-manager.service

[Install]
WantedBy=suspend.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target

sudo systemctl enable /etc/systemd/system/wifi-resume.service

sudo systemctl enable wifi-resume.service

sudo systemctl restart NetworkManager.service

sudo systemctl status wifi-resume.service


sudo nano /lib/systemd/system/wifi-resume.service

*Contents*

#/lib/systemd/system/wifi-resume.service
#sudo systemctl enable wifi-resume.service
[Unit]
Description=Restart network-manager at resume
After=suspend.target
After=hibernate.target
After=hybrid-sleep.target 

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart network-manager

[Install]
WantedBy=suspend.target
WantedBy=hibernate.target
WantedBy=hybrid-sleep.target


sudo systemctl enable /lib/systemd/system/wifi-resume.service





Solution 22:
sudo nano /lib/systemd/system-sleep/wififix

*Content*

#!/bin/bash

set -e

if [ "$2" = "suspend" ] || [ "$2" = "hybrid-sleep" ]; then

case "$1" in

pre) true ;;

post) sleep 1 &amp;&amp; service network-manager restart ;;

esac

fi

		(OR)
		
#!/bin/sh

# /lib/systemd/system-sleep/network-manager

# Restarts network-manager after sleep.
# This prevents the problem where the network is not
# available after resume.

case $1 in
    post)
        /usr/bin/sleep 10
        /usr/bin/systemctl restart network-manager
        ;;
esac

		(OR)
		
#!/bin/sh
set -e
if [ "$2" = "suspend" ] || [ "$2" = "hybrid-sleep" ]; then
     case "$1" in
             pre) true ;;
             post) sleep 1 && service network-manager restart ;;
       esac
fi



sudo chmod +x /lib/systemd/system-sleep/wififix


Solution 23:
sudo nano /etc/pm/sleep.d/10_resume_wifi

*Content*

#!/bin/sh

 case "${1}" in
   resume|thaw)
     nmcli r wifi off && nmcli r wifi on ;;
esac


		(OR)

#!/bin/sh
		
case "${1}" in
    resume|thaw)
        # systemctl restart network-manager.service
        service NetworkManager restart
;;
esac

sudo nano /lib/systemd/system-sleep/hdparm

*Contents*

#!/bin/sh

case $1 in 
  post)
    /usr/lib/pm-utils/power.d/95hdparm-apm resume
    ## Paste your command to run your script
    ;; esac
    
    
Solution 24: cd /etc/modprobe.d/

sudo nano audio_disable_powersave.conf

*Conetents*
options snd_hda_intel power_save=0
options snd_usb_audio power_save=0

Solution 25: Add iwconfig wlan0 power off to run at startup via crontab -e

sudo iwconfig wlan0 power off
sudo crontab -e

*Add the line below at the bottom of crontab file*
@reboot iwconfig wlan0 power off

*Save and Exit*

sudo systemctl restart cron.service rsyslog.service

*Next reboot your machine and then run iwconfig to see if power saver mode is off...if not run sudo iwconfig wlan0 power off and reboot again*


Solution 26: Setting TTL 

sudo sysctl net.ipv4.ip_default_ttl=65

echo 65 | sudo tee /proc/sys/net/ipv4/ip_default_ttl

*Next navigate to the path below*

sudo nano /etc/sysctl.conf

*Then add the line(s) below to '/etc/sysctl.conf'*

net.ipv4.ip_default_ttl=65


Soulution 27: Setting MTU Size/ disabling ipv6 via nmcli / setting txpower to auto via iw

sudo ip link set dev wlan0 mtu 1200 && sudo nmcli device modify wlan0 ipv6.method disabled && sudo iw dev wlan0 set txpower auto && sudo systemctl restart NetworkManager
