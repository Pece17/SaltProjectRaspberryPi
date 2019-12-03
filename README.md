# SaltProjectRaspberryPi

Salt project for Linux Server Management course in Haaga-Helia University of Applied Sciences where we use Raspberry Pi 3 Model B+ as a Salt Master and local Salt Minion


## Project team

- Rio Olanterä
- Lauri Lappalainen
- Pekka Hämäläinen


# 1. Raspberry Pi initialization

First we formatted the SD card so that we would have a clean slate starting this project

We downloaded ```Raspbian Buster with desktop``` from address https://www.raspberrypi.org/downloads/raspbian/ to a Windows workstation and created the image of the OS to an SD card with ```Rufus 3.8``` that we downloaded from address https://rufus.ie/

We then hooked up the Raspberry Pi to a monitor and LAN cable in Servula and the desktop view opened

We opened the terminal with ```Ctrl + Alt + T``` and updated package lists for upgrades and new packages from repositories

```
sudo apt-get update
```

We then upgraded all the existing packages to newer versions

```
sudo apt-get upgrade
```

We find out the OS (Operating system) with this command

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





# 2. 
