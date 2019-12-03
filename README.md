# SaltProjectRaspberryPi

Salt project for Linux Server Management course in Haaga-Helia University of Applied Sciences where we use Raspberry Pi 3 Model B+ as a Salt Master and local Salt Minion


## Project team

- Rio Olanterä
- Lauri Lappalainen
- Pekka Hämäläinen


# 1. Raspberry Pi

We are using Raspberry Pi 3 Model B+ on this project with Raspbian Buster operating system


## 1.1. Raspberry Pi initialization

First we formatted the SD card so that we would have a clean slate starting this project

We downloaded ```Raspbian Buster with desktop``` from address https://www.raspberrypi.org/downloads/raspbian/ to a Windows workstation 

<img src="http://myy.haaga-helia.fi/~bgg135/kuvat/Raspian_buster_image.JPG">

Then we created the image of the OS (Operating system) to an SD card with ```Rufus 3.8``` that we downloaded from address https://rufus.ie/

<img src="http://myy.haaga-helia.fi/~bgg135/kuvat/Rufus_screenshot.JPG">

We then hooked up the Raspberry Pi to a monitor and LAN cable in Servula and the desktop view opened

We opened the terminal with ```Ctrl + Alt + T``` and updated package lists for upgrades and new packages from repositories

```
sudo apt-get update
```

We then upgraded all the existing packages to newer versions

```
sudo apt-get upgrade
```

We find out the OS with this command

```
cat /etc/os-release
```

The output shows that we have the correct Raspbian version in use currently

```
PRETTY_NAME="Raspbian GNU/Linux 10 (buster)"
NAME="Raspbian GNU/Linux"
VERSION_ID="10"
VERSION="10 (buster)"
VERSION_CODENAME=buster
ID=raspbian
ID_LIKE=debian
HOME_URL="http://www.raspbian.org/"
SUPPORT_URL="http://www.raspbian.org/RaspbianForums"
BUG_REPORT_URL="http://www.raspbian.org/RaspbianBugs"
```

We use the following command to find out our current dynamic IPv4 (Internet Protocol version 4) and IPv6 (Internet Protocol version 6) addresses, since we can't get static IP addresses in this time range

```
hostname -I
```

## 1.2. Creating a new user on Raspberry Pi

Create a new user ```chief``` for security reasons, since we don't want to use the default root user ```pi```

```
sudo adduser chief
```

The following output appears and we need to assign a new password and repeat it - we press ```Enter``` for empty for all other values, and finally press ```Y``` to confirm the information

```
Adding user `chief' ...
Adding new group `chief' (1001) ...
Adding new user `chief' (1001) with group `chief' ...
Creating home directory `/home/chief' ...
Copying files from `/etc/skel' ...
Uusi salasana: 
Anna uudelleen uusi salasana: 
passwd: salasanan päivitys onnistui
Muutetaan käyttäjän chief tietoja
Syötä uusi arvo tai paina ENTER jättääksesi oletuksen
	Koko nimi []: 
	Huonenumero []: 
	Työpuhelin []: 
	Kotipuhelin []: 
	Muu []: 
Is the information correct? [Y/n]
```

Give user ```chief``` sudo (superuser do) admin privileges

```
sudo adduser chief sudo
```

Add new user ```chief``` to same groups as root user ```pi```

```
for GROUP in $(groups pi | sed -e 's/^pi //'); do
sudo adduser chief $GROUP; done
```

Copy ```nopasswd``` rules from root user ```pi``` to user ```chief```

```
sudo cp /etc/sudoers.d/010_pi-nopasswd /etc/sudoers.d/010_chief-nopasswd
```

Give the following command related to ```nopasswd``` rules

```
sudo chmod u+w /etc/sudoers.d/010_chief-nopasswd
```

Give the following command related to ```nopasswd``` rules

```
sudo sed -i 's/pi/chief/g' /etc/sudoers.d/010_chief-nopasswd
```

Give the following command related to ```nopasswd``` rules

```
sudo chmod u-w /etc/sudoers.d/010_chief-nopasswd
```

Reboot the system 

```
sudo reboot
```

If the log in screen opens after rebooting, log in as user ```chief```

If the log in screen does not open automatically and opens the root user ```pi``` desktop instead, open the terminal and log in manually as user ```chief```

```
su - chief
```

Delete root user ```pi```

```
sudo deluser -remove-home pi
```

Delete ```nopasswd``` rules from root user ```pi```

```
sudo rm -vf /etc/sudoers.d/010_pi-nopasswd
```


## 1.3. Configuring firewall on Raspberry Pi

Install UFW (Uncomplicated Firewall)

```
sudo apt-get install ufw
```

Check firewall status

```
sudo ufw status
```

Allow port 22 that is responsible for SSH (Secure Shell)

```
sudo ufw allow 22/tcp
```

Allow port 80 that is responsible for HTTP (Hypertext Transfer Protocol)

```
sudo ufw allow 80/tcp
```

Allow port 443 that is responsible for HTTP over TLS/SSL (Transport Layer Security/Secure Sockets Layer)

```
sudo ufw allow 443/tcp
```

Allow port 4505 that is responsible for Salt Master

```
sudo ufw allow 4505/tcp
```

Allow port 4506 that is also responsible for Salt Master

```
sudo ufw allow 4506/tcp
```

Enable firewall after allowing the desired ports

```
sudo ufw enable
```

Check firewall status again

```
sudo ufw status
```

Output shows that firewall is not active with previously allowed ports

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
4505/tcp                   ALLOW       Anywhere                  
4506/tcp                   ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
4505/tcp (v6)              ALLOW       Anywhere (v6)             
4506/tcp (v6)              ALLOW       Anywhere (v6)
```


## 1.4. Changing hostname on Raspberry Pi

Replace the old hostname with ```raspi```

```
sudo hostnamectl set-hostname raspi
```

Check that the new hostname was changed

```
hostname
```

Edit ```/etc/hosts``` file

```
sudo nano /etc/hosts
```

Add new hostname ```raspi``` after ```127.0.1.1```

```
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1       raspi
```

Reboot the system

```
sudo reboot
```


## 1.5. Installing Salt Master on Raspberry Pi

Install Salt Master

```
sudo apt-get -y install salt-master
```

Install Salt Minion

```
sudo apt-get -y install salt-minion
```

Edit ```/etc/salt/minion``` file

```
sudo nano /etc/salt/minion
```

Change ```# master: salt``` to ```master: localhost``` and remove the hashtag

```
#### Primary configuration settings #####
##########################################
# This configuration file is used to manage the behavior of the Salt Minion.
# With the exception of the location of the Salt Master Server, values that are
# commented out but have an empty line after the comment are defaults that need
# not be set in the config. If there is no blank line after the comment, the
# value is presented as an example and is not the default.

# Per default the minion will automatically include all config files
# from minion.d/*.conf (minion.d is a directory in the same directory
# as the main minion config file).
#default_include: minion.d/*.conf

# Set the location of the salt master server. If the master server cannot be
# resolved, then the minion will fail to start.
master: localhost
```

Also, change ```# id:``` to ```id: minion2``` and remove the hashtag

```
# Explicitly declare the id for this minion to use, if left commented the id
# will be the hostname as returned by the python call: socket.getfqdn()
# Since salt uses detached ids it is possible to run multiple minions on the
# same machine but with different ids, this can be useful for salt compute
# clusters.
id: minion2
```

Restart Salt Minion

```
sudo service salt-minion restart
```

Restart Salt Master

```
sudo service salt-master restart
```

List all public keys

```
sudo salt-key L
```

Accept the specified public key for ```minion2```

```
sudo salt-key -a 'minion2'
```

The ouput asks to accept the key ```minion2``` for which we press ```Y```

```
The following keys are going to be accepted:
Unaccepted Keys:
minion2
Proceed? [n/Y] 
```

Send a message to ```minion2``` the minions and tell them to return ```True``` to check which minions are alive

```
sudo salt 'minion2' test.ping
```

Send a message to all the minions and tell them to return ```True``` to check which minions are alive

```
sudo salt '*' test.ping
```
