# Salt Project Raspberry Pi

Salt project for Linux Server Management course in Haaga-Helia University of Applied Sciences where we use Raspberry Pi 3 Model B+ as a Salt Master and local Salt Minion


## Project team

- Rio Olanterä
- Lauri Lappalainen
- Pekka Hämäläinen


# 1. Raspberry Pi

We are using ```Raspberry Pi 3 Model B+``` on this project with ```Raspbian Buster with desktop``` operating system

<img src="http://myy.haaga-helia.fi/~bgg135/kuvat/raspberry.jpg">


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
sudo salt-key -L
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

Send a message to ```minion2``` minion and tell it to return ```True``` to check which minions are alive

```
sudo salt 'minion2' test.ping
```

Send a message to all the minions and tell them to return ```True``` to check which minions are alive

```
sudo salt '*' test.ping
```


# 1.6. Creating Salt states on Raspberry Pi

Create ```/srv/salt/``` directory where all the Salt states will 

```
sudo mkdir /srv/salt/
```

Go to ```/srv/salt/``` directory

```
cd /srv/salt/
```

We can create a ```kansio``` example folder using Salt command to ```minion2``` 

```
sudo salt 'minion2' cmd.run 'mkdir /home/chief/kansio'
```

We can create a ```kansio``` example folder using Salt command for all minions, if there were others

```
sudo salt '*' cmd.run 'mkdir /home/chief/kansio'
```

We try to create an example Salt state ```vim.sls``` which would install Vim text editor using instructions from https://www.digitalocean.com/community/tutorials/how-to-create-your-first-salt-formula

```
sudo nano vim.sls
```

We paste the following text inside ```vim.sls``` file

```
vim:
  pkg:
    - installed
```

We try to apply the state to ```minion2```

```
sudo salt 'minion2' state.sls vim
```

We try to apply the state to all minions

```
sudo salt '*' state.sls vim
```

The installation does not work for some reason and the following output appears

```
minion2:
----------
          ID: vim
    Function: pkg.installed
      Result: False
     Comment: An error was encountered while installing package(s): E: The repository 'https://repo.saltstack.com/apt/debian/10/armhf/latest buster Release' does not have a Release file.
     Started: 16:10:27.350873
    Duration: 17907.016 ms
     Changes:   

Summary for minion2
------------
Succeeded: 0
Failed:    1
------------
Total states run:     1
Total run time:  17.907 s
ERROR: Minions returned with non-zero exit code
chief@raspi:/srv/salt $ sudo vi
vi        view      viewres   vigr      vim.tiny  vipw      visudo    
chief@raspi:/srv/salt $ sudo vi
vi        view      viewres   vigr      vim.tiny  vipw      visudo    
chief@raspi:/srv/salt $ sudo vi
vi        view      viewres   vigr      vim.tiny  vipw      visudo    
chief@raspi:/srv/salt $ sudo vim vim.sls
sudo: vim: komentoa ei löytynyt
chief@raspi:/srv/salt $ sudo salt '*' state.sls vim
minion2:
----------
          ID: vim
    Function: pkg.installed
      Result: False
     Comment: An error was encountered while installing package(s): E: The repository 'https://repo.saltstack.com/apt/debian/10/armhf/latest buster Release' does not have a Release file.
     Started: 16:13:48.811058
    Duration: 21524.096 ms
     Changes:   

Summary for minion2
------------
Succeeded: 0
Failed:    1
------------
Total states run:     1
Total run time:  21.524 s
ERROR: Minions returned with non-zero exit code
```

# r/WitcherMemes CSS

```
Data hosted with ♥ by Pastebin.com - Download Raw - See Original

/*--------------------------------------------------------------------------
        Minimaluminiumalism Base Theme
        Version 1.37
        Header Style A
        Developer Version
        Base theme last modified on January 20, 2016
--------------------------------------------------------------------------*/
 
/*SUBREDDIT LOGO*/
span.pagename.redditname a {
    color: transparent;
    display: block;
    width: 100%; /*DO NOT CHANGE - TWEAK HEIGHT INSTEAD*/
    height: 35px;
    background-image: url(%%sublogo%%);
    background-size: contain;
    background-position: center center;
    background-repeat: no-repeat;
    position: absolute;
    top: 75px;
 }
 
/*HEADER A EXCLUSIVE CODE*/
/* THIS CODE CENTERS THE TAB MENU*/
 
#header::after {
    position: absolute;
    top: 177px;
    width: 100%;
    height: 48px;
    background: rgba(0, 0, 0, 0.5);
    content: " ";
    z-index: -1;
}
 
#header-bottom-left .tabmenu {
    background-color: transparent;
    text-align: center;
    position: absolute;
    left: 0px;
    right: 0px;
    top: 155px;
 }
 
/*END*/
   
/*TABMENU*/
.tabmenu {margin-top: 22px;}
 
    .tabmenu li {
        background-color: transparent;
        margin: 0 !important;
        height: 36px;
        width: 102px;
     }
 
.selected, .choice.primary {font-weight: 600;}
 
a {
    text-decoration: none;
    color: #999;
 }
 
.tabmenu li a {
    padding: 0px 20px!important;
    background-color: transparent;
    border-left: none;
    border-right: none;
    display: inline-block;
    line-height: 48px;
    text-align: center;
    text-transform: Uppercase;
    transition: 150ms;
    -webkit-transition:150ms;
    font-weight: normal;
    color: #e5e5e5 !important;
    text-shadow: 0px 1px 8px rgba(0, 0, 0, 0.5);
 }
 
.tabmenu li:hover a {
    background-color: rgba(255, 255, 255, 0.08);
    color: #FFF !important;
    font-weight: bold !important;
 }
 
.tabmenu li.selected a {
    color: #FFF !important;
    background-color: rgba(255, 255, 255, 0.15);
    font-weight: bold !important;
    border: none !important;
 }
 
 
/*--- Tabs - Hide unimportant ones  ---*/
#header .tabmenu li:nth-of-type(8) {
    display: none;
    visibility: hidden;
 }
 
/*--- Hide Promoted (Tab 8) --- */
.listing-page .tabmenu li a[href*="/gilded"],
.listing-page .tabmenu li a[href*="/rising"],
.listing-page .tabmenu li a[href*="/controversial"]
 
 {display: none;}
 
#header .tabmenu li:nth-of-type(9) {
    display: none;
    visibility: hidden;
 }
 
/*--- Hide View Images (Tab 9) ---*/
/*--- End Tabs - Hide unimportant ones  ---*/
 
#header-img {
    position: relative;
    height: 30px !important;
    width: 30px;
    margin-top: 163px;
    margin-left: 35px;
    background-size: contain !important;
    background-repeat: no-repeat !important;
    z-index: 1;
 }
 
/*END HEADER A EXCLUSIVE CODE*/
/*CORE CODE*/
 
/*UNCATEGORIZED FIXES*/
 
.commentarea span.score {font-weight: bold;}
 
.deepthread::after {background-image: none; content: ">"; font-weight:bold; font-size: 12px; }
 
.deepthread {font-weight: bold; text-transform: uppercase; font-size: 9.2px;}
 
.markdownEditor .edit-btn:not(.btn-macro) {background-color: transparent !important; border: 1px solid #c6c6c6; border-radius: 3px;}
 
#hsts_pixel {display: none;}
 
.side #search_showmore {color: #bbb;}
karmaZ_
.search-expando-button-label-collapsed,
.search-expando-button-label-expanded {
    text-transform: uppercase;
    font-weight: bold;
    font-size: 10px;
}
 
.sr-bar .separator {
    color: transparent;
    font-size: 8px;
}
 
.BEFoot button {margin-right: 5px;}
 
a#sr-more-link[href="https://www.reddit.com/subreddits/"] {
    background: rgba(0, 0, 0, 0.6) none repeat scroll 0% 0% !important;
    color: rgb(255, 255, 255);
    padding: 3px 15px;
}
 
a {color: #666;}
 
.usertext .help-toggle, .usertext a.reddiquette {text-transform: uppercase;}
 
.usertext .help-toggle:hover, .usertext a.reddiquette:hover {font-weight: bold}
 
.thing .title:visited {color: #331f6d;}
 
.flairselector li:hover {
    width: 400px !important;
    background-color: rgb(221, 221, 221);
    border-radius: 20px;
}
 
.flair {cursor: default;}
 
.loadFlat {
    position: relative !important;
    z-index: 1;
    font-weight: bold;
    text-transform: uppercase;
    font-size: 10px;
    margin-left: 10px;
    transition: 200ms;
}
 
.link.last-clicked {border: none;}
 
div#RESStyleSheetTipPane {border: none !important;}
div#RESStyleSheetTipPane-header {border: none !important;}
 
.toc ul {background: #fff}
 
.commentNavSortType:hover, .commentarea .menuarea .toggle a:hover{text-decoration: none !important;}
 
.roundfield textarea, .roundfield input[type="url"], .roundfield input[type="text"] {transition: 300ms; border: none; border-bottom: 1px solid #ccc; font-size: 20px; height: 25px;}
 
    .commentarea textarea:not(:focus):not(:hover) {
            background-color: #F8F8F8 !important;
    }
 
    .commentarea textarea:hover, .roundfield textarea:hover, .roundfield input[type="url"]:hover, .roundfield input[type="text"]:hover {border-bottom: 1px solid #999;}
 
 
    .roundfield input[type=url]:focus,.roundfield input[type=text]:focus,.roundfield textarea:focus,#img-name:focus {border-bottom: 2px solid;}
 
    .usertext-edit textarea:hover{border: 1px solid #999;}
 
    .usertext-edit textarea:focus {border-top: 1px solid #e1e1e1 !important; border-left: 1px solid #e1e1e1 !important; border-right: 1px solid #e1e1e1 !important; border-bottom: 3px solid;}
 
    .usertext-edit textarea {transition: 200ms; height: 100px; font-size: 1em; border: 1px solid #ccc;}
 
 
 
#RESPrefsDropdown {
    background-color: rgba(0, 0, 0, 0.85) !important;
    border: none;
    border-radius: 6px;
 }
 
.RESDropdownList li {
    background-color: transparent;
    color: #fff !important;
    border: none !important;
    transition: all 0.15s ease;
    text-transform: uppercase;
 }
 
    .RESDropdownList li:hover,
    .RESDropdownList li a:hover {
        background-color: rgba(80, 80, 80, 0.6) !important;
        color: #fff !important;
     }
 
#NERFail {
    padding-top: 20px;
    border: none;
 }
 
/* Domain */
.link .domain {visibility: hidden;}
 
    .link .domain a {
        visibility: visible;
        color: #b3b3b3;
        transition: all 0.15s ease;
     }
 
.sr-bar a.gold {color: gold;}
 
.icon-menu a {background: none;}
 
textarea {border: 1px solid rgba(0, 0, 0, 0.12); /*MAKES IT LOOK BETTER ON FIREFOX*/}
 
a.madeVisible {margin-top: 20px;}
 
img.RESImage {border-radius: 12px;}
 
.gold-accent.comment-visits-box {color: black;}
 
.gold-accent {
    background-color: transparent;
    border: 0px;
 }
.nsfw-stamp {
    color: white;
    border: none !important;
    background: #D10023;
    padding: 0px 8px !important;
    border-radius: 20px;
    font-weight: bold;
}
 
.action-form {
    border: medium none;
    max-width: 300px;
    padding: 20px;
    margin: 5px 0px;
    font-size: 12px;
    text-transform: uppercase;
    font-weight: bold;
    line-height: 25px;
    background: rgba(0, 0, 0, 0.85);
    color: white;
    position: absolute;
    z-index: 999;
 }
 
    .action-form button {margin-right: 10px;}
 
    .action-form input:disabled {display: none;}
 
    .action-form ol {margin-bottom: 15px;}
 
.pretty-button {
    background: none !important;
    font-weight: bold;
    padding: 1px 5px;
    border-radius: 20px;
    font-size: 10px !important;
    border: medium none !important;
    text-transform: uppercase;
}
 
.pretty-button.negative {color: #D10023;}
.pretty-button.neutral {color: #444;}
.pretty-button.positive {color: #2B8B27;}
 
.pretty-button.pressed  {font-size: 0px !important;}
.pretty-button.pressed:after {font-size: 10px; content: "Action Done"; text-transform: uppercase; font-weight: bold;}
 
 
button[disabled] {display:none;}
 
ul.report-reasons {background-color: transparent; text-transform: uppercase; border: none;}
 
.content .entry .buttons li.reported-stamp,
body .content .stickied .entry .buttons li.reported-stamp,
body .content .over18 .entry .buttons li.reported-stamp {
    border: none !important;
    background: #666 !important;
    color: white !important;
    font-weight: bold;
    text-transform: uppercase !important;
    padding: 1px 8px;
    border-radius: 20px;
    font-size: 9px !important;
}
 
.commentarea .nestedlisting {margin: 0 -10px;}
 
.res-commentBoxes .nestedlisting > .comment {margin: 0 0 8px!important;}
 
.res-commentBoxes .nestedlisting .comment {overflow: visible!important;}
 
.dropdown.srdrop .selected {color: white;}
 
.flat-list li a.gold {color: gold;}
 
/*MODERATOR END*/
 
.tabpane-content {border: none;}
 
.linefield {background: transparent;}
 
.linefield .title {text-transform: uppercase; font-weight:bold; font-size: 13px;}
 
.flairgrant.flairlist {padding:10px;}
 
.flair-jump button {font-size: 12px;}
 
.fancy-settings h1 {
    margin: 10px 5px;
    text-transform: uppercase;
    padding: 20px 30px 5px;
    font-size: 18px;
}
 
#img-name, .flairlist .flair-jump input[type="text"] {
    border: none;
    border-bottom: 1px solid #ccc;
    transition: 300ms;
    display: block;
    font-size: 22px;
    margin-bottom: 20px;
}
 
#img-name:hover, .flairlist .flair-jump input[type="text"]:hover {border-color: #666;}
 
.flairlisthome {
    text-transform: uppercase;
    font-weight: bold;
    font-size: 12px;
}
 
.stylesheet-customize-container h2 {font-weight:bold; text-transform: uppercase; font-size: 13px;}
 
.pretty-form {padding: 20px 0}
 
#img-preview-container {border: none; background: #444; border-radius: 20px; margin: 25px 10px -25px 0px; padding: 10px;}
 
.linefield.mobile {
    width: 512px;
    background-color: transparent;
    border: none;
}
 
.flairlist.usertable {margin-top: 35px;}
 
.flairrow {
    font-weight: bold;
    text-transform: uppercase;
    font-size: 13px;
}
 
/*Stylesheet page fix*/
.sheets {
    margin-right: 20px;
    margin-bottom: 20px;
 }
 
#preview-table {padding-left: 20px;}
 
div#images {
    margin-left: 20px;
    padding-right: 20px;
 }
 
ul#image-preview-list {
    margin: -15px;
    margin-bottom: -15px;
    margin-top: 20px;
    padding-top: 0px;
 }
 
    ul#image-preview-list li {
        background: #fff;
        padding: 8px 8px;
        margin: 0 0 5px 5px;
        border-radius: 10px;
        border: 1px solid #e1e1e1;
        width: 296.5px;
        height: 100px;
        font-size: 14px;
        color: #666;
     }
 
    ul#image-preview-list .preview {
        position: absolute;
        top: 0;
        bottom: 0;
        left: 8px;
        right: 0;
        margin: auto 0;
        line-height: 98px;
     }
 
        ul#image-preview-list .preview img {vertical-align: middle;}
 
/*SUBREDDIT FONT*/
.login-form-side input[type="text"], .login-form-side input[type="password"],
#search input[type="text"],
.md textarea,
textarea,
body {font-family: -apple-system, San Francisco Display, Helvetica Neue, Helvetica, Arial, sans-serif;}
 
 
/*RES TEMP BUGFIX*/
.title + .expando-button + .expando-button {display: none;}
 
/*DROP DOWN CHOICES*/
.drop-choices.lightdrop {
    background: rgba(0, 0, 0, 0.85);
    border: medium none;
    padding: 5px;
    border-radius: 0px 0px 4px 4px;
    font-weight: bold;
    margin-top: 2px;
    color: rgb(255, 255, 255) !important;
 }
 
.drop-choices a.choice {color: #FFF;}
 
    .drop-choices a.choice:hover {background: rgba(80, 80, 80, 0.6);}
 
/*ICONOGRAPHY*/
 
#mail.nohavemail,#mail.havemail, #modmail.havemail,#modmail.nohavemail, .thumbnail.nsfw, .thumbnail.self, .thumbnail.default, .stickied .thumbnail.self, #header-bottom-right a.pref-lang, .res #header-bottom-right .gearIcon, .arrow.upmod, .arrow.downmod, .arrow.up, .arrow.down,.expando-button {background: url(%%spritesheet%%) !important; background-size: 50px 502px !important;}
 
#mail.nohavemail {background-position: 0 0}
#mail.havemail {background-position: 0 -58.66px !important;}
#modmail.nohavemail {background-position: 0 -82px !important}
#modmail.havemail {background-position: 0 -105.333px !important}
#header-bottom-right a.pref-lang {background-position: 0 -128.666px !important}
.res #header-bottom-right .gearIcon {background-position: 0 -152px !important}
 
#mail.nohavemail,#mail.havemail,#modmail.nohavemail,#modmail.havemail,#header-bottom-right a.pref-lang,.res #header-bottom-right .gearIcon {width: 20px; height: 20px; vertical-align: middle !important;}
 
.thumbnail.nsfw, .thumbnail.self, .thumbnail.default, .stickied .thumbnail.self {width: 75px; height: 50px; background-size: 75px 753px !important;}
 
.stickied .thumbnail.self {background-position: 0 -265px !important;}
.thumbnail.self {background-position: 0 -345px !important}
.thumbnail.default {background-position: 0 -426px !important}
.thumbnail.nsfw {background-position: 0 -506px !important}
 
.arrow.up {background-position: 3px -685px !important; background-size: 37.5px 376.5px !important;}
.arrow.upmod {background-position: 2.3px -666px !important; background-size: 37.5px 376.5px !important;}
.arrow.down {background-position: 3px -699px !important; background-size: 37.5px 376.5px !important;}
.arrow.downmod {background-position: 3px -716px !important; background-size: 37.5px 376.5px !important;}
 
.expando-button,
.expando-button:hover,
.expando-button.expanded:hover,
.expando-button.expanded {background-position: 0 -482px !important;}
 
 
#modmail.havemail,
#modmail.nohavemail {height: 18px; top: -5px; margin-bottom: 0px !important;}
 
.res #modmail.havemail, .res #modmail.nohavemail {margin-bottom: -6px !important;}
 
body.res #header-bottom-right #mail.havemail {top: -1px !important;}
 
#mail.nohavemail,
#mail.havemail,
#modmail.havemail,
#modmail.nohavemail,
#header-bottom-right a.pref-lang.choice {
    background-repeat: no-repeat !important;
    width: 20px !important;
    height: 20px !important;
    top: 0px !important;
}
 
 
 
/*VOTE ARROWS*/
.arrow.downmod,
.arrow.upmod,
.arrow.up,
.arrow.down {
    height: 20px;
    width: 20px;
    border-radius: 20px;
    transition:200ms;
}
 
.arrow.upmod {background-color: orangered !important; margin-bottom: 5px;}
.arrow.downmod {background-color: #2B9BDB !important; margin-top: 5px;}
 
/*EXPANDO BUTTONS*/
.expando-button,
.expando-button:hover,
.expando-button.expanded:hover,
.expando-button.expanded {
    width: 20px !important; height: 20px !important;
    transition: 150ms;
}
 
.expando-button.expanded {transform: rotate(45deg);-webkit-transform: rotate(45deg);}
 
.expando-button:hover,
.expando-button.expanded:hover {opacity: 0.5;}
 
 
#header-bottom-right a.pref-lang {
    display: inline-block;
    text-indent: -9999px;
}
   
 
.message-count {left: 2px; top: -6px; background-color: #ff9400;}
 
/*USER BAR*/
#header-bottom-right {background:none; top: 192px !important; right: 20px; padding: 0px !important;}
#header-bottom-right a, #header-bottom-right .user, #header-bottom-right .user .userkarma {
    color: #FFF;
    font-size: 11px;
    font-weight: bold;
    text-transform: uppercase;
}
 
.res #header-bottom-right #modmail {top: -4px !important;}
 
.separator {visibility:hidden;}
 
.gearIcon {background-repeat:no-repeat !important;}
 
#RESMainGearOverlay {display: none !important;}
 
/*USERBAR TOGGLE*/
.res #userbarToggle {
    width: 11px;
    height: 48px;
    background-color: transparent;
    border: medium none;
    line-height: 47px;
    color: #FFF;
    padding-left: 5px;
    padding-right: 8px;
    margin-left: -6px;
    transition: all 200ms ease 0s;
    bottom: 2px;
    border-radius: 0px;
    margin-bottom: -15px;
}
 
.res #userbarToggle:hover {
    background-color: rgba(255, 255, 255, 0.1);
}
 
#userbarToggle.userbarShow {
    background-color: rgba(255, 255, 255, 0.1);
    left: -3px !important;
    padding-right: 20px;
    font-size: 25px;
    padding-left: 15px;
    margin-left: -12px;
    transition:200ms;
    top: -15px !important;
}
#userbarToggle.userbarShow:hover {
    background-color: rgba(255, 255, 255, 0.3);
}
 
/*GENERAL RES FIXES*/
.livePreview, .livePreview .md {text-transform: none;}
 
.RESDialogSmall.livePreview {
    margin: 15px 0px;
}
 
.new-comment .usertext-body {
    border: none !important;
 }
 
/*AD fixes*/
.organic-listing {border: 0px;}
 
.infobar.newsletterbar {display: none !important;}
 
/*MENU AREA*/
.menuarea {overflow: visible;}
 
body:not(.search-page) .menuarea {
    font-size: 12px;
    font-weight: bold;
    text-transform: uppercase;
    margin-top: 20px !important;
 }
 
.comments-page .menuarea {
    font-size: 12px;
    font-weight: bold;
    text-transform: uppercase;
    margin-top: 5px !important;
 }
 
/*SEARCH PAGE*/
.combined-search-page .searchfacets {
    margin: 0px !important;
    max-width: none !important;
 }
 
    .combined-search-page .searchfacets > h4.title {color: #444;}
 
.combined-search-page .search-result-group {
    padding-left: 0px;
    padding-right: 0px;
    max-width: 100%
 }
 
.combined-search-page .search-result {
    margin-bottom: 25px;
    margin-top: 10px;
    padding: 0px 30px;
    border-bottom: 1px solid #E1E1E1;
    padding-bottom: 20px;
 }
 
.combined-search-page .search-result-group .search-result-group-header {
    color: inherit;
    margin-bottom: 20px;
    margin-top: 20px;
    padding: 15px 30px;
    text-transform: uppercase;
    font-weight: bold;
    background: #eee;
    border-top: 1px solid #e1e1e1;
    border-bottom: 1px solid #e1e1e1;
 }
 
.combined-search-page .searchpane {background: #444;}
 
.combined-search-page #search input[type="text"] {max-width: none;color: #fff !important}
 
.combined-search-page .search-icon {
    background-size: 30px 30px;
    height: 30px;
    width: 30px;
 }
 
.combined-search-page .search-submit-button {
    background: none;
    border: none;
    float: left;
    margin-top: -45px;
    margin-left: -25px;
 }
 
.combined-search-page .search-result-subreddit .search-subscribe-button .add, .combined-search-page .search-result-subreddit .search-subscribe-button .remove {width: auto;}
 
.combined-search-page .search-result .search-result-body {
    font-size: 1em;
    line-height: 1.25em;
    color: #4F4F4F;
    padding: 10px 0px;
    width: 800px;
 }
 
.combined-search-page .search-result a, .combined-search-page .search-result a > mark {color: inherit;}
 
.combined-search-page .search-result .search-result-meta, .combined-search-page .search-result .search-score {
    font-size: 10px;
    line-height: 2em;
    color: #808080;
    padding: 8px 0px 0px 0px;
    font-weight: bold;
    text-transform: uppercase;
 }
 
#moresearchinfo {
    border: none;
    background: rgba(0, 0, 0, 0.9);
    border-radius: 0 0 10px 10px;
    line-height: 1.7em;
    padding-left: 20px;
    padding-right: 10px;
    padding-bottom: 10px;
    text-align: left;
 }
 
.search-page div.content {
    margin-top: 100px;
    margin-left: auto;
    margin-bottom: 300px;
    margin-right: auto;
    width: 80%;
    min-width: 1000px;
 }
 
.search-page #noresults {
    padding-bottom: 20px !important;
    margin-left: 15px !important;
 }
 
.search-page .side {display: none;}
 
.search-page .menuarea {
    border-bottom: none;
    padding: 5px 10px;
    margin: 5px;
    font-size: 12px;
    font-weight: bold;
    text-transform: uppercase;
    margin-top: -45px;
    position: absolute;
    top: 418px;
 }
 
.search-page div#moresearchinfo {
    background: rgba(0, 0, 0, 0.85);
    position: absolute;
    z-index: 9999;
    padding-left: 22px;
    padding-bottom: 20px;
    color: white;
    padding-right: 20px;
    border-radius: 0 0 8px 8px;
    top: 65px;
 }
 
.search-page a#search_showmore {
    position: absolute;
    top: 100px;
    font-size: 12px;
    text-transform: uppercase;
    color: #999;
    font-weight: bold;
    right: 13px;
 }
 
.search-page .searchfacets {
    border: none;
    box-shadow: none;
    background: #E0E0E0;
    color: #434343;
    text-transform: uppercase;
    font-size: 12px;
    padding-left: 20px;
 }
 
.searchfacet a {color: #434343 !important;}
 
#previoussearch label {
    display: block;
    margin: 5px 0px;
    position: absolute;
    top: 95px;
    text-transform: uppercase;
    font-size: 12px;
    font-weight: bold;
 }
 
.search-page .searchpane {
    margin: auto;
    height: 50px;
    border: none;
    border-radius: 8px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.24);
    position: relative;
    top: -85px;
    margin-bottom: -45px;
 }
 
.search-page #search input[type="submit"], .search-page #search input[type="submit"]:hover {display: none;}
 
.dropdown.lightdrop .selected {text-decoration: none;}
 
.search-page #search input[type="text"] {
    background: none;
    border: medium none;
    font-size: 27px;
    width: 100% !important;
    position: relative;
    top: 2px;
    margin-left: 30px;
    outline: none;
 }
 
.search-page .raisedbox h4 {display: none;}
 
.search-summary {display: none}
 
.searchpane-toggle-hide {display: none;}
 
/*COMMENTS PAGE*/
 
.tagline .score {font-weight: bold;}
 
/*COMMENT EXPANDOS*/
.comment.noncollapsed .midcol {margin-left: 20px;}
.res .comment.noncollapsed .midcol {margin-left: 10px;}
 
.commentarea .comment a.expand
{
    position:absolute;
    top:0;
    left:0;
    padding:4px 4px 0;
    height:100%;
    opacity:.5;
    box-sizing: border-box;
    transition: 200ms;
}
 
.commentarea .comment a.expand:hover
{
    opacity:1;
    color: white;
    border-radius: 4px;
    text-decoration: none;
    font-weight: bold;
}
 
.res .commentarea .comment a.expand:hover {border-radius: 4px 0px 0px 4px;}
 
.commentarea .thing,.midcol
{
    position:relative
}
.comments-page .content .infobar {
    border: 0px none;
    background: #fff;
    max-width: 300px;
    padding: 10px 20px;
    text-transform: uppercase;
    font-weight: bold;
    font-size: 11px;
    border-radius: 4px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.24);
    margin-top: 30px;
    margin-left: -10px !important;
    margin-bottom: 15px;
 }
 
.panestack-title a.title-button {
    position: relative;
    z-index: 2;
 }
 
    .panestack-title a.title-button.gold, .panestack-title a.title-button {
        padding: 5px 10px;
        text-transform: uppercase;
        font-weight: bold;
        font-size: 9px;
        color: #fff;
        border: none;
        border-radius: 3px;
     }
 
 
 
 
.res .comments-page .side {margin: 275px 16px 100px;}
 
 .comments-page .side {margin: 260px 16px 100px;}
 
.res .content .RESBigEditorPop {
    line-height: inherit;
    font-size: 11px;
    box-shadow: none;
    color: #666;
    border: none;
    padding: 0;
    margin-top: 3px;
    background-color: transparent;
    background-image: none;
    margin-left: -12px;
 }
 
.RESSubscriptionButton {
    display: inline-block;
    position: relative;
    z-index: 1;
    margin-left: 15px;
    padding: 5px !important;
    font-size: 9px;
    text-transform: uppercase;
    text-align: center;
    cursor: pointer;
    color: #fff!important;
    border: 0 solid #CCC!important;
    border-radius: 2px;
    transition: .25s ease
 }
 
body.comments-page div.content {background: none !important;}
 
body.comments-page .link {
    background: #FFF;
    border: none;
    border-radius: 5px 5px 0px 0px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
 }
 
body.comments-page .sitetable {
    position: relative;
 }
 
.commentarea > .usertext {
    background-color: #F8F8F8;
    border-radius: 0px 0px 2px 2px;
    box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.08);
    overflow: visible;
    padding-top: 165px;
    padding-left: 29px;
    margin-top: -160px;
    margin-left: -10px;
    margin-right: -10px;
 }
 
.res .commentarea > .usertext {
    position: relative;
    top: -110px;
    margin-bottom: -90px;
    margin-top: -160px;
    padding-top: 136px;
    clear: left;
 }
 
.commentarea .panestack-title {
    margin: 16px 0 -16px 22px;
    border-bottom: 0
 }
 
.commentarea span.title {
    text-transform: uppercase !important;
    font-weight: bold;
    font-size: 13px;
    position: relative;
    z-index: 1;
 }
 
.commentarea .panestack-title .title {color: #4D5763}
 
.commentarea .menuarea {
    margin-left: 22px !important;
    text-transform: uppercase;
    font-size: 12px;
    position: relative;
    z-index: 1;
    overflow: visible;
 }
 
.commentarea .thing {
    border-radius: 2px;
    background-color: #fff;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
    margin: 0 0 8px;
    padding: 16px 16px 1px 16px;
 }
 
.res .commentarea .thing {padding:15px 15px 0px !important}
 
.commentarea .thing.collapsed {padding: 10px 15px 10px !important;}
 
.res .commentarea .entry .flat-list {padding-bottom: 8px;}
 
.comment .flat-list li a[onclick*="reply"] {font-weight: bold;}
 
.commentarea .child .thing {
    box-shadow: none;
    padding: 4px;
    margin-left: 2px !important;
    margin-top: 15px;
    border: 1px solid #fff !important;
 }
 
body.comments-page div.content {border: none !important;}
 
div.sitetable.nestedlisting, div.child {box-shadow: none;}
 
/*END COMMENT AREA*/
/*WIKI PAGE*/
.wiki-page div.content {
    border: 0px;
    background: none;
    margin-right: 335px !important;
 }
 
.wiki-page-content .md h1, .wiki-page-content .md h6 {text-transform: none !important;}
 
.wiki-page .wikititle {
    background-color: #fff;
    padding: 12px 16px 10px;
    border-radius: 5px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
    font-size: 12px;
    text-transform: uppercase;
    margin: 0;
    color: #444;
 }
 
.wiki-page .wiki-page-content .md.wiki h4 {
    padding: 15px 0px;
    font-weight: bold;
 }
 
.wiki-page .pageactions {
    background-color: #fff;
    margin-left: 16px;
    border-radius: 5px;
    padding: 8px 16px;
    font-size: 12px;
    border: none;
    text-transform: uppercase;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08)
 }
 
    .wiki-page .pageactions .wikiaction {
        margin: 0;
        background-color: transparent;
        border-radius: 0;
        color: #B3B3B3;
     }
 
    .wiki-page .pageactions .wikiaction-current {
        color: #444;
        font-weight: bold;
     }
 
    .wiki-page .pageactions .wikiaction:hover,.wiki-page .pageactions .wikiaction-current:hover {
        background: none;
        color: #666;
        font-weight: bold;
     }
 
.wiki-page .wiki-page-content {
    margin: 16px 0px 16px 0;
    background-color: #fff;
    padding: 16px;
    border-radius: 2px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08)
 }
 
    .wiki-page .wiki-page-content .wiki>.toc>ul {border: none}
 
    .wiki-page .wiki-page-content .wiki.md {color: #4D5763;}
 
        .wiki-page .wiki-page-content .wiki.md h2 {color: #444;}
 
        .wiki-page .wiki-page-content .wiki.md p {
            font-size: 14px;
            line-height: 1.4285714285714em
         }
 
    .wiki-page .wiki-page-content hr {
        border-style: solid;
        border-color: #e5e5e5
     }
 
/*SUBMIT PAGE*/
.submit-page .submit_text {width: 640px !important;}
 
    .submit-page .submit_text.enabled {text-transform: none !important;}
 
.formtabs-content .infobar {
    font-size: 15px;
    font-weight: bold;
    border: none;
    background-color: white;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
    padding: 20px;
    border-radius: 4px;
    text-align: center;
    color: #000;
 }
 
#sr-more-link {background-color: #fff !important;}
 
.roundfield textarea, .roundfield input[type="text"], .roundfield input[type="url"], .roundfield input[type="password"], .roundfield input[type="number"], .roundfield .usertext-edit,.submit-page .markdownEditor-wrapper {width: 100%; outline: none;}
 
.roundfield .title {color: #666;}
 
.roundfield {
    background: #FFF;
    width: auto;
    padding: 15px 20px;
    text-transform: uppercase;
    font-weight: bold;
    font-size: 12px;
    line-height: 1.5em;
    border-radius: 4px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
 }
 
 
 
    .roundfield input[type="text"] {margin-bottom: 15px;}
 
 
.content.submit .info-notice {
    margin-bottom: 12px;
    margin-left: 10px;
    margin-top: -10px;
    margin-right: 10px;
    font-size: 12px;
    padding: 9px;
    background-color: #FFF;
    border: none;
 }
 
.submit-page .content .spacer {margin-left: 0px;}
 
.submit-page .content {
    width: 700px;
    margin: 0 auto;
 }
 
.submit-page div.content {
    margin-top: 20px !important;
    margin-bottom: 50px !important;
    background: none !important;
    border: none !important;
 }
 
.submit-page .tabmenu.formtab .selected a {
    color: #cecece;
    background-color: transparent;
    border: none;
    font-size: inherit;
 }
 
.submit-page ul.tabmenu.formtab {
    display: block;
    padding-left: 10px;
    font-size: larger;
    border-radius: 4px;
 }
 
.submit-page .tabmenu li a {
    background: transparent;
    border: none;
 }
 
.submit-page .tabmenu {margin-top: 0px;}
 
.formtabs-content {border-top: 0px solid!important;}
 
body:not(.submit-page) .tabmenu::after {display: none;}
 
.submit-page #newlink.submit.content .btn {
    height: 42px;
    line-height: 41px;
    width: 97%;
    margin-left: 10px;
 }
 
/*Sidebar*/
.sidecontentbox .title h1 {
    color: #555;
    font-weight: bold;
    font-size: 14px;
 }
 
.md-container {text-align: left;}
 
.titlebox {text-align: center;}
 
.submit-page .side, .submit-page .content h1 {display: none !important;}
 
.formtabs-content {border-top: 0px solid!important;}
 
body:not(.submit-page) .tabmenu::after {
    position: absolute;
    height: 49px;
    content: '';
    z-index: -1;
    left: 0px;
    right: 0px;
    background: #666;
 }
 
.formtabs-content {
    width: auto;
    border-top: 4px solid #5f99cf;
    padding: 10px;
    margin-top: 15px;
 }
 
/*No comment message*/
.comments-page #noresults:after {content: ".";}
 
/* --- Message the mod button --- */
.sidecontentbox .title h1 {display: inline-block;}
 
.sidecontentbox a.helplink {
    margin-right: 30px;
    margin-bottom: 20px;
    margin-top: 10px;
    width: 215px;
    border: none;
    border-radius: 4px;
    color: #fff!important;
    text-align: center;
    font-weight: 700;
    font-size: 13px;
    text-transform: uppercase;
    line-height: 30px;
    transition: .25s ease
 }
 
    .sidecontentbox a.helplink:hover {text-decoration: none;}
 
    .sidecontentbox a.helplink:active {background-color: #ae9df4}
 
/*OP TAG*/
 
div .thing .tagline .author.submitter:before,
div .thing .tagline .author.submitter {
    font-weight: Bold;
    line-height: 20px;
    color: #FFF;
    display: inline-block;
    font-size: 10px;
}
 
div .thing .tagline .author.submitter:before {
    background: RoyalBlue;
    padding: 0 7px 0 10px;
    border-radius: 20px 0px 0px 20px;
    font-size: 10px;
    content: "OP ";
    margin-right: 7px;
 }
 
div .thing .tagline .author.submitter {
    background: #27408b !important;
    padding: 0px 10px 0px 0px;
    border-radius: 20px;
    transition: 300ms;
 }
 
 div .thing .tagline .author.submitter:hover {background: RoyalBlue !important; text-decoration: none;}
 
/*MODERATOR DISTINGUISH TAG*/
 
div .thing .tagline .author.moderator:before,
div .thing .tagline .author.moderator,
div .thing .tagline .author.admin:before,
div .thing .tagline .author.admin {
    font-weight: Bold;
    line-height: 20px;
    color: #FFF;
    display: inline-block;
    font-size: 10px;
}
 
div.thing .tagline .author.moderator:before,
div.thing .tagline .author.admin:before {
    padding: 0 7px 0 10px;
    border-radius: 20px 0px 0px 20px;
    font-size: 10px;
    margin-right: 7px;
}
 
body div .thing .tagline .author.moderator,
body div .thing .tagline .author.admin {
    padding: 0px 10px 0px 0px;
    border-radius: 20px;
    transition: 300ms;
}
 
div.thing .tagline .author.moderator:before {background: #282 !important; content: "Mod ";}
 
body div .thing .tagline .author.moderator {background: #1A4B17 !important;}
 
body div .thing .tagline .author.moderator:hover {background: #282 !important; text-decoration: none;}
 
.tagline .moderator:after {content: "oderator - speaking officially.";}
 
a.author.moderator:after {content: "";}
 
/*REDDIT ADMIN DISTINGUISH*/
 
div.thing .tagline .author.admin:before {background-color: #fe506e !important; content: "Admin ";}
 
body div .thing .tagline .author.admin {background: #c41959 !important;}
 
body div .thing .tagline .author.admin:hover {background: #fe506e !important; text-decoration: none;}
 
.tagline .admin:after {content: "dmin - speaking officially";}
 
a.author.admin:after {content: "";}
 
a.author.admin:before {content: "";}
 
a.author.moderator:after {content: "";}
 
/*Body and post appearance*/
.link {
    margin-bottom: 0px;
 }
 
body:not(.comments-page) div.content .entry {
    border-bottom: 1px solid #e1e1e1;
    padding: 13px;
}
 
.comments-page .link {padding: 20px 10px;}
 
/*Image sidebar sub title*/
.res h1.redditname {overflow: visible;}
 
/* RES Selection */
.res .RES-keyNav-activeElement {
    outline: 0px dashed #EEEEEE !important;
    transition: all 0.15s ease;
}
 
.res .entry {transition: all 0.10s ease;}
 
.RES-keyNav-activeElement, .RES-keyNav-activeElement .md-container {
    padding-left: 15px !important;
    border-left: 3px solid;
}
 
div.commentarea div.RES-keyNav-activeElement .md {
    background-color: transparent !important;
}
 
/*Body background*/
body {background: #e6e6e8;}
 
/*Expando no outline*/
.entry.RES-keyNav-activeElement div.md, .commentarea .entry.RES-keyNav-activeElement div.noncollapsed, .entry.RES-keyNav-activeElement .usertext-body, .new-comment .usertext-body .md {
    outline: medium none !important;
    border: 0px none !important;
 }
 
/*RES Never Ending Reddit*/
.NERPageMarker, #progressIndicator {
    font-size: 14px !important;
    font-weight: bold !important;
    text-align: left !important;
    background-color: transparent !important;
    border: none !important;
    border-radius: 0 !important;
    padding: 10px !important;
    text-transform: uppercase;
    margin-left: 20px;
 }
 
#progressIndicator {
    font-size: 12px !important;
    text-align: center;
    text-transform: uppercase;
    color: #666;
 }
 
    #progressIndicator h2 {
        text-align: center;
        font-size: 16px !important;
        text-transform: uppercase;
        font-weight: bold;
        margin-top: 5px;
     }
 
/*Sub Selector*/
#sr-header-area {
    background-color: rgba(0, 0, 0, 0.5);
    border: none;
    color: #fff;
    padding-top: 3px;
    padding-bottom: 3px;
    font-weight: 400;
    font-size: 9px;
 }
 
/*
 
Subreddit list dropdown
Code derived from /r/Apicem
Thanks to /u/Karma_4_Free
 
*/
 
 
        #sr-header-area .drop-choices.srdrop,
        .res #srList,
        .res #RESSubredditGroupDropdown {
          top: 30px!important;
          margin-left: 10px;
          border: 0;
          padding: 10px;
          text-transform: uppercase;
          letter-spacing: .5px;
          font-size: 9px
          border-radius: 6px;
        }
 
        .res #srList {background: rgba(0, 0, 0, 0.75);}        
        #sr-header-area .drop-choices.srdrop { top: 21px!important; background: rgba(0, 0, 0, 0.9); }
       
        .res #srList {
          display: block!important;
          background: transparent;
          position: fixed;
          left: 50%;
          margin-left: -250px;
          width: 500px;
          height: calc(100vh - 60px);
          padding-top: 0;
          overflow: visible;
          max-height: none!important;
          opacity: 1;
          pointer-events: auto;
          transition: .2s;
          z-index: 2147483647
        }
       
        .res #srList:not([style*="display: block"]),
        .res #srList:not([style*="display: block"]):before,
        .res #srList:not([style*="display: block"]):after {
          opacity: 0;
          pointer-events: none
        }
       
        .res #srList tbody { display: block; max-height: calc(100% - 50px); overflow: scroll; background: transparent; }
       
        .res #srList:before {
          position: fixed;
          left: 50%;
          margin-left: -250px;
          top: 30px;
          width: 500px;
          height: calc(100vh - 60px);
          background: rgba(0, 0, 0, 0.8);
          content: "";
          z-index: -1;
          transition: .2s opacity;
          border-radius: 4px;
        }
       
        .res #srList:after {
          position: fixed;
          left: 0;
          top: 0;
          width: 100vw;
          height: 100vh;
          padding: 50px 70px 0 0;
          background: rgba(0,0,0,.6);
          background-size: 100px 100px, cover, cover;
          background-position: center, center, center;
          content: "✕";
          text-align: right;
          text-transform: lowercase;
          font-size: 40px;
          font-weight: 100;
          color: #fff;
          z-index: -2;
          transition: .2s opacity;
          pointer-events: none;
          margin-left: -35px; /*Adjusts the close icon*/
        }
       
        .sortAsc, .sortDesc { margin-top: -46px; margin-right: 10px }
       
        #sr-header-area .drop-choices.srdrop a,
        .res #srList tr,
        .res #RESSubredditGroupDropdown li a {
          height: 25px;
          padding: 0 10px;
          border-top: solid 1px #444;
          border-bottom: 0;
          line-height: 25px;
          color: #ccc
        }
       
        .res #srList thead {
          display: block;
          height: 50px;
        }
       
        #sr-header-area .drop-choices.srdrop:before,
        .res #srList thead td:nth-of-type(1) {
          height: 50px;
          line-height: 45px;
          font-size: 13px;
          font-weight: bold;
          color: #fff;
          content: "SUBSCRIBED SUBREDDITS"
        }
       
        #sr-header-area .drop-choices.srdrop:before { padding: 0 50px 0 10px }
       
        .res #srList thead td:nth-of-type(1):before { content: "SUBSCRIBED " }
        .res #srList thead td:nth-of-type(1):after { margin-left: -5px; content: "S" }
       
        .res #srList thead tr {
          height: 50px;
          line-height: 50px;
          white-space: nowrap
        }
       
        .res #srList td { padding: 0 }
        .res #srList tr td:nth-of-type(1) { width: 300px }
        .res #srList td.RESvisited { width: 120px }
        .res #srList td.RESshortcut { width: 60px }
       
        .res #srList td.RESvisited,
        .res #srList td.RESshortcut { text-transform: uppercase }
       
        .res #srList a,
        .res #RESSubredditGroupDropdown .RESShortcutsEditButtons .res-icon { color: #fff }
       
        #sr-header-area .drop-choices.srdrop a,
        .res #RESSubredditGroupDropdown li a { display: block; min-width: 50% }
       
        .res #srList thead tr,
        .res #RESSubredditGroupDropdown li:nth-of-type(1) a { border: 0 }
       
        .res #RESSubredditGroupDropdown li + .RESShortcutsEditButtons { margin: 0; border: 0; padding: 0 }
        .res #RESSubredditGroupDropdown li + .RESShortcutsEditButtons a.delete { text-align: right }
       
        .res #RESSubredditGroupDropdown li { margin: 0; padding: 0 }
       
        .res #srList tr:hover,
        .res #RESSubredditGroupDropdown li:hover { background: none }
       
 
 
body.res #sr-header-area .sr-bar a.RESShortcutsCurrentSub {color: #fff!important; /*Color of sub selector, active*/}
 
#sr-header-area a {font-size: 9px; transition: 200ms;}
 
    #sr-header-area a:hover {font-weight: bold; color: #fff !important}
 
.sr-bar a {color: #fff;}
 
div.score {
    font-weight: 500;
    color: #999;
 }
 
.md p, .md h1 {line-height: 20px;}
 
.md ul {line-height: 20px;}
 
h2 {
    color: #252525;
    font-size: 18px;
    font-weight: normal;
 }
 
 /*Linkinfo*/
 
.side div.score, span.totalvotes {
    text-transform: uppercase;
    font-size: 14px;
    font-weight: 500 !important;
    color: #444;
    padding: 0px 16px;
}
 .linkinfo .score .number {
    font-size: 14px;
    text-align: left;
 }
 
 .linkinfo .totalvotes {font-size: 100%;}
 
.linkinfo {
    border: none !important;
    background-color: #fff;
    top: 375px;
    font-family: inherit !important;
    padding: 0px;
    position: absolute;
    box-shadow: 0 1px 5px rgba(0,0,0,0.08);
    border-radius: 4px;
    width: 300px;
 }
 
 .linkinfo .date {
    font-weight: bold;
    font-size: 11px;
    text-transform: uppercase;
    color: #666;
    text-align: left;
    padding: 14px 16px 0px;
}
 
.linkinfo .shortlink input {
    box-shadow: none !important;
    font-family: monospace !important;
    transition: all 0.15s ease 0s !important;
    font-size: 13px;
    color: white;
    border: medium none;
    width: 280px;
    background: #444;
    border-radius: 0px 0px 4px 4px;
    padding: 10px;
    text-align: center;
}
 
.linkinfo .shortlink {
    text-transform: uppercase;
    margin-top: 20px;
    font-weight: bold;
    text-align: center;
    font-size: 0px;
}
 
 
.comment .collapsed {
    padding-left: 24px;
    padding-bottom: 10px;
    padding-top: 2px;
 }
 
.comment .midcol {width: 20px !important;}
 
.gadget .midcol {width: 20px !important;}
 
/*Sub Selector*/
/*Header*/
#header {
    background: #444;
    border: none;
    height: 225px;
    transition: 300ms;
    -webkit-transition: 300ms;
 }
 
#header span.pagename.redditname a{
    -webkit-animation-duration: 1.2s;
    -webkit-animation-name: godown;
    animation-duration: 1.2s;
    animation-name: godown;
}
@keyframes godown {
0% {opacity:0;transform:translateY(-60px);-webkit-transform:translateY(-60px)}
100% {opacity:1;transform:translateY(0px);-webkit-transform:translateY(0px)}
}
 
@-moz-keyframes godown {
0% {opacity:0;transform:translateY(-60px);-webkit-transform:translateY(-60px)}
100% {opacity:1;transform:translateY(0px);-webkit-transform:translateY(0px)}
}
 
@-webkit-keyframes godown {
0% {opacity:0;transform:translateY(-60px);-webkit-transform:translateY(-60px)}
100% {opacity:1;transform:translateY(0px);-webkit-transform:translateY(0px)}
}
 
@-o-keyframes godown {
0% {opacity:0;transform:translateY(-60px);-webkit-transform:translateY(-60px)}
100% {opacity:1;transform:translateY(0px);-webkit-transform:translateY(0px)}
}
 
.pagename a {
    height: 100px;
    width: 200px;
    display: inline-block;
    text-indent: -99999px;
    background-repeat: no-repeat;
 }
    span.pagename.redditname a:hover {opacity: 0.8}
 
.pagename {font-size: 0px;}
 
.subtitle, .sidebox .spacer, .sidebox .spacer a, .sidebox.create, .morelink .nub {display: none;}
 
/*Disable CSS style toggle*/
.res .titlebox>div:first-of-type, .titlebox form.toggle {
    font-weight: normal;
    text-align: center;
    padding-top: 5px;
    padding-bottom: 5px;
 }
 
/*Subscribe+Sidebar Toggles*/
 
.fancy-toggle-button .add,
.fancy-toggle-button .remove {
    padding: 5px 20px;
    border-radius: 20px;
}
 
.res .fancy-toggle-button .remove{padding: 5px 8px !important;}
 
.res .fancy-toggle-button .add,
.res .fancy-toggle-button .remove,
.fancy-toggle-button .add,
.fancy-toggle-button .remove,
.RESshortcutside,
.RESDashboardToggle,
.RESshortcutside.remove,
.RESDashboardToggle.remove {
    font-size: 9px !important;
    border: none !important;
    text-transform: uppercase;
    transition: 200ms;
    background-image: none !important;
}
 
.res .fancy-toggle-button .add,
.res .fancy-toggle-button .remove,
.RESshortcutside,
.RESDashboardToggle,
.RESshortcutside.remove,
.RESDashboardToggle.remove {
    padding: 5px 8px !important;
    border-radius:3px !important;
}
 
/*End sidebar toggles*/
 
/* reddit name on sidebar below search box */
.titlebox h1.redditname {
    width: 100%;
    text-align: center;
 }
 
.titlebox .redditname {height: 25px}
 
.res .titlebox .fancy-toggle-button {
    display: inline-block;
    margin-top: 10px;
    margin-right: 5px;
    margin-left: 0px;
    width: auto;
 }
 
.titlebox .fancy-toggle-button {
    margin: 0px auto;
    width: 100%;
 }
 
.titlebox .fancy-toggle-button {
    display: inline-block;
    margin-top: 4px;
    transition: 300ms;
 }
 
.flat-list {
    list-style-type: none;
    display: inline;
 }
 
/*COMMENTS BUTTON COLOR*/
.entry .buttons .may-blank {
    font-weight: bold;
 }
 
.link .tagline {
    color: #666;
    font-size: 12px;
    margin-top: 3px;
 }
 
.entry .buttons li a, .entry .buttons li + li {color: #666;}
 
 
.comment .flat-list li a:hover,
.link .entry .buttons li a:hover,
.link .domain a:hover {
    font-weight: bold;
    text-decoration: none;
    transition: 200ms;
 }
 
/*COMMENTS AND EVERYTHING ELSE BUTTONS*/
button {
    border-radius: 4px;
    color: #FFF;
    cursor: pointer;
    font-size: 14px;
    line-height: 2.16;
    text-align: center;
    white-space: nowrap;
    border: 0px none;
    padding: 0px 1.06667em;
    text-transform: uppercase;
    transition: all 200ms ease 0s;
    font-weight: bold;
    font-size: 12px;
}
 
/*BODY MARGINS AND SETTINGS */
body:not(.submit-page) .tabmenu::after {
    position: absolute;
    height: 49px;
    content: '';
    z-index: -1;
    left: 0px;
    right: 0px;
    background: #666; /*TABMENU BACKGROUND SELECT*/
 }
 
/* TAB BAR SETTINGS */
#sr-header-area {
    width: 100%;
    z-index: 9999;
 }
 
/* BODY MARGINS */
div.content {
    border-radius: 4px;
    margin: 15px 335px 10px 16px;
    border: 1px solid #e1e1e1;
    background-color: #fff;
    font-size: 12px;
 }
 
.midcol {margin-left: 0;}
 
body >.content .link .rank, .rank-spacer {display: none;}
 
body > .content .link .midcol, .midcol-spacer { width: 40px !important;}
 
body:not(.comments-page) > .content .link .midcol, .midcol-spacer {margin: 10px 0px;}
 
 
.link .title {font-weight: normal;}
 
    .link .title .domain {
        color: #999;
        font-size: 0;
        font-weight: normal
     }
 
        .link .title .domain::before {font-size: 12px}
 
        .link .title .domain a {
            font-size: 12px;
            margin-left: 1px;
            margin-top: -10px;
         }
 
 
/*SIDEBAR*/
.side {
    border: 1px solid #e1e1e1;
    border-radius: 5px;
    padding: 0px;
    margin: 150px 16px 100px;
    width: 300px;
    background: #ffffff;
 }
 
#searchexpando {
    border-radius: 0px 0px 4px 4px;
    text-align: center;
    width: 280px;
    background: rgba(0, 0, 0, 0.85);
    border: medium none;
    box-shadow: 0px 10px 10px rgba(0, 0, 0, 0.3);
    color: white;
    margin-top: 0px;
    font-weight: bold;
    text-transform: uppercase;
    font-size: 11px;
    line-height: 2.5em;
    height: auto;
 }
 
.infobar {border: 0px}
 
.side #search {
    top: 240px;
    position: absolute;
    margin-right: 25px;
    z-index: 4;
    width: 317px;
 }
 
/*SEARCH ANIMATION*/
 
body:not(.search-page) #search input[type=text] {
    border-radius: 6px;
    padding:9px;
    border: 1px solid #e1e1e1;
    text-align: center;
    transition: 300ms;
 }
    body:not(.search-page) #search input[type=text]:hover {border: 1px solid #a0a0a0;}
 
    body:not(.search-page) #search input[type=text]:focus {
        border-color: #aaa;
        text-align: left;
        padding-left: 8px;
        outline: 0;
        padding-left: 6px;
        animation-name: search;
        animation-duration: 0.4s;
        animation-timing-function: ease;
        animation-iteration-count: 1;
        animation-play-state: running;
     }
@-webkit-keyframes search {
    0% {padding-left: 125px;}
    8% {padding-left: 107px;}
    15% {padding-left: 89px;}
    30% {padding-left: 71px;}
    50% {padding-left: 53px;}
    60% {padding-left: 18px;}
    100% {padding-left: 6px;}
}
@keyframes search {
    0% {padding-left: 125px;}
    8% {padding-left: 107px;}
    15% {padding-left: 89px;}
    30% {padding-left: 71px;}
    50% {padding-left: 53px;}
    60% {padding-left: 18px;}
    100% {padding-left: 6px;}
}
 
/*SUBMIT LINKS */
.morelink {
    border: none;
    letter-spacing: 0;
    font-size: 14px !important;
    border-radius: 3.5px;
    padding: 3px;
    text-transform: uppercase;
    transition:200ms;
 }
 
    .morelink a, .morelink a:hover, .morelink a:visited {color: #fff;}
 
/*RESTRICTED*/
.disabled .morelink, .disabled .morelink:hover {
    background-image: none !important;
    top: 314px;
    position: absolute;
    width: 300px;
 }
 
.side .submit-link {
    position: absolute;
    width: 300px;
    top: 285px;
    right: 16px;
 }
 
.side .submit-text {
    position: absolute;
    width: 300px;
    top: 330px;
    right: 16px;
 }
 
.morelink .nub {display: none;}
 
/*TITLEBOX PADDING*/
.titlebox {padding: 12px;}
 
.side .titlebox .md h3 a {margin-left: 5px;}
 
.side md {
    padding-left: 5px;
    padding-right: 5px;
 }
 
.side #moderation_tools {
    padding-left: 15px;
    margin-bottom: 15px;
 }
 
.sidecontentbox {padding: 15px;}
 
.infobar {
    background: #f2f2f2;
    border: 1px solid #ebebeb;
 }
.message-count {
    position: relative;
    top: -2px;
 }
 
.commentarea {padding: 0 10px 10px;}
 
.users-online {color: #535353;}
 
    .users-online:before {display: none;}
 
.flair, .linkflairlabel {
    margin-right: 0.8em;
    padding: 2px 4px;
 }
 
.nextprev, .next-suggestions {
    margin: 10px !important;
    padding: 10px !important;
    display: block;
 }
 
.nextprev a, .next-suggestions a {font-weight: normal !important;}
 
.link .usertext .md {border: 1px solid #e1e1e1;}
 
.link .usertext-body .md {
    border: none !important;
    background: none !important;
 }
 
 
 /*LINK SCORES TYPE*/
.link .score.likes {color: #FF5F3F;}
.link .score.dislikes {color: #2B9BDB;}
.score.unvoted {color: #666;}
div.midcol.likes, div.midcol.dislikes, div.midcol.unvoted,div.reddit-entry {padding: 0px 10px;}
 
#header-img.default-header:hover, #header-img:hover {opacity: .7;}
 
.users-online {margin-bottom: 2em;}
 
.titlebox span.subscribers,.titlebox .number {
    font-size: 12px;
    position: relative;
 }
 
.titlebox span.subscribers, .titlebox .users-online {
    color: #999;
    position: relative;
    top: 10px;
 }
 
div.leavemoderator {
    margin-left: 18px;
    background: none;
 }
 
.titlebox .users-online {
    font-size: 12px;
    position: relative;
    top: 15px;
 }
 
    .titlebox .users-online:before {display: none}
 
    .titlebox .users-online .number {font-style: italic}
 
        .titlebox .users-online .number:before {content: ""}
 
.titlelebox .word {display: none}
 
.titlebox .users-online,.titlebox .number {cursor: text}
 
 
/* Text */
.side .md .-blocks,.side .md .-lists,.side .md pre,.side .md blockquote,.side .md table,.side .md p,.side .md ul,.side .md ol {margin: 15px 0;}
 
    .md table thead tr th{color: white}
 
/*SIDEBAR STYLING*/
 
.titlebox h1.redditname {font-weight:normal;}
.side .md h1 {
    position: relative;
    margin: 0 20px 20px 20px;
    padding: 5px 0;
    font-size: 18px;
    font-weight: bold;
    text-align: center;
    text-decoration: none;
}
 
/*BOXED SIDEBAR STYLING*/
 
.side .md h2 {
    position: relative;
    color: #fff;
    height: 30px;
    line-height: 30px;
    margin: 0 0px;
    font-size: 13px!important;
    font-weight: bold;
    background: #444;
    text-align: center;
    border-radius: 5px 5px 0 0;
}
 
.side .md h2 a {color: #A4D11B}
 
.side .md h2:after {
    top: 100%;
    left: 5%;
    border: solid transparent;
    content: " ";
    height: 0;
    width: 0;
    position: absolute;
    pointer-events: none;
    border-width: 7px;
    margin-left: -7px; }
 
.side .md h2 + ul, .side .md h2 + ol {
    list-style-position: outside;
    border-radius: 0 0 5px 5px;
    background:#efefef;
    border-top: 0;
    padding: 10px 10px 10px 26px;
    margin: 0 0px 20px 0px;
}
 
 
.side .md h2 + ul li { margin-bottom: 10px; font-size: 95%; }
 
.side .usertext {margin-top: 20px;}
 
/*SIDEBAR BUTTONS COLORS*/
 
.fancy-toggle-button .add,
.RESshortcutside,
.RESDashboardToggle
 
    {background: #80d04b;}
 
/*SIDEBAR BUTTONS HOVER*/
 
.fancy-toggle-button .add:hover,
body.res .RESshortcutside:hover,
.RESDashboardToggle:hover
 
   {background: #A6DE81;}
 
/*SIDEBAR BUTTONS ACTIVE*/
 
.fancy-toggle-button .add:active,
body.res .RESshortcutside:active,
.RESDashboardToggle:active
 
   {background: #599134;}
 
 
/*BUTTONS ON SIDEBAR - Buttonstyle one */
.side .titlebox .md h3 a {
    border-radius: 3.5px;
    padding: 8px 0px;
    border: none;
    color: #fff;
    letter-spacing: 0;
    font-weight: bold;
    text-align: center;
    transition: .3s background ease;
    font-size: 12px !important;
    width: 276px;
    margin: 1.5em 0;
    display: block;
 }
 
/*BUTTONS ON SIDEBAR - Buttonstyle two */
.side .titlebox .md h4 a {
    border-radius: 3.5px;
    padding: 8px 0px;
    border: none;
    color: #fff;
    letter-spacing: 0;
    font-weight: bold;
    text-align: center;
    transition: .3s background ease;
    font-size: 12px !important;
    width: 276px;
    margin: 1.5em 0;
    display: block;
 }
 
.titlebox .bottom {display: none}
 
.sidecontentbox .collapse-button {
    display: inline-block;
    width: 12px;
    height: 12px;
    line-height: 10px;
    text-align: center;
    font-size: 10px;
    margin: 1px 8px;
    background-color: transparent;
    color: RoyalBlue;
    vertical-align: middle;
    border-radius: 2px;
    border: 1px solid RoyalBlue;
 }
 
    .sidecontentbox .collapse-button:hover {
        background-color: transparent;
        color: #009884;
        vertical-align: middle;
        border-radius: 2px;
        border: 1px solid #27408b;
     }
 
.side .sidecontentbox .content {
    border: none;
    padding: 8px 0
 }
 
/* Recently viewed links */
.gadget .midcol {width: 38px}
 
/* Body Content - Frontpage */
.content {
    margin: 96px 16px 0;
    padding: 0
 }
 
.entry .buttons li a, .entry .buttons li + li {font-weight: normal;}
 
/*FLAIRS*/
.flair-redflair[title]:hover {background: #E14169;}
.flair-purpleflair[title]:hover {background:#6941E1;}
.flair-greenflair[title]:hover {background:green;}
.flair-orangeflair[title]:hover {background: #E16941;}
.flair-blueflair[title]:hover {background:RoyalBlue;}
 
.flair-redflair[title],
.flair-purpleflair[title],
.flair-greenflair[title],
.flair-orangeflair[title],
.flair-blueflair[title] {
    color: white;
    font-size: 10px;
    padding: 0px 10px 0px 0px;
    -webkit-transition: 300ms;
    transition: 300ms;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    border-radius: 20px;
}
 
.flair-redflair[title]:before,
.flair-purpleflair[title]:before,
.flair-greenflair[title]:before,
.flair-orangeflair[title]:before,
.flair-blueflair[title]:before {
    font-weight: Bold;
    display: inline-block;
    font-size: 10px;
    padding: 4px 8px;
    border-radius: 20px 0px 0px 20px;
    margin-right: 8px;
}
 
.flair-redflair[title] {background: #9d2d49;}
.flair-purpleflair[title] {background: #492d9d;}
.flair-greenflair[title] {background: #005900;}
.flair-orangeflair[title] {background: #9d492d;}
.flair-blueflair[title] {background: #27408b;}
 
.flair-redflair[title]:before {background: #E14169;}
.flair-purpleflair[title]:before {background: #6941E1;}
.flair-greenflair[title]:before {background: green;}
.flair-orangeflair[title]:before {background: #E16941;}
.flair-blueflair[title]:before {background: RoyalBlue;}
 
 
 
 
 
.titlebox form.toggle {background: none;}
 
.titlebox .tagline {
    text-align: center;
    max-height: 50px;
 }
 
.side .tagline {color: transparent;}
 
/*NO PRESET DEFAULT FLAIR*/
.linkflairlabel {
    background: RoyalBlue;
    color: #FFF;
    font-weight: bold;
    padding: 2px 10px;
    border-radius: 20px;
}
 
/*COLORFUL LINK FLAIRS*/
.linkflairlabel {
    max-width: none;
    border: 0px none;
 }
 
.linkflair-red .linkflairlabel,
.linkflair-purple .linkflairlabel,
.linkflair-orange .linkflairlabel,
.linkflair-green .linkflairlabel,
.linkflair-blue .linkflairlabel {
    color: white;
    font-weight: Bold;
    border-radius: 20px;
    padding: 2px 10px;
    transition: 300ms;
}
 
/*DEFAULT BACKGROUND*/
.linkflair-red .linkflairlabel {background: #fe506e;}
.linkflair-purple .linkflairlabel {background: #6941E1;}
.linkflair-orange .linkflairlabel {background: #E16941;}
.linkflair-green .linkflairlabel {background: #228b22;}
.linkflair-blue .linkflairlabel {background: RoyalBlue;}
 
/*HOVER BACKGROUND*/
.linkflair-red .linkflairlabel:hover {background: #b1384d;}
.linkflair-purple .linkflairlabel:hover {background: #492d9d;}
.linkflair-orange .linkflairlabel:hover {background: #9d492d;}
.linkflair-green .linkflairlabel:hover {background: #176117;}
.linkflair-blue .linkflairlabel:hover {background: #2d499d;}
 
/* Flair Selector */
.flairselector.drop-choices.active {
    border: 1px solid #e1e1e1;
    border-radius: 8px !important;
    visibility: visible;
    background-color: #FFF;
    padding: 20px !important;
 }
 
.flairselector .flairselection {margin-bottom: 17px;}
 
.flairselector .customizer {
    display: inline-block;
    vertical-align: middle;
 }
 
.side .flairselector {
    position: fixed;
    top: 150px!important;
    left: 50%!important;
    margin-left: -220px;
    width: 400px !important;
    border: none;
    border-radius: 2px;
    box-shadow: 0 0 16px rgba(0,0,0,0.64);
 }
 
.flairselector h2 {
    margin-bottom: 4px;
    background-color: transparent;
    color: #555 !important;
    text-align: left;
    text-transform: uppercase !important;
    font-weight: bold;
    font-size: 12px !important;
 }
 
.flairselector form button {
    text-transform: uppercase;
    font-size: 10px;
    font-weight: bold;
 }
 
body.comments-page .sitetable {z-index: 3;}
 
.side .flairselector:before {
    display: block;
    position: fixed;
    top: 0px;
    left: 0px;
    z-index: -2;
    overflow: hidden;
    width: 100%;
    height: 100%;
    background-color: rgba(255, 255, 255, 0.75);
    content: "";
    cursor: default;
    transition: all 0.25s ease;
    pointer-events: none; /*DO NOT REMOVE*/
 }
 
.flairselector.drop-choices.active {
    visibility: visible !important;
    background-color: #fff;
    border: 1px solid #d8d8d8;
    border-radius: 2px;
 }
 
.flairselector h2 {
    padding: 5px 5px 5px 5px;
    background-color: #fff;
    border-bottom: 1px solid #e5e5e5;
    text-transform: capitalize;
 }
 
.flairoptionpane {text-align: left;}
 
    .flairoptionpane ul {visibility: visible !important;}
 
        .flairoptionpane ul a.title {font-size: 100% !important;}
 
.flairselector li {
    padding: 3px 0px 3px 0px;
    width: 182px;
 }
 
    .flairselector li:hover {background-color: #efefef;}
 
.flairselector form {border-top: 1px solid #e5e5e5;}
 
.flairsample-left {text-align: left !important;}
 
.flairselector ul {overflow: visible;}
 
.flairselector li.selected {
    background-color: #e5ebf8;
    border: none;
 }
 
/*LOGIN FORM*/
.login-form-side {
    border: medium none;
    padding: 16px;
    margin-top: -10px;
 }
 
    .login-form-side input[type="text"], .login-form-side input[type="password"] {
        font-size: 11px;
        box-sizing: border-box;
        border: 1px solid #999;
        width: 258px;
     }
 
    .login-form-side #remember-me * {vertical-align: middle;}
 
    .login-form-side .submit {
        float: right;
        margin-top: 5px;
        margin-right: 5px;
     }
 
 
/*THUMBNAILS*/
.thumbnail {
    border-radius: 5px;
    margin-right: 15px !important;
    position: relative;
    top: 33px;
    transform: translateY(-50%);
    -webkit-transform: translateY(-50%);
}
body:not(.comments-page) .thumbnail {margin: 10px 5px 0px 0px !important;}
 
body .stickied .thumbnail.self,
.thumbnail.nsfw,
.thumbnail.self,
.thumbnail.default {
    background-repeat: no-repeat !important;
    padding: 10px 0px;
    border-radius: 15px;
}
 
/*STICKIED*/
.stickied a.title:link,
.stickied a.title:visited,
body:not(.comments-page) .stickied .link .tagline,
body:not(.comments-page) .stickied .tagline,
body:not(.comments-page) .stickied .entry .buttons li a,
body:not(.comments-page) .stickied .entry .buttons li + li {color: #2B8B27 !important;}
 
/*NSFW*/
.over18 a.title:link,
.over18 a.title:visited,
body:not(.comments-page) .over18 .link .tagline,
body:not(.comments-page) .over18 .tagline,
body:not(.comments-page) .over18 .entry .buttons li a,
body:not(.comments-page) .over18 .entry .buttons li + li {color: #D10023!important;}
 
/*UPPER RIGHT RES THINGY*/
.res #RESShortcutsEditContainer, .res #sr-more-link {background: transparent;}
 
#sr-more-link {
    position: absolute;
    top: -3px;
 }
 
#RESShortcutsSort, #RESShortcutsRight, #RESShortcutsLeft, #RESShortcutsAdd, #RESShortcutsTrash {
    height: 25px;
    line-height: 25px!important;
    margin-right: 4px;
    background: transparent!important;
    color: #fff !important;
 }
 
.link .flat-list {
    display: block;
    padding: 1px 0px;
    margin-top: 3px;
 }
 
li.searchfacet {width: 20em}
 
/*LOADING MESSAGE AND ERROR FORMAT*/
.error {
    font-size: 12px;
    text-transform: uppercase;
    font-weight: bold;
 }
 
/*STYLESHEET EDITING PAGE FIX*/
.pretty-form {
    font-size: larger;
    margin-left: 25px;
 }
 
#preview-table > table {
    border-width: 0.2em;
    border-style: dashed;
    border-color: #D3D3D3;
    padding: 5px;
    margin: 5px;
    width: 900px;
    margin-left: 25px;
 }
 
 
form#image-upload.image-upload {
    margin-left: 25px;
    padding: 9px;
    line-height: 24px;
    font-weight: bold;
    text-transform: uppercase;
    font-size: 10px;
    margin-bottom: 20px;
    color: #666;
}
 
/*FOOTER*/
.debuginfo, .footer-parent {background-color: #222;}
 
.footer-parent {
    font-size: 14px;
    padding-top: 20px;
    padding-bottom: 20px;
    margin-top: 300px;
 }
 
.footer .col {margin: 10px 0px 30px 0px;}
 
.footer {border: 0 solid;}
 
.col {border: none !important;}
 
.footer .flat-vert li a {
    font-size: 11px;
    font-weight: bold;
    text-transform: uppercase;
    color: #999 !important;
 }
 
    .footer .flat-vert li a:hover {
        text-decoration: none !important;
        color: #fff !important;
     }
 
.footer .flat-vert .title {
    font-size: 16px;
    font-weight: 700;
    transition: color .25s ease, background-color .25s ease;
    text-transform: uppercase;
    color: white;
 }
 
.bottommenu {
    font-size: 10px;
    color: #FFF !important;
    border: medium none !important;
    text-transform: uppercase;
    font-weight: bold;
 }
 
    .bottommenu a {
        text-decoration: none;
        color: #999;
     }
 
/*NEW MAIL ANIMATION*/
/*TAKEN FROM /R/APPLE*/
#mail.havemail:before {
    position: fixed;
    bottom: 24px;
    z-index: 100;
    padding: 45px 0px 10px 0px;
    background-color: rgba(0, 0, 0, 0.85);
    border-radius: 6px;
    box-shadow: 0 1px 5px rgba(0,0,0,0.5);
    color: #fff;
    width: 210px;
    height: 15px;
    left: 0px;
    right: 0px;
    content: "YOU HAVE NEW MESSAGES";
    text-indent: 30px;
    font-size: 11px;
    line-height: 1;
    -webkit-transform: translateY(112px);
    transform: translateY(112px);
    transition: background-color 0.25s ease, box-shadow 0.25s ease;
    background-image: url(%%havemail%%);
    background-position: 85px 8px;
    margin: 0px auto;
    background-size: 32px;
    font-weight: bold;
    background-repeat: no-repeat;
 }
 
    /*Animation */
        #mail.havemail:before {
            animation-name: toast;
            animation-duration: 8s;
            animation-iteration-count: 1;
            animation-timing-function: ease;
 
            -webkit-animation-name: toast;
            -webkit-animation-duration: 8s;
            -webkit-animation-iteration-count: 1;
            -webkit-animation-timing-function: ease;
        }
 
            #mail.havemail:hover:before {
                -webkit-animation-play-state: paused;
                 -moz-animation-play-state: paused;
                 -o-animation-play-state: paused;
                  animation-play-state: paused;
            }
 
 
            @-webkit-keyframes toast {
              0%    {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              20%   {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              25%   {transform:translateY(-8px);    -webkit-transform:translateY(-8px); opacity: 0.8;}
              27%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              92%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              97%   {transform:translateY(16px);    -webkit-transform:translateY(16px); opacity: 0.8;}
              100%  {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
            }
 
            @-o-keyframes toast {
              0%    {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              20%   {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              25%   {transform:translateY(-8px);    -webkit-transform:translateY(-8px); opacity: 0.8;}
              27%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              92%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              97%   {transform:translateY(16px);    -webkit-transform:translateY(16px); opacity: 0.8;}
              100%  {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
            }
 
 
            @-moz-keyframes toast {
              0%    {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              20%   {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              25%   {transform:translateY(-8px);    -webkit-transform:translateY(-8px); opacity: 0.8;}
              27%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              92%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              97%   {transform:translateY(16px);    -webkit-transform:translateY(16px); opacity: 0.8;}
              100%  {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
            }
 
 
            @keyframes toast {
              0%    {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              20%   {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
              25%   {transform:translateY(-8px);    -webkit-transform:translateY(-8px); opacity: 0.8;}
              27%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              92%   {transform:translateY(00px);    -webkit-transform:translateY(00px); opacity: 0.8;}
              97%   {transform:translateY(16px);    -webkit-transform:translateY(16px); opacity: 0.8;}
              100%  {transform:translateY(76px);    -webkit-transform:translateY(76px); opacity: 0;}
            }
 
 
/*——————————————————————————————————————————————
 
        SUBREDDIT COLOR SCHEME
 
For aesthetic purposes, please keep everything
of the same set in one color.
 
The text color for the buttons and backgrounds
is white. Please make sure the chosen background
color will provide enough contrast with the text
color.
——————————————————————————————————————————————*/
 
/*SET ONE*/
 
        /*DEFAULT COLOR*/
 
        .RESDashboardToggle.remove,.RESSubscriptionButton,.RESshortcutside.remove,.fancy-toggle-button .remove,.panestack-title a.title-button,.panestack-title a.title-button.gold,.side .titlebox .md h4 a,.sidecontentbox a.helplink,button,.commentarea .comment a.expand:hover
 
        {background:RoyalBlue}
 
        .commentarea span.score, .comment .flat-list li a:hover,.comment .flat-list li a[onclick*=reply],.commentNavSortType:hover,.commentarea .menuarea .toggle a:hover,.entry .buttons .may-blank,.error,.link .domain a:hover,.link .entry .buttons li a:hover,.loadFlat:hover,.submit-page .roundfield a,.thing .title,#progressIndicator a#NERStaticLink,.deepthread a,.deepthread::after,.linefield .title
 
        {color:RoyalBlue}
 
 
        .roundfield input[type=url]:focus,.roundfield input[type=text]:focus,.roundfield textarea:focus,.usertext-edit textarea:focus,#img-name:focus,.roundfield input[type="url"]:focus,.flairlist .flair-jump input[type="text"]:focus, .RES-keyNav-activeElement, .RES-keyNav-activeElement .md-container
 
        {border-color: RoyalBlue}
 
 
        /*HOVER - LIGHTER SHADE OF DEFAULT COLOR*/
 
        .RESDashboardToggle.remove:hover,.comments-page .RESSubscriptionButton:hover,.fancy-toggle-button .remove:hover,.panestack-title a.title-button.gold:hover,.panestack-title a.title-button:hover,.side .titlebox .md h4 a:hover,.sidecontentbox a.helplink:hover,body.res .side .RESshortcutside.remove:hover,button:hover
 
        {background:#6687e7}
 
        /*ACTIVE - DARKER SHADE OF DEFAULT COLOR*/
 
        .RESDashboardToggle.remove:active,.RESshortcutside.remove:active,.comments-page .RESSubscriptionButton:active,.fancy-toggle-button .remove:active,.panestack-title a.title-button.gold:active,.panestack-title a.title-button:active,.side .titlebox .md h4 a:active,.sidecontentbox a.helplink:active,button:active
 
        {background:#27408b}
 
/*SET TWO - SECONDARY COLOR*/
 
        /*DEFAULT*/
 
        .thing .md table thead tr th,.morelink,.side .titlebox .md h3 a,.tabmenu,.wiki-page-content .md table thead tr th {background:#444}
 
        /*HOVER - LIGHTER SHADE OF DEFAULT COLOR*/
 
        .morelink:hover,.side .titlebox .md h3 a:hover {background:#666}
 
        /*ACTIVE - DARKER SHADE OF DEFAULT COLOR*/
 
        .morelink:active, .side .titlebox .md h3 a:active {background: #222;}
 
/*END CHANGE COLOR SCHEME*/
 
 
/*
  EnTypo v0.2 (http://www.reddit.com/r/EnTypo)
  Modified for Minimaluminiumalism
  Copyright (c) 2015 Timur Kuzhagaliyev (TimboKZ)
*/
 
#siteTable .thing .md blockquote,
.wiki-page-content .md blockquote,
.commentarea .md blockquote,
#siteTable .thing .md ul,
.commentarea .md ul,
#siteTable .thing .md ol,
.commentarea .thing .md ol,
#siteTable .thing .md table,
.commentarea .thing .md table {
    font-family: Arial, Helvetica, sans-serif;
    margin: 20px 10px 10px 10px;
}
#siteTable .thing .md blockquote,
.wiki-page-content .md blockquote,
.commentarea .md blockquote {
    padding: 10px 10px;
    border-left: solid 2px #666;
    background-color: #f2f2f2;
    color: #2f2f2f;
    line-height: 20px;
    font-size: 14px;
}
#siteTable .thing .md ul,
.commentarea .md ul,
.commentarea .thing .md ol,
#siteTable .thing .md ol {
    border-right: solid 1px #eaeaea;
    border-left: solid 1px #eaeaea;
    border-top: solid 1px #eaeaea;
    line-height: 20px;
    font-size: 14px;
    padding: 0;
}
#siteTable .thing .md ul li,
.commentarea .md ul li,
.commentarea .thing .md ol li,
#siteTable .thing .md ol li {
    border-bottom: solid 1px #eaeaea;
    background-color: #f8f8f8;
    padding: 5px;
}
#siteTable .thing .md ul li:nth-child(odd),
.commentarea .md ul:nth-child(odd),
.commentarea .thing .md ol li:nth-child(odd),
#siteTable .thing .md ol li:nth-child(odd) {
    background-color: #f2f2f2;
}
#siteTable .thing .md ul,
.commentarea .md ul{
    list-style: none;
}
#siteTable .thing .md ol,
.commentarea .thing .md ol{
    list-style-position: inside;
}
#siteTable .thing .md table thead tr th,
.commentarea .thing .md table thead tr th{
    font-weight: normal;
    padding: 10px 20px;
    font-size: 14px;
    color: white;
    border: none;
}
#siteTable .thing .md table tbody tr,
.commentarea .thing .md table thead tr th {
    border-bottom: solid 1px #eaeaea;
}
#siteTable .thing .md table tbody tr td,
.commentarea .thing .md table tbody tr td {
    padding: 10px 20px;
    border: none;
}
 
.res #RESShortcutsViewport {margin-right: 70px;}
 
/*DARK MODE*/
 
html[lang="dm"] .linkinfo .shortlink input {background: #444;}
 
html[lang="dm"] #siteTable .thing .md blockquote,
html[lang="dm"]  .wiki-page-content .md blockquote,
html[lang="dm"] .commentarea .md blockquote {
    padding: 10px;
    border-left: 2px solid #888;
    background-color: #2f2f2f;
    color: #fff;
    line-height: 20px;
    font-size: 14px;
}
 
html[lang="dm"] #siteTable .thing .md table thead tr th, html[lang="dm"]  .wiki-page-content .md table thead tr th {background: #333;}
 
html[lang="dm"] #siteTable .thing .md table tbody tr,
html[lang="dm"] .commentarea .thing .md table tbody tr,
html[lang="dm"] .wiki-page-content .md table tbody tr {
    border-bottom: 1px solid #999;
}
 
html[lang="dm"] .md hr {color: #888;}
 
html[lang="dm"] #siteTable .thing .md ul li:nth-child(odd),
html[lang="dm"] .commentarea .md ul li:nth-child(odd),
html[lang="dm"] #siteTable .thing .md ol li:nth-child(odd),
html[lang="dm"] .commentarea .thing .md ol li:nth-child(odd),
html[lang="dm"] .wiki-page-content .md ol li:nth-child(odd) {
  background-color: #333;
}
 
html[lang="dm"] #siteTable .thing .md ul li,
html[lang="dm"] .commentarea .md ul li,
html[lang="dm"] #siteTable .thing .md ol li,
html[lang="dm"] .commentarea .thing .md ol li,
html[lang="dm"] .wiki-page-content .md ol li {
  border-bottom: solid 1px #555;
  background-color: #222;
  padding: 5px;
}
 
html[lang="dm"] #siteTable .thing .md ul,
html[lang="dm"] .commentarea .md ul,
html[lang="dm"] #siteTable .thing .md ol,
html[lang="dm"] .commentarea .thing .md ol,
html[lang="dm"] .wiki-page-content .md ol {
  border-right: solid 1px #555;
  border-left: solid 1px #555;
  border-top: solid 1px #555;
  line-height: 20px;
  font-size: 14px;
}
 
 
html[lang="dm"] .content .entry .buttons li.reported-stamp, html[lang="dm"] body .content .stickied .entry .buttons li.reported-stamp  {
 
    background: #222 !important;
    color: #d1d1d1 !important;
 
}
html[lang="dm"] .pretty-button.positive {color: #7DDC1F !important;}
 
html[lang="dm"] .pretty-button.neutral {color: #ccc;}
 
html[lang="dm"] .pretty-button.negative {color:#FE728B !important;}
 
html[lang="dm"] .nsfw-stamp {background: #FE728B;}
 
html[lang="dm"] .combined-search-page .search-result-group .search-header-label,html[lang="dm"] .combined-search-page .search-result .search-result-body,html[lang="dm"] .combined-search-page .search-result .search-result-header .search-title,html[lang="dm"] .combined-search-page .search-result .search-result-footer .search-link,html[lang="dm"] .combined-search-page .search-result .search-result-meta,html[lang="dm"] .combined-search-page .search-result .search-score,html[lang="dm"] .combined-search-page .search-result .search-comments,html[lang="dm"] .combined-search-page .search-result-group .search-result-group-header {color: #fff}
 
html[lang="dm"] .combined-search-page .search-result .search-expando.collapsed::before {background: transparent linear-gradient(to bottom,rgba(255,255,255,0) 0%,#444 100%) repeat scroll 0 0}
 
html[lang="dm"] .combined-search-page .search-result .search-expando-button {color: #A4D11B}
 
html[lang="dm"] .combined-search-page .search-result {border-bottom: 1px solid #666;}
 
html[lang="dm"] .combined-search-page .search-result-group .search-result-group-header {
    background: #222;
    border-bottom: 1px solid #666;
    border-top: 1px solid #666;
}
 
html[lang="dm"] .stickied a.title:link,html[lang="dm"] body:not(.comments-page) .stickied a.title:visited,html[lang="dm"] body:not(.comments-page) .stickied .link .tagline,html[lang="dm"] body:not(.comments-page) .stickied .tagline,html[lang="dm"] body:not(.comments-page) .stickied .entry .buttons li a,html[lang="dm"] body:not(.comments-page) .stickied .entry .buttons li + li,html[lang="dm"] body:not(.comments-page) .stickied .md p {color: #7ddc1f!important}
 
html[lang="dm"] .over18 a.title:link,html[lang="dm"] .over18 a.title:visited,html[lang="dm"] body:not(.comments-page) .over18 .link .tagline,html[lang="dm"] body:not(.comments-page) .over18 .tagline,html[lang="dm"] body:not(.comments-page) .over18 .entry .buttons li a,html[lang="dm"] body:not(.comments-page) .over18 .entry .buttons li + li,html[lang="dm"] body:not(.comments-page) .over18 .md p {color: #fe728b!important}
 
html[lang="dm"] body {background: #222}
 
html[lang="dm"] div.content {background-color: #333;border:none}
 
html[lang="dm"] .entry .buttons li a {color: #ccc!important}
 
html[lang="dm"] .thing .title:visited,html[lang="dm"] .commentarea .menuarea .toggle a,html[lang="dm"] .commentNavSortType,html[lang="dm"] .dropdown.lightdrop .selected {color: #ababab !important;}
 
html[lang="dm"] .thing .title {color: #fff !important}
 
html[lang="dm"] input[name="uh"] ~ a:after {display: none}
 
html[lang="dm"] .entry .buttons .may-blank {color: #fff}
 
html[lang="dm"] a.comments::after {color: #ccc!important}
 
html[lang="dm"] .flair {border: none}
 
html[lang="dm"] .side {border: #4c4c4c;background:#333}
 
html[lang="dm"] div.leavemoderator,html[lang="dm"] .toggle .option.active,html[lang="dm"] .titlebox form.toggle {color: #ccc}
 
html[lang="dm"] .side .usertext-body {background-image: none;padding:0}
 
html[lang="dm"] .icon-menu a {color: #fff}
 
html[lang="dm"] .sidecontentbox .title h1 {color: #fff;font-weight:700}
 
html[lang="dm"] .side .titlebox .md h3 a {background: #444}
 
html[lang="dm"] .side .titlebox .md h3 a:hover {background: #666}
 
html[lang="dm"] .side .titlebox .md h3 a:active {background: #222}
 
html[lang="dm"] .titlebox span.subscribers,html[lang="dm"] .titlebox .users-online {color: #fff!important}
 
html[lang="dm"] #sr-header-area a,
html[lang="dm"] h2,html[lang="dm"] .sidecontentbox .collapse-button {color: #fff}
 
html[lang="dm"] .sidecontentbox .collapse-button {border: 1px solid #fff;}
 
html[lang="dm"] body.res #sr-header-area .sr-bar a.RESShortcutsCurrentSub,html[lang="dm"] .commentarea span.score {color: #fff!important}
 
html[lang="dm"] .side .md h4 a {color: #fff!important}
 
html[lang="dm"] .side .md h4 {background: #323232}
 
html[lang="dm"] body:not(.comments-page) div.content .entry,html[lang="dm"] .md hr {border-bottom: 1px solid #666}
 
html[lang="dm"] .content .md a {color: #a4d11b !important}
 
html[lang="dm"] .titlebox a {color: #a4d11b}
 
html[lang="dm"] .link.promotedlink.promoted {background-color: #565656}
 
html[lang="dm"] span.score,html[lang="dm"] .sidecontentbox .more a {color: #ccc!important}
 
html[lang="dm"] #progressIndicator,html[lang="dm"] .commentarea .panestack-title .title,html[lang="dm"] .commentingAs,html[lang="dm"] .md,html[lang="dm"] .linkinfo .date,html[lang="dm"] div.score,html[lang="dm"] span.totalvotes,html[lang="dm"] .linkinfo .shortlink {color: #fff!important}
 
html[lang="dm"] body.comments-page .link {background: #323232}
 
html[lang="dm"] .commentarea > .usertext {background-color: #292929}
 
html[lang="dm"] .commentarea .thing,
html[lang="dm"] .commentarea .child .thing.comment,
html[lang="dm"] .commentarea .child .thing.comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment .comment .comment {
    border:1px solid #333 !important
}
 
html[lang="dm"] .commentarea .child .thing.comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment .comment .comment {
    background: #222 !important;
}
 
html[lang="dm"] .commentarea .thing,
html[lang="dm"] .commentarea .child .thing.comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment,
html[lang="dm"] .commentarea .child .thing.comment .comment .comment .comment .comment .comment .comment .comment .comment .comment {
       background: #292929 !important;
}
 
 
html[lang="dm"] button {background-color: #999;color:#000}
 
html[lang="dm"] .tagline a,html[lang="dm"] #RESShortcutsSort,html[lang="dm"] #RESShortcutsRight,html[lang="dm"] #RESShortcutsLeft,html[lang="dm"] #RESShortcutsAdd,html[lang="dm"] #RESShortcutsTrash {color: #fff}
 
html[lang="dm"] .shortlink .input {
    background-color: #222;
    color: #fff;
    border: 1px solid #666!important;
 }
 
html[lang="dm"] .side .titlebox .md h1,html[lang="dm"] .side .md ol,html[lang="dm"] .error,html[lang="dm"] .md del,html[lang="dm"] .dropdown.srdrop span.selected.title,html[lang="dm"] .menuarea,html[lang="dm"] #previoussearch label,html[lang="dm"] .search-page .searchfacets,html[lang="dm"] .searchfacet a,html[lang="dm"] .wiki-page-content .md h1,html[lang="dm"] .titlebox h1 a,html[lang="dm"] .wiki-page .wikititle,html[lang="dm"] .wiki-page .pageactions .wikiaction-current,html[lang="dm"] .generic-table,html[lang="dm"] .md pre code,html[lang="dm"] .formtabs-content .infobar,html[lang="dm"] .roundfield,html[lang="dm"] .wiki-page .wiki-page-content .wiki.md,html[lang="dm"] .wiki-page .wiki-page-content .wiki.md h2, html[lang="dm"] .side .md p,html[lang="dm"] .link .tagline,html[lang="dm"] .entry .buttons .may-blank,html[lang="dm"] .comment .flat-list li a:hover,html[lang="dm"] .link .entry .buttons li a:hover,html[lang="dm"] .link .domain a:hover, html[lang="dm"] #preview-table > table > tbody > tr > th,html[lang="dm"] form#image-upload.image-upload,html[lang="dm"] .pretty-form,  html[lang="dm"] .commentNavSortType:hover,html[lang="dm"] .commentarea .menuarea .toggle a:hover,html[lang="dm"] .comment .flat-list li a[onclick*="reply"],html[lang="dm"] .loadFlat:hover,html[lang="dm"] .submit-page .title,html[lang="dm"] .error,html[lang="dm"] #progressIndicator a#NERStaticLink, html[lang="dm"] .deepthread a, html[lang="dm"] .deepthread::after {color: #fff!important}
 
html[lang="dm"] .search-page .searchfacets {background: #555}
 
html[lang="dm"] .roundfield,html[lang="dm"] .formtabs-content .infobar {background-color: #333}
 
html[lang="dm"] .md code {background-color: #222;border:1px solid #666!important}
 
html[lang="dm"] .md pre {padding: 0;background-color:transparent!important; border: none;}
 
html[lang="dm"] .wiki-page div.content {background-color: transparent;border:none}
 
html[lang="dm"] .livePreview .md p {color: #000}
 
html[lang="dm"] .wiki-page .infobar {background: none;border:none}
 
html[lang="dm"] .wiki-page .wiki-page-content,html[lang="dm"] .wiki-page .pageactions,html[lang="dm"] .wiki-page .wikititle,html[lang="dm"] .content.submit .info-notice {background-color: #444!important}
 
html[lang="dm"] body:not(.search-page) #search input[type="text"] {
    background: #222!important;
    border: 1px solid #444;
    color: #fff
 }
 
html[lang="dm"] .submit-page .content a {color: #999 !important;}
 
html[lang="dm"] .md table,html[lang="dm"] .md ul {color: #fff!important}
 
html[lang="dm"] .side .titlebox {color: #fff}
 
html[lang="dm"] ::-moz-selection {background: #666;color:#fff}
 
html[lang="dm"] .linkinfo {background-color: #303030}
 
html[lang="dm"] .debuginfo, html[lang="dm"] .footer-parent {background: #111;}
 
html[lang="dm"] .gold-accent.comment-visits-box {color: #fff}
 
html[lang="dm"] ul#image-preview-list li {
    background: #333;
    border: none;
    color: #fff
 }
 
html[lang="dm"] .flairselector h2,html[lang="dm"] .flairselector .tagline a {color: #000!important}
 
html[lang="dm"] .wiki-page .wiki-page-content .md.wiki h4 {color: #fff!important}
 
html[lang="dm"] .comments-page .content .infobar {
    background: #444;
    color: #fff;
 }
 
html[lang="dm"]     .commentarea textarea:not(:focus):not(:hover) {
            color: transparent;
            background-color: #444 !important;
        text-shadow: 0px 0px 12px #000;
    }
 
html[lang="dm"]     .commentarea textarea:hover,
html[lang="dm"] .roundfield textarea:hover,
html[lang="dm"] .roundfield input[type="url"]:hover,
html[lang="dm"] .roundfield input[type="text"]:hover {border: 1px solid #9f9f9f;}
 
html[lang="dm"] textarea,
html[lang="dm"] .roundfield input[type="text"],
html[lang="dm"] .roundfield input[type="url"] {
    background-color: #222!important;
    color:#fff!important;
    transition: 300ms;
    transition: 300ms;
    border: 1px solid #666;
}
 
html[lang="dm"] .usertext-edit textarea:focus,
html[lang="dm"] .roundfield textarea:focus,
html[lang="dm"] .roundfield input[type="url"]:focus,
html[lang="dm"] .roundfield input[type="text"]:focus {
    border: 1px solid #666;
    transition: 0.1s ease;
    border-bottom: 3px solid #999 !important;
}
 
html[lang="dm"] .toc ul {background: transparent;}
 
html[lang="dm"] .deepthread a, .deepthread::after {color: white;}
 
html[lang="dm"] .commentarea .comment a.expand:hover {background: #999; color: #333;}
 
html[lang="dm"] .markdownEditor .edit-btn:not(.btn-macro) {background-color: #ccc !important; border: none !important; border-radius: 4px !important;}
 
html[lang="dm"] .expando-button,html[lang="dm"]  .expando-button.expanded,html[lang="dm"]  .expando-button.expanded:hover,html[lang="dm"] .expando-button:hover {border-radius: 20px !important; background-color: #999 !important;}
 
html[lang="dm"] #REScommentSubToggle {
    background: #ccc !important;
    color: #000 !important;
}
 
html[lang="dm"] .tabmenu {background: #333;}
 
html[lang="dm"] .side .md h2 + ul {background: #222;}
 
html[lang="dm"] .side .sidecontentbox a {color: #ccc}
 
html[lang="dm"] .RES-keyNav-activeElement,
html[lang="dm"] .RES-keyNav-activeElement .md-container {
    background-color: #444 !important;
    border-color: #aaa;
}
 
 
 
/*
 
Credits
 
/r/EnTypo for Enhanced Typograhy
/r/Apple for Messages alert
/r/Naut for Moderator Distinguish Message
/u/Karma_4_free for Subreddit List style from /r/Apicem
/r/Overwatch for comment collapse button
/r/Slight for input box underline
 
MANDATORY LEGAL NOTICES/CREDITS - DO NOT REMOVE
 
Gear by Alexander Simone from the Noun Project
Compass by Vicons Design from the Noun Project
Arrow Up by Raashid.A from the Noun Project
Add by John Chapman from the Noun Project
Thumbtack by Bonnie Beach from the Noun Project
Conversation by vijay sekhar from the Noun Project
Warning by Lorena Salagre from the Noun Project
Sliders by Remco Homberg from the Noun Project
Power by Gregor Črešnar from the Noun Project
message by Hakan Yalcin from the Noun Project
read message by Hakan Yalcin from the Noun Project
 
*/
 
 
 
/*ADD SNIPPETS BELOW THIS LINE

    ============================================================================*/
/*HEADER*/

#header { 
   background : url(%%banner1%%); 
}
/*END HEADER*/
/*SIDEBAR IMAGE*/
.side::before {
    display: block;
    content: "";
    padding-top: 322px;
    width: 300px;
    background: url(%%side%%) no-repeat;
    border-radius: 5px 5px 0px 0px;
    background-size:contain;
    background-size:300px;
}
/*END SIDEBAR IMAGE*/
/*ENABLE SIDEBAR ON SUBMIT PAGE*/

.submit-page .footer {display: none;}

.submit-page div.content {
    margin-top: 20px !important;
    margin-bottom: 50px !important;
    background: none;
    border: none;
        position: absolute; 
        left: calc(50% - 510px);
 }

.submit-page .side {
    display: block !important;
    border: 1px solid #E1E1E1;
    border-radius: 5px;
    padding: 0px;
    width: 300px;
    background: #FFF none repeat scroll 0% 0%;
    position: absolute;
    margin-top: 65px;
    left: calc(50% - -185px);
}

.submit-page .content::before, .submit-page .bottommenu,.submit-page .debuginfo, .submit-page .footer-parent,  .submit-page .content h1 {display: none;}
.submit-page .side #search {top: -45px;}
.submit-page .sidecontentbox .content {width: 265px;}

.submit-page ul.tabmenu.formtab {
        margin-left: 8px;
        width: 672px;
}

.subscribers .word, .users-online .word{
display:none;
}

.subscribers .number:after { 
content: " friends of humanity"
}

.users-online .number:after { 
content: " tossing coins to their Witcher" 
}

/*END ENABLE SIDEBAR ON SUBMIT PAGE*/
```
