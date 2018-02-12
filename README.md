# raspberry pi zero -tips

sudo apt-get install ssmtp
sudo apt-get install mailx
sudo nano /etc/ssmtp/ssmtp.conf

# addline
AuthUser= Your-Gmail@gmail.com
AuthPass=Your-Gmail-Password
FromLineOverride=YES
mailhub=smtp.gmail.com:587
UseSTARTTLS=YES



Reference: https://blog.iamlevi.net/2017/01/send-raspberry-pi-ip-address-gmail-boot/
