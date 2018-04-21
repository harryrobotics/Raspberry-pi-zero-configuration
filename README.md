# Raspberry Pi Zero - Configuration tips

## 1. SSH using USB port

(No USB keyboard, mouse or HDMI monitor is needed)

1. Flash Raspbian Jessie full or Raspbian Jessie Lite onto the SD card.

2. Once Raspbian is flashed, open up the boot partition (in Windows Explorer, Finder etc) and add to the bottom of the `config.txt` file `dtoverlay=dwc2` on a new line, then save the file.

3. If using a recent release of Jessie (Dec 2016 onwards), then create a new file simply called ssh in the SD card as well. By default SSH is now disabled so this is required to enable it. Remember - Make sure your file doesn't have an extension (like .txt etc)!

4. Finally, open up the `cmdline.txt`. Be careful with this file, it is very picky with its formatting! Each parameter is seperated by a single space (it does not use newlines). Insert `modules-load=dwc2,g_ether` after `rootwait`. To compare, an edited version of the `cmdline.txt` file at the time of writing, can be found here.

5. That's it, eject the SD card from your computer, put it in your Raspberry Pi Zero and connect it via USB to your computer. It will take up to 90s to boot up (shorter on subsequent boots). It should then appear as a USB Ethernet device. You can SSH into it using  `raspberrypi.local` as the address.

>username: pi

>password: raspberry

### To access using vnc-viewer:

https://www.raspberrypi.org/documentation/remote-access/vnc/README.md

## 2. Connect to Enterprise WPA2 Network (E.g : SUTD network)

**Edit the file `wpa_supplicant.conf`**
```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```
Add the network configuration into wpa_supplicant.conf as follow (becareful of `"` sign):
**NOTE**: I have an issue when key in the bellow network information. The space and the `"` sign must be exactly as the one bellow.
```
network={
ssid="SUTD_Staff"
key_mgmt=WPA-EAP
eap=PEAP
identity="xxxxxxxx"
password="yyyyyyy"
pairwise=CCMP TKIP
group=CCMP TKIP
phase2="auth=MSCHAPV2"
}
```
**Continue to edit the file `interfaces`**
```
sudo nano /etc/network/interfaces
```
Add following lines:
```
iface lo inet loopback
iface eth0 inet dhcp
#address 192.168.1.2
#gateway 192.168.0.254
#netmask 255.255.255.0
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
#iface default inet static
#address 192.168.0.110
#netmask 255.255.255.0
#gateway 192.168.0.254

#iface default inet dhcp
```
## 3. Sends IP to an email

### Install ssmtp
```
sudo apt-get install ssmtp
```
### Install mailx
```
sudo apt-get install bsd-mailx
```
### Edit ssmtp config file   
```
sudo nano /etc/ssmtp/ssmtp.conf
``` 
**Addline**
```
 AuthUser= Your-Gmail@gmail.com
 AuthPass=Your-Gmail-Password
 FromLineOverride=YES
 mailhub=smtp.gmail.com:587
 UseSTARTTLS=YES
```
### Test

```
echo "Test text" | mailx -s "Test Mail" your_email@gmail.com
```
### And then configure mailip file
```
sudo nano /etc/network/if-up.d/mailip
```
**Add the content**
```
#!/bin/sh
# Send a mail with the IP address after interface comes up
 
# It is safe to ignore localhost
if [ "$IFACE" = lo ]; then
    exit 0
fi
 
# Only run from ifup.
if [ "$MODE" != start ]; then
    exit 0
fi
 
# We only care about IPv4 and IPv6
case $ADDRFAM in
    inet|inet6|NetworkManager)
        ;;  
    *)  
        exit 0
        ;;  
esac
# We wait for DHCP to assign an IP address
sleep 15
# Store the IP address to a variable
MYIP="$(/bin/hostname --all-ip-addresses)"
 
# Send the mail if the address is not empty
if [ -z "$MYIP" ]; then
    exit 0
else
    echo "$MYIP" | /usr/bin/mail -s "PI is up" your_username@gmail.com
fi
exit 0

```
**make the above script executable**
```
sudo chmod +x /etc/network/if-up.d/mailip
```

**finally, reboot**
### Reference: https://blog.iamlevi.net/2017/01/send-raspberry-pi-ip-address-gmail-boot/

## 4. Serial Port configuration

### Reference:
https://spellfoundry.com/2016/05/29/configuring-gpio-serial-port-raspbian-jessie-including-pi-3/

https://www.raspberrypi.org/documentation/configuration/uart.md

**1.	Enable the GPIO serial port:**
```
sudo nano /boot/config.txt
```
Add the line: `enable_uart=1`
**2.	If you are using the serial port for anything other than the console you need to disable it.**
```
sudo systemctl disable serial-getty@ttyS0.service
```
**3.	You also need to remove the console from the `cmdline.txt`. If you edit this with:**
```
sudo nano /boot/cmdline.txt
```

you will see something like: `dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes root wait`

remove the line: `console=serial0,115200` and **save** and **reboot** for changes to take effect.

**To switch bluetooth to software UART and set /dev/ttyAM0 to real UART**

In Linux device terms, by default, /dev/ttyS0 refers to the mini UART, and /dev/ttyAMA0 refers to the PL011. The primary UART is that assigned to the Linux console, which depends on the Raspberry Pi model as described above, and can be accessed via /dev/serial0.

Keep in mind that this one will remain possible software problem on bluetooth (software UART), but not on Serial (Hardware)

Edit the file `/boot/config.txt` and add the following line at the end :

```
dtoverlay=pi3-miniuart-bt
core_freq=250
```

Edit the file `/lib/systemd/system/hciuart.service` and replace  `/dev/ttyAMA0`  with  `/dev/ttyS0`

If you have a system with udev rules that create `/dev/serial0`  and `/dev/serial1` (look if you have these one), and if so use `/dev/serial1 `.

Then **reboot**
