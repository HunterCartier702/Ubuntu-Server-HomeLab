# Welcome to my Ubuntu-Server 24.04.2 LTS HomeLab

## Table of Contents

  - [Introduction](#intro)
  - [Inital Setup](#initial)

## <a name="intro"></a>Intro: The Home-Labtop
I turned my old gaming laptop into an Ubuntu server for practice. I've been using Linux for a over a year now starting back with VM's in Virtual Box. Then I upgraded to downloading an ISO with a USB stick and putting Linux Mint on both my laptops and dual booting my desktop. Then I took my stab at Linux+ and passed. Now while I don't care much for the certification I did learn a lot more about Linux in the process of studying for it. I then bought book about Ubuntu server and learned even more to add to my previous studying. And that is what brought me here. Finally testing some of what I had learned. I don't have fancy hardware for this home lab.. yet. I used this as more of a test. This setup I have is very temporary while I look into a more long-term solution as I continue to learn and figure out what I am after. So far I setup Samba, NFS, MariaDB, and Apache2 with my own web page that I made for guests to visit when connected to our WiFi. I then installed a new SSD and partitioned it with fdisk. I had a lot of fun with this and plan on doing more in the future. 
## <a name="initial"></a>Initial Setup
I downloaded the Ubuntu server ISO to my USB stick and wiped the drive. The setup was very easy and straight forward. I used DHCP for the inital interface setup, but used my routers web interface to set the server to a static IP. Now I can login and get started.

**First commands:**
```shell
# First I update the system
$ sudo apt update && sudo apt upgrade
# I then install what I can think of off the top of my head especially Vim
$ sudo apt install vim build-essential vlock htop tree
$ sudo ufw enable
# Don't forget to edit firewall rules for added services!
```
***HTOP is one of those things that looks confusing when you first see it, but once you start to learn what its showing it really is an awesome tool***
<p align="center"><img alt="HTOP" src="images/3HTOP.png" height="auto" width="600"></p>

I then create my .vimrc file with my personal preferences and copy it to the /etc/skel directory, so when I create a new user they also get that file. Openssh-server was installed initially so now it's time to sit comfortably at my desktop and SSH in.

***SSH:***
```shell
# I edit the /etc/banner and edit /etc/ssh/sshd_config file to allow a banner so I can add my own custom one
# Adding servers hostname to hosts file
$ sudo echo -e "192.168.x.x\tubuntu-home-server" >> /etc/hosts
# Creating an alias for the ssh command
$ echo 'alias server="ssh wannabe_admin@ubuntu-home-server"' >> .bashrc
# Generating keys for pub key auth
$ ssh-keygen
# Copying public key to server
$ ssh-copy-id wannabe_admin@ubuntu-home-server

```

<p align="center"><img alt="SSH Banner" src="images/1SSHLogin.png" height="auto" width="600"></p>

***Fixing the timezone:***
```shell
$ timedatectl
# timezone is ahead about 4 hours and this will abviously mess up any cron jobs
$ timedatectl list-timezones | grep Los
$ sudo timedatectl set-timezone America/Los_Angeles
# That fixed it
```

