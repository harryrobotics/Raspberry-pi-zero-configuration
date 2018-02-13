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

## 2. Connect to Enterprise WPA2 Network (E.g : SUTD network)

**Edit the file `wpa_supplicant.conf`**
```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```
Add the network configuration into wpa_supplicant.conf as follow:

```
network={
ssid = “SUTD_Staff”
key_mgmt=WPA-EAP
eap=PEAP
identity=”xxxxxxxx”
password=”yyyyyyy”
pairwise=CCMP TKIP
group=CCMP TKIP
phase2=”auth=MSCHAPV2”
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
sudo apt-get install mailx
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
echo "Test text" | mailx -s "Test Mail" raspberryorion@gmail.com
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
