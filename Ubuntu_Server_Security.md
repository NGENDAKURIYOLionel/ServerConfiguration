# Secure Ubuntu Xenial Xerus LTS 16.04
Here are a compilation of some server configurations that we request at Innoveos.
### OS: Ubuntu Server LTS 16.04
We will provide you:
* An email to which all the notifications will be sent.innoveos_report and sender innoveos_notifier
* An name of the machine : Machine_Name

# Configure the firewall
The firewall must be configured in so that the in-going and the out-going network traffic will be controlled.

The firewall must be configured to:
* Allow incoming HTTP(S) (TCP port 80-443) requests
* Allow outgoing ntp (port 123) requests
* Allow outgoing smtp **(TCP port 25 & 587 & 465)** requests
* Allow outgoing ssh **(TCP port 72)** requests * Notice that it is not the default port*
* Drop the rest

#Configure the email notifier SSMTP
SSMTP is our choice because it is more lightweight than  POSTFIX.

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
AuthPass=xxxxxxxxx
FromLineOverride=YES
#UseTLS=yes                   # apparently has been deprecated
UseSTARTTLS=YES

#For further details (man ssmtp && man ssmtp.conf)
```
The value of `innoveos_notifier` will be given to you.

*Note*: At gmail the `innoveos_notifier` must be an account that is configured as `less secure` if not email notifications will not be sent from the server.
### How to send an email notification
Here is an example of how to send an notification to our email (i.e `innoveos_report`)

```python
#Notice that there is no space between \n and the begining of the body
echo -e "Subject: Report Alert \nHere it is the body of the message " | ssmtp innoveos_report@gmail.com
```

#Configure an HIDS (Host based intrusion detection system)
An HIDS must be installed so that we controll any modification of some specific folders and files.

The HIDS must send us a **email notification** to our email address(that we will provide i.e innoveos_report) via ssmtp in case of a supervised folder or file is modified.

We recommend the AIDE (Advanced intrusion detection system) tool. Because of its detailled reports.
#### Installation AIDE
To install do:
```python
apt-get install aide
```
Some important folders are:
* /etc/aide/aide File containing some configuration variables
* /etc/aide/aide.conf and /etc/aide/aide.conf.d/ – Default AIDE configuration files.
* /var/lib/aide/aide.db – Default location for AIDE database.
* /var/lib/aide/aide.db.new – Default location for newly-created AIDE database.

Now ,update some some variables in the file ` /etc/default/aide` 
```nginx
FQDN=Machine_Name
MAILSUBJ="Report for $FQDN"
MAILTO=innoveos_report
QUIETREPORTS=yes 
```
Now, in the file  `/etc/aide/aide.conf` , append the following lines 
```
# check only permissions, inode, user and group for etc
CustomRule = p+i+n+u+g+s+b+m+c+md5+sha1 

/etc CustomRule     		
/bin CustomRule      		
/sbin CustomRule     		
/var CustomRule			

# Folders to ignore
!/var/log/.*     		# ignore the log dir it changes too often
!/var/spool/.*   		# ignore spool dirs as they change too often
!/var/adm/utmp$  		# ignore the file /var/adm/utmp
!/var/cache/.*   		# ignore the cache
!/usr/src/.*			# ignore src folder
!/usr/tmp/.*			# ignore tmp folder
!/root/.bash_history$   	# ignore file - its always going to change
!/root/.nano_history$   	# ignore file - its always going to change
!/var/www/logs/.*		# ignore the weblogs
!/*.log				# ignore log files
!/var/lib/nginx/proxy/.* 	# ignore transparent proxy files
!/var/lib/php5/.*		# ignore php session files that gc does not clean up

# Some reported Attributes 

report_attributes = u+g+ftype+md5+p+m+c
ignore_list = b+i+n+l+s+a+S+I+sha1+sha256+sha512+rmd160+tiger+haval+crc32

```

Here are the entity that must be monitored :
* /etc : folder
* /usr : folder
* /root: folder
....





