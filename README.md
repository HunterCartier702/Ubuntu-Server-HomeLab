# Welcome to my Ubuntu-Server 24.04.2 LTS HomeLab

## Table of Contents

  - [Introduction](#intro)
  - [Inital Setup](#initial)
  - [Installing Samba](#samba)
  - [Installing NFS](#nfs)

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
***HTOP is one of those tools that looks really confusing when you first see it, but once you start to learn what its showing it really is an awesome tool:***
<p align="center"><img alt="HTOP" src="images/3HTOP.png" height="auto" width="600"></p>

***I then create my .vimrc file with my personal preferences and copy it to the /etc/skel directory, so when I create a new user they also get that file. Openssh-server was installed initially so now it's time to sit comfortably at my desktop and SSH in.***

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

***Adding my first unprivileged user:***
```shell
$ sudo useradd -m -d /home/cartier -c "Hunter Cartier" -s /bin/bash cartier
$ tail -1 /etc/passwd
cartier:x:1001:1001:Hunter Cartier:/home/cartier:/bin/bash
$ sudo passwd cartier
New password: 
Retype new password: 
passwd: password updated successfully
```

***I decided to create a simple cronjob just cause***
<p align="center"><img alt="Cron" src="images/2Crontab.png" height="auto" width="600"></p>

***Now time to install services. Ubuntu starts services when they're installed, so after editing the config files I usually just run "sudo systemctl restart/reload \<service\>"***

## <a name="samba"></a>Installing Samba
I set up a samba share as I have one windows pc and it may come in handy related to school projects and transferring files.
```shell
$ sudo apt install samba

$ testparm /etc/samba/smb.conf
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
# A simple configuration file to get started:

# Global parameters
[global]
	map to guest = Bad User
	name resolve order = host bcast wins
	security = USER
	server string = File Server
	idmap config * : backend = tdb
	include = /etc/samba/smbshared.conf


[Documents]
	force group = users
	force user = cartier
	guest ok = Yes
	path = /share/documents


[Public]
	create mask = 0664
	directory mask = 0777
	force create mode = 0644
	force directory mode = 0777
	force group = users
	force user = cartier
	guest ok = Yes
	path = /share/public
	read only = No


$ sudo mkdir -p /share/documents 
$ sudo mkdir /share/public
$ sudo chown -R cartier:users /share
$ sudo systemctl start smbd
$ smbclient //ubuntu-home-server/documents -U cartier
```
<p align="center"><img alt="smbclient" src="images/4SMBClient.png" height="auto" width="600"></p>

## <a name="nfs"></a>Installing NFS
I know I already have a Samba share, but I still wanted to do this for the practice. 
```shell
# On server: 
$ man exports # documentation
# Creating directories for the shares. Each share in NFS is known as an export
$ sudo mkdir -p /exports/backup
$ sudo mkdir /exports/documents
$ sudo mkdir /exports/public
$ sudo apt install nfs-kernel-server

# Creating the exports file
$ cat /etc/exports
/exports *(ro,fsid=0,no_subtree_check)
/exports/backup 192.168.0.1/255.255.255.0(rw,no_subtree_check)
/exports/documents 192.168.0.1/255.255.255.0(ro,no_subtree_check)
/exports/public 192.168.0.1/255.255.255.0(rw,no_subtree_check)

# From client:
$ sudo apt update && sudo apt install nfs-common
$ sudo mkdir /mnt/documents
$ sudo mount 192.168.0.231:/documents /mnt/documents
# NFS isn't as easy as Samba. What do you think?
$ ls /mnt/documents
nfs_share.txt
```
<p align="center"><img alt="NFS" src="images/5NFS_Share.png" height="auto" width="600"></p>
