# Secure Ubuntu Xenial Xerus LTS 16.04
Here are a compilation of some server configurations that we request at Innoveos.
### OS: Ubuntu Server LTS 16.04
We will provide you:
* An email to which all the notifications will be sent.
* 

# Configure the firewall
The firewall must be configured in so that the in-going and the out-going network traffic will be controlled.

The firewall must be configured to:
* Allow incoming HTTP(S) (TCP port 80-443) requests
* Allow outgoing ntp (port 123) requests
* Allow outgoing smtp **(TCP port 25 & 587 & 465)** requests
* Allow outgoing ssh **(TCP port 72)** requests * Notice that it is not the default port*
* Drop the rest

#Configure the MTA
SSMTP is our choice because it is more lightweight than  POSTFIX.

Download and install the MTA ssmtp.
```python
apt-get install ssmtp
```
Edit the file `/etc/ssmtp/ssmtp.conf` so it will look like this:
```ngnix
root=innoveos_HIDS@gmail.com
mailhub=smtp.gmail.com:587
rewriteDomain=gmail.com
hostname=Innoveos_Server_Project1
AuthUser=innoveos_HIDS
AuthPass=xxxxxxxxx
FromLineOverride=NO
#UseTLS=yes                   # apparently has been deprecated
UseSTARTTLS=YES

#For further details (man ssmtp && man ssmtp.conf)
```
The value of `innoveos_HIDS` will be given to you.
*Note*: Note that at gmail the `innoveos_HIDS` must be an account that is configured as `less secure` if not email notifications will not be sent from our server.


#Configure an HIDS (Host based intrusion detection system)
An HIDS must be installed so that we controll any modification of some specific folders and files.

The HIDS must send us a **email notification** to our email address(that we will provide) in case of a supervised folder
or file is modified.

Here are the entity that must be supervised:
* /etc : folder
* /usr : folder
* /root: folder
....





