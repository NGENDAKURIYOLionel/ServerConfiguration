# Secure Ubuntu Xenial Xerus LTS 16.04
Here are a compilation of some server configurations that we request at Innoveos.
### OS: Ubuntu Server LTS 16.04
We will provide you:
* An email to which all the notifications will be sent.innoveos_report and sender innoveos_notifier
* An name of the machine : Machine_Name

# Configure the firewall
A firewall must be configured in so that the in-going and the out-going network traffic can be controlled.
### General rules
The firewall must be configured to:
* Allow incoming HTTP(S) (TCP port 80-443) requests
* Allow outgoing ntp (port 123) requests
* Allow outgoing smtp **(TCP port 25 & 587 & 465)** requests
* Allow incoming ssh **(TCP port 7200)** requests * Notice that it is not the default port *
* Allow all traffic from the localhost.
* Drop the rest

###  Rules against remote brute-force
First ,drop incoming connections which make more than 5 connections attempts upon ssh port within 60 seconds
```
ssh_port=7200
$IPT -I INPUT -p tcp --dport ${ssh_port} -i ${inet_if} -m state --state NEW -m recent  --set
$IPT -I INPUT -p tcp --dport ${ssh_port} -i ${inet_if} -m state --state NEW -m recent  --update --seconds 60 --hitcount 5 -j DROP
```
Second, secure the web server by limiting connections per second
```
/sbin/iptables -A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --set
/sbin/iptables -A INPUT -p tcp --dport 80 -i eth0 -m state --state NEW -m recent --update --seconds 60  --hitcount 15 -j DROP
service iptables save
```

# Configure port knocking
 We use port knocking for ssh port. It means that the port is closed until some 
 specific packets are sent on a sequence of ports.(That we will provide).
 
 For example, edit the configuration file `/etc/knockd.conf` so that the port will be automatically closed 
 behind the logged user after just 10 seconds.

 ```
[options]
    UseSyslog

[SSH]
    sequence = 5438,3428,3280,4479
    tcpflags = syn
    seq_timeout = 15
    start_command = /sbin/iptables -I INPUT 1 -s %IP% -p tcp --dport 7200 -j ACCEPT
    cmd_timeout   = 10
    stop_command  = /usr/sbin/iptables -D INPUT -s %IP% -p tcp --syn --dport 7200 -j ACCEPT
 ```
Now enable the knockd service by editing the file `/etc/default/knockd` as follow:
```
START_KNOCKD=1
```
Then, start the service by `sudo service knockd start`.
## Knocking
To open the specified port, just use the knockd program available for linux, MAC and Windows from the 
(official website)[http://www.zeroflux.org/projects].
Then, just do:
```
knock server_ip_address 5438 3428 3280 4479
```
If it is not available, you can use nmap like this.
### Open the ssh port :
```
for x in 5438 3428 3280 4479; do nmap -Pn --host_timeout 201 --max-retries 0 -p $x server_ip_address; done
```
`Note`: For more informations that are more precised, check out the book `Linux server security, Pge 16`  on scanlibs.com

#Configure the email notifier SSMTP

`SSMTP` is our choice because it is more lightweight than  POSTFIX.

Download and install the Mail Transfer Agent ssmtp.
```python
apt-get install ssmtp
```
Edit the file `/etc/ssmtp/ssmtp.conf` so it looks like this:
```ngnix
root=innoveos_notifier@gmail.com
mailhub=smtp.gmail.com:587
rewriteDomain=gmail.com
hostname=Machine_Name
AuthUser=innoveos_notifier
AuthPass=innoveos_notifier_password
FromLineOverride=YES
#UseTLS=yes                   # apparently has been deprecated
UseSTARTTLS=YES

#For further details (man ssmtp && man ssmtp.conf)
```
The value of `innoveos_notifier` will be given to you.

*Note*: At gmail the `innoveos_notifier` must be an account that is configured as `less secure` 
if not email notifications will not be sent from the server.
### How to send an email notification
Here is an example of how to send an notification to our email 
(#Notice that there is no space between \n and the begining of the body)
```python
echo -e "Subject: Report Alert \nHere it is the body of the message " | ssmtp innoveos_report@gmail.com
```

#Configure an HIDS (Host based intrusion detection system)
An HIDS must be installed so that we controll any modification of some specific folders and files.

The HIDS must send us a **email notification** to our email address(that we will provide i.e innoveos_report) via ssmtp in case of a supervised folder or file is modified.

We recommend the file integrity checker `tripwire` because of it's detailled reports.

## Configure Tripwire HIDS
During the installation you will be asked to give 2 passwords (keys). Don't forget them.
##### Install tripwire
```
sudo apt-get install tripwire 
```
##### Generate the policy file for tripwire by doing
 ```
 sudo twadmin --create-polfile /etc/tripwire/twpol.txt
 ```
##### Initialize the database of tripwire to check if all the specified paths in ` /etc/tripwire/twpol.txt` exist on the
 system 
 ```
sudo tripwire --init
```

##### If there is some paths that are labeled as non-existant, comment them out in the policy file ` /etc/tripwire/twpol.txt`
```
sudo nano /etc/tripwire/twpol.txt
```

##### Notice the errors  resulting  that some folders don't exists on   your computer...then comment them out

##### Comment out some paths that changes too much. For example `/var/log/` by `#/var/log/` 
###### Replace `YOURMAIL` by you current e-mail.

The policy file should like that:

```

#
# Standard Debian Tripwire configuration
#
#
# This configuration covers the contents of all 'Essential: yes'
# packages along with any packages necessary for access to an internet
# or system availability, e.g. name services, mail services, PCMCIA
# support, RAID support, and backup/restore support.
#

#
# Global Variable Definitions
#
# These definitions override those in to configuration file.  Do not         
# change them unless you understand what you're doing.
#

@@section GLOBAL
TWBIN = /usr/sbin;
TWETC = /etc/tripwire;
TWVAR = /var/lib/tripwire;

#
# File System Definitions
#
@@section FS

#
# First, some variables to make configuration easier
#
SEC_CRIT      = $(IgnoreNone)-SHa ; # Critical files that cannot change

SEC_BIN       = $(ReadOnly) ;        # Binaries that should not change

SEC_CONFIG    = $(Dynamic) ;         # Config files that are changed
		        # infrequently but accessed
		        # often

SEC_LOG       = $(Growing) ;         # Files that grow, but that
			             # should never change ownership

SEC_INVARIANT = +tpug ;              # Directories that should never
		        # change permission or ownership

SIG_LOW       = 33 ;                 # Non-critical files that are of
				     # minimal security impact

SIG_MED       = 66 ;                 # Non-critical files that are of
				     # significant security impact

SIG_HI        = 100 ;                # Critical files that are
				     # significant points of
				     # vulnerability

MAILTO = YOURMAIL@gmail.com;

#
# Tripwire Binaries
#
(
  rulename = "Tripwire Binaries",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	$(TWBIN)/siggen			-> $(SEC_BIN) ;
	$(TWBIN)/tripwire		-> $(SEC_BIN) ;
	$(TWBIN)/twadmin		-> $(SEC_BIN) ;
	$(TWBIN)/twprint		-> $(SEC_BIN) ;
}

#
# Tripwire Data Files - Configuration Files, Policy Files, Keys,
# Reports, Databases
#

# NOTE: We remove the inode attribute because when Tripwire creates a
# backup, it does so by renaming the old file and creating a new one
# (which will have a new inode number).  Inode is left turned on for
# keys, which shouldn't ever change.

# NOTE: The first integrity check triggers this rule and each
# integrity check afterward triggers this rule until a database update
# is run, since the database file does not exist before that point.
(
  rulename = "Tripwire Data Files",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	$(TWVAR)/$(HOSTNAME).twd	-> $(SEC_CONFIG) -i ;
	$(TWETC)/tw.pol			-> $(SEC_BIN) -i ;
	$(TWETC)/tw.cfg			-> $(SEC_BIN) -i ;
	$(TWETC)/$(HOSTNAME)-local.key	-> $(SEC_BIN) ;
	$(TWETC)/site.key		-> $(SEC_BIN) ;

	#don't scan the individual reports
	$(TWVAR)/report			-> $(SEC_CONFIG) (recurse=0) ;
}

#
# Critical System Boot Files
# These files are critical to a correct system boot.
#
(
  rulename = "Critical system boot files",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/boot			-> $(SEC_CRIT) ;
	/lib/modules		-> $(SEC_CRIT) ;
}

(
  rulename = "Boot Scripts",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/etc/init.d		-> $(SEC_BIN) ;
	#/etc/rc.boot		-> $(SEC_BIN) ;
	/etc/rcS.d		-> $(SEC_BIN) ;
	/etc/rc0.d		-> $(SEC_BIN) ;
	/etc/rc1.d		-> $(SEC_BIN) ;
	/etc/rc2.d		-> $(SEC_BIN) ;
	/etc/rc3.d		-> $(SEC_BIN) ;
	/etc/rc4.d		-> $(SEC_BIN) ;
	/etc/rc5.d		-> $(SEC_BIN) ;
	/etc/rc6.d		-> $(SEC_BIN) ;
}


#
# Critical executables
#
(
  rulename = "Root file-system executables",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/bin			-> $(SEC_BIN) ;
	/sbin			-> $(SEC_BIN) ;
}

#
# Critical Libraries
#
(
  rulename = "Root file-system libraries",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/lib			-> $(SEC_BIN) ;
}


#
# Login and Privilege Raising Programs
#
(
  rulename = "Security Control",
  severity = $(SIG_MED),
  emailto = $(MAILTO)
)
{
	/etc/passwd		-> $(SEC_CONFIG) ;
	/etc/shadow		-> $(SEC_CONFIG) ;
}




#
# These files change every time the system boots
#
(
  rulename = "System boot changes",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/var/lock		-> $(SEC_CONFIG) ;
	/var/run		-> $(SEC_CONFIG) ; # daemon PIDs
	/var/log		-> $(SEC_CONFIG) ;
}

# These files change the behavior of the root account
(
  rulename = "Root config files",
  severity = 100,
  emailto = $(MAILTO)
)
{
	/root				-> $(SEC_CRIT) ; # Catch all additions to /root
#	/root/mail			-> $(SEC_CONFIG) ;
#	/root/Mail			-> $(SEC_CONFIG) ;
#	/root/.xsession-errors		-> $(SEC_CONFIG) ;
#	/root/.xauth			-> $(SEC_CONFIG) ;
#	/root/.tcshrc			-> $(SEC_CONFIG) ;
#	/root/.sawfish			-> $(SEC_CONFIG) ;
#	/root/.pinerc			-> $(SEC_CONFIG) ;
#	/root/.mc			-> $(SEC_CONFIG) ;
#	/root/.gnome_private		-> $(SEC_CONFIG) ;
#	/root/.gnome-desktop		-> $(SEC_CONFIG) ;
#	/root/.gnome			-> $(SEC_CONFIG) ;
#	/root/.esd_auth			-> $(SEC_CONFIG) ;
#	/root/.elm			-> $(SEC_CONFIG) ;
#	/root/.cshrc		        -> $(SEC_CONFIG) ;
	/root/.bashrc			-> $(SEC_CONFIG) ;
#	/root/.bash_profile		-> $(SEC_CONFIG) ;
#	/root/.bash_logout		-> $(SEC_CONFIG) ;
#	/root/.bash_history		-> $(SEC_CONFIG) ;
#	/root/.amandahosts		-> $(SEC_CONFIG) ;
#	/root/.addressbook.lu		-> $(SEC_CONFIG) ;
#	/root/.addressbook		-> $(SEC_CONFIG) ;
#	/root/.Xresources		-> $(SEC_CONFIG) ;
#	/root/.Xauthority		-> $(SEC_CONFIG) -i ; # Changes Inode number on login
#	/root/.ICEauthority		    -> $(SEC_CONFIG) ;
}

#
# Critical devices
#
(
  rulename = "Devices & Kernel information",
  severity = $(SIG_HI),
  emailto = $(MAILTO)
)
{
	/dev		-> $(Device) ;
#	/proc		-> $(Device) ;
    /dev/pts                -> $(Device) ;
    #/proc                  -> $(Device) ;
    /proc/devices           -> $(Device) ;
    /proc/net               -> $(Device) ;
    /proc/tty               -> $(Device) ;
    /proc/sys               -> $(Device) ;
    /proc/cpuinfo           -> $(Device) ;
    /proc/modules           -> $(Device) ;
    /proc/mounts            -> $(Device) ;
    /proc/dma               -> $(Device) ;
    /proc/filesystems       -> $(Device) ;
    /proc/interrupts        -> $(Device) ;
    /proc/ioports           -> $(Device) ;
    /proc/scsi              -> $(Device) ;
    /proc/kcore             -> $(Device) ;
    /proc/self              -> $(Device) ;
    /proc/kmsg              -> $(Device) ;
    /proc/stat              -> $(Device) ;
    /proc/loadavg           -> $(Device) ;
    /proc/uptime            -> $(Device) ;
    /proc/locks             -> $(Device) ;
    /proc/meminfo           -> $(Device) ;
    /proc/misc              -> $(Device) ;
}

#
# Other configuration files
#
(
  rulename = "Other configuration files",
  severity = $(SIG_MED),
  emailto = $(MAILTO)
)
{
	/etc		-> $(SEC_BIN) ;
}

#
# Binaries
#
(
  rulename = "Other binaries",
  severity = $(SIG_MED),
  emailto = $(MAILTO)
)
{
	/usr/local/sbin	-> $(SEC_BIN) ;
	/usr/local/bin	-> $(SEC_BIN) ;
	/usr/sbin	-> $(SEC_BIN) ;
	/usr/bin	-> $(SEC_BIN) ;
}

#
# Libraries
#
(
  rulename = "Other libraries",
  severity = $(SIG_MED),
  emailto = $(MAILTO)
)
{
	/usr/local/lib	-> $(SEC_BIN) ;
	/usr/lib	-> $(SEC_BIN) ;
}

#
# Commonly accessed directories that should remain static with regards
# to owner and group
#
(
  rulename = "Invariant Directories",
  severity = $(SIG_MED),
  emailto = $(MAILTO)
)
{
	/		-> $(SEC_INVARIANT) (recurse = 0) ;
	/home		-> $(SEC_INVARIANT) (recurse = 0) ;
	/tmp		-> $(SEC_INVARIANT) (recurse = 0) ;
	/usr		-> $(SEC_INVARIANT) (recurse = 0) ;
	/var		-> $(SEC_INVARIANT) (recurse = 0) ;
	/var/tmp	-> $(SEC_INVARIANT) (recurse = 0) ;
}

```
##### Edit also the configuration policy of tripwire, `/etc/tripwire/twcfg.txt`
and it should look like this:

```
ROOT          =/usr/sbin
POLFILE       =/etc/tripwire/tw.pol
DBFILE        =/var/lib/tripwire/$(HOSTNAME).twd
REPORTFILE    =/var/lib/tripwire/report/$(HOSTNAME)-$(DATE).twr
SITEKEYFILE   =/etc/tripwire/site.key
LOCALKEYFILE  =/etc/tripwire/$(HOSTNAME)-local.key
EDITOR        =/usr/bin/editor
LATEPROMPTING =false
LOOSEDIRECTORYCHECKING =false
MAILNOVIOLATIONS =false
EMAILREPORTLEVEL =3
REPORTLEVEL   =3
SYSLOGREPORTING =true
MAILMETHOD    =SENDMAIL
MAILPROGRAM   =/usr/sbin/sendmail -oi -t
 #SMTPHOST      =localhost
 #SMTPPORT      =25
TEMPDIRECTORY =/tmp
```

##### Recompile the encrypted and configuration policy files to consider new settings
```
sudo twadmin --create-cfgfile -S /etc/tripwire/site.key /etc/tripwire/twcfg.txt
sudo twadmin -m P /etc/tripwire/twpol.txt
````
##### Reinitialize the database to implement our new policy:
```
sudo tripwire --init
```
##### Test if tripwire can send email
```
/usr/sbin/tripwire --test --email xxxxxx@xxxx.com
```
### Run a scan and see the report via email preconfigured
```
sudo tripwire --check -M
```
### Schedule tripwire to run hourly in this file `/etc/cron.hourly/tripwire` by editing it to:

```
#!/bin/sh -e

tripwire=/usr/sbin/tripwire
[ -x $tripwire ] || exit 0
umask 027
$tripwire --check --quiet --email-report

```
More about crontab :https://www.unixmen.com/add-cron-jobs-linux-unix/
# Define permissions 
```
#chmod 700 /root

#chmod 700 /var/log/audit

#chmod 740 /etc/rc.d/init.d/iptables

#chmod 740 /sbin/iptables

#chmod –R 700 /etc/skel

#chmod 600 /etc/rsyslog.conf

#chmod 640 /etc/security/access.conf

#chmod 600 /etc/sysctl.conf
```
# Harden the kernel via the sysctl interface 
Sysctl is an interface for examining and dynamically changing parameters in the Linux operating system.

Now, optimize  kernel parameters by editing the file `/etc/sysctl.conf` this way:

```ngnix
# Turn on execshield
kernel.exec-shield=1
kernel.randomize_va_space=1

# Enable IP spoofing protection
net.ipv4.conf.all.rp_filter=1

# Disable IP source routing
net.ipv4.conf.all.accept_source_route=0

# Ignoring broadcasts request
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.icmp_ignore_bogus_error_messages=1

# Make sure spoofed packets get logged
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Disable ICMP routing redirects
sysctl -w net.ipv4.conf.all.accept_redirects=0
sysctl -w net.ipv6.conf.all.accept_redirects=0
sysctl -w net.ipv4.conf.all.send_redirects=0
sysctl -w net.ipv6.conf.all.send_redirects=0

# Disables the magic-sysrq key
kernel.sysrq = 0

# Turn off the tcp_sack
net.ipv4.tcp_sack = 0

# Turn off the tcp_timestamps
net.ipv4.tcp_timestamps = 0

# Enable TCP SYN Cookie Protection
net.ipv4.tcp_syncookies = 1

# Enable bad error message Protection
net.ipv4.icmp_ignore_bogus_error_responses = 1

#Disable IPV6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

```
Then load the new settings by:
```
 sudo sysctl -p
```
# Enable Security enhanced Linux (SELINUX)
Harden the OS by defining a mandatory access control  policy via `selinux`.
Edit the file `/etc/sysconfig/selinux` to:
```nginx
SELINUX=enforcing
```
Test enforcing by doing:
```
getenforce
```
# Verify the file system 
All SUID/SGID bits enabled file can be used for malicious activities, when the SUID/SGID executable has a security problem.

All local or remote user can use such file.
### Identify unwanted SUID and SGID binaries
```
find / \( -perm -4000 -o -perm -2000 \) -print
find / -path -prune -o -type f -perm +6000 -ls
```
### Identify world writable files
```
find /dir -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print
```
### Identify Orphaned files and folders
```nginx
find / -xdev \( -nouser -o -nogroup \) -print
```

#13: Separate Disk Partitions

# Configure secure shell server (OpenSSH)
SSH service must have a secure configuration to allow and do what is required.

##### Authentication
The secure shell server must implement two-factor authentication that is both authentication via password and via public-key.

A authentication without public-key must be refused.
##### Ssh configuration file
Make sure these following variables are set in the  `/etc/ssh/sshd_config` or `/etc/ssh/ssh_config ` configuration file :

```
# CHANGE DEFAULT PORT
Port 7200

#ONLY USE PROTOCOL 2
Protocol 2

#ALLOW ONLY SPECIFIC USERS OR GROUPS (THAT WE WILL PROVIDE)
AllowUsers Inno_user1 Inno_user2 Inno_user3
AllowGroups sysadmin developers ...

#DISCONNECT SSH WHEN NO ACTIVITY,[TEST EACH 3 MINS IF THE CLIENT IS  ALIVE IF NOT TEST
#AGAIN 2 TIMES THEN CLOSE THE SESSION]
ClientAliveInterval 180
ClientAliveCountMax 3

#MUST SPECIFY HOW MANY SECONDS TO KEEP THE CONNECTION ALIVE WITHOUT SUCCESSFULLY LOGGING IN. (30 SECONDS)
LoginGraceTime 30

# DISABLE ROOT LOGIN ONLY SUDO
PermitRootLogin no

# REFUSE A LOGIN ATTEMPT IF THE AUTHENTICATION FILES ARE READABLE BY EVERYONE.
StrictModes yes

PasswordAuthentication yes
PermitEmptyPasswords no
PublicKeyAuthentication yes
HostbasedAuthentication no

# Banner: YOU ARE ACCESSING A INNOVEOS INFORMATION SYSTEM,..., YOU ARE RESPONSIBLE ...
Banner /etc/innoveos_banner
```
# Update openssl
View the version of openssl
 ```
 sudo openssl version -v && openssl version -b
 ```
 If the date is older than “Mon Apr 7 20:33:29 UTC 2014,” and the version is “1.0.1,” then your system is vulnerable to the Heartbleed bug.

To fix this bug, update OpenSSL to the latest version and run
 ```
sudo apt-get update
sudo apt-get upgrade openssl libssl-dev
sudo apt-cache policy openssl libssl-dev
```
# Check services running and disable those not necessary
```
sudo initctl list | grep running
```
# Backups 
Using rsnapshot

# Keep the system up-to-date
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```


more :[mailto:ngendlio@gmail.com](Contact me)
