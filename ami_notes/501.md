## My personal notes for creating the KevinCrook.com 501 AWS AMI

**Purpose:**  AMI to provide Ubuntu with a desktop, Chrome Browser, Anaconda 2 & 3 (Python 2 & 3, R), Jupyter Notebook, SageMath

**Note:** for now, I'm putting this AMI on ice.  AWS has a new product that uses AWS Linux 2 and the Mate desktop.  I'm going to wait and see if they publish instructions for how to get that working.

**Alternative suggestion:** If you really need a desktop in the cloud, the Windows 2016 Server works well.  It took me about 10 minutes to create an instance and get everything I needed working and connected to a Linux instance on the back end.  If you really need a desktop in the cloud and need Linux on the back end, this is the easiest solution.  The downside is that you have to pay to run two instances, one for Linux and one for Windows.

### Status

Amazon has not traditionally provided AMIs that include a Linux desktop.  The Amazon AMIs so far have only provided a Linux server, without any GUI, which must use the Linux command line inteface (CLI).  

Amazon has always cited difficulty in providing this as the reason when I've asked their tech support.  After spending considerable time and trying several desktop packages, I have a great appreciation for the difficulty.  

It's really easy to get a Linux desktop to come up on AWS and look like it's working at first glance.  You can even use them for a while for basic things. However, when you start using it for real things, show stopping issues come up, such as black screens, grey screens, core dumps, etc.

I've tried the following packages:  Xfce, Lubuntu, and Mate.  All came up, but had show stopping issues.  None of the official documentation for any of these provides detailed instructions for how to make them work in AWS. I found some instructions in various forums, but instructions were usually minimal and basic, and only worked for simple cases.

## Here are my notes on what I've tried so far:

### Ubuntu Server

As of August 2018, when this was made, AWS was only supporting Ubuntu Server 16.04 LTS "Xenial".  
(note: 18.04 LTS "Bionic" has been out for 4 months, but not yet certified for AWS.)

Here is the image used:

Ubuntu Server 16.04 LTS (HVM), SSD Volume Type - ami-759bc50a

Launched details:
* N. Virginia Region
* t2.large - 2 CPUs, 8 GiB RAM, 9 cents an hour current pricing
* 100 GiB magnetic storage, $5 a month current pricing (not SSD which would be $10 a month)
* security Group I named 001, open ports: 22 (ssh), 3389 (rdp)

---
### Upgrade ubuntu existing packages since the AWS image is really old

The Ubuntu existing packages are really old and how tons of needed updates.  I tried the desktops both with and without these updates. 

Update the list of packages (always do this before installing anything)
```
sudo apt-get update
```

Run an upgrade on existing packages:
```
sudo apt-get -y upgrade
```

---
### Set password for ubuntu

Set a password for ubuntu.  It will be needed for remote desktop connections:
```
sudo password ubuntu
```

---
### Mate Desktop

Mate seems to be the best choice for a desktop, especially since AWS has selected it for their new product.  However, Mate has almost non-existant documentation.  I found a few articles written by people on forums that suggested the following.  I tried these and the desktop comes up, but soon give grey screens and core dumps.

Update the list of packages (always do this before installing anything)
```
sudo apt-get update
```

I tried the mate-desktop-environment packages:
```
sudo apt-get install -y mate-desktop-environment

sudo apt-get install -y mate-desktop-environment-extras
```

I also tried the ubuntu-mate-desktop:
```
sudo apt-get install -y ubuntu-mate-desktop
```

---
### xfce Desktop

Update the list of packages (always do this before installing anything)
```
sudo apt-get update
```

Install the xfce desktop package.  The -y will answer yes to everything.
```
sudo apt-get install -y xfce4
```

---
### Lubuntu Desktop


Update the list of packages (always do this before installing anything)
```
sudo apt-get update
```

Install the lubuntu-desktop package.  The -y will answer yes to everything.
```
sudo apt-get install -y lubuntu-desktop
```
---
### RDP

RDP seemed to be really, really slow.  Oddly, it's lightning fast with Windows 2016 Server on AWS, so the issue has to be with the Linux side.  

Probably better to just use VNC, although VNC has major security issues even when run over SSH.

The rdp version on Ubuntu 16.04 is really old 0.6.x versus the current 0.9.x on Ubuntu 18.04.  It was not allowing cut and paste with the newest version of the lubuntu desktop, which I cannot live without.  So we must first upgrade rdp to 0.9.x

We need to add the rdp 0.9.x package to the apt repository:
```
sudo add-apt-repository ppa:hermlnx/xrdp
```

Update the list of packages to pickup the repository we just added (also, always do this before installing anything)
```
sudo apt-get update
```

Install the xrdp package and will use the rdp 0.9.x package we just added to the .  The -y will answer yes to everything.
```
sudo apt-get install -y xrdp
```

You probably won't have an ~/.xsession file.  If not create one as follows.  If you have one, back it up, delete it, and create a new one as follows:
```
echo "lxsession -e LXDE -s Lubuntu" | tee ~/.xsession
```
```
echo "xfce4-session" > ~/.xsession
```

Check the /etc/xrdp directory for a file named startwm.sh  This file controls the starting of the windows manager.  It should look similar to this.  The last line is crucial because it tells it to use Xsession which will use our .~/.xsession file we just created.  If necessary use sudo to edit the file and correct it.
```
#!/bin/sh

if [ -r /etc/default/locale ]; then
  . /etc/default/locale
  export LANG LANGUAGE
fi

. /etc/X11/Xsession
```

Restart the xrdp service so it will read and use our ~/.xsession file we just created.
```
sudo service xrdp restart
```

---
### Windows Remote Desktop Connection

In Windows, find and run the Remote Desktop Connection.

Upper left corner, change to "smart sizing" to get rid of scroll bars.

---
### Change the number of lubuntu desktops to 1

Lubuntu desktop has 2 desktops by default.  The mouse wheel is used to move between them.  This is very useful, but since most users will not be familiar with this and will be confused by it, I will set the desktops to 1.  Users can always create more if they want.

Application Menu => Desktops on left panel style menu => Number of desktops - change to 1

---
### Disable the lubuntu desktop screen saver

Application Menu => Preferences => Screensaver => change Mode: to Disable Screen Saver

---
### Disable lubuntu desktop power management

Application Menu => Preferences => Power Manager => Display tab => uncheck Handle display power management

---
### Chrome Browser

Of, of everything I've tried, the Chrome Browser was by far the best documented, easiest to instal, and worked flawlessly.  If nothing else comes of this, at least I have good instructions for running Chrome on Linux desktop.

Using sudo, edit the /etc/apt/source.list file and add the following line at the end:
```
deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main
```

Download the linux signing key for the debian package source we just added:
```
wget https://dl.google.com/linux/linux_signing_key.pub
```

Add the key to our apt keys:
```
sudo apt-key add linux_signing_key.pub
```

Update the list of packages to pickup the debian package source we just added (also, always do this before installing anything)
```
sudo apt-get update
```

Finally, we can install chrome (stable versions):
```
sudo apt install google-chrome-stable
```

---
### Anaconda 

Downloaded, made executable, ran as ubuntu, worked.

Anaconda is installed at the individual user level, rather than at the system level.  You probably already have python installed, so best to not mess with that as lost of packages depend on it.  It will create an anaconda2 and anaconda3 directories under your user home directory.  It will add them to the front of your PATH.  Do 2 first then 3.  That way 3 will come first in the path, that way if you type "python" it will by anaconda 3, but you can always type "python2" to use version 2.  Also, you can activate the python 2 kernel for Jupyter Notebook for anaconda 3, so you can run notebooks of either version.

https://repo.anaconda.com/archive/Anaconda2-5.2.0-Linux-x86_64.sh

https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh

---
### SageMath

tbd

---
### Docker

tbd

--- 
### git 

tbd

---
### AMI

tbd
