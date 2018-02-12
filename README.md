# Raspberry pi zero -tips

## Sends IP to an email

### Install ssmtp
```
sudo apt-get install ssmtp*
```
### Install mailx
```
sudo apt-get install mailx*
```
### Edit ssmtp config file   
```
sudo nano /etc/ssmtp/ssmtp.conf*
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
