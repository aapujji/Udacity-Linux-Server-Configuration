# Udacity-Linux-Server-Configuration

## Description

IP Address: 18.188.114.75
Port: 2200

## Setup

1. Update currently installed packages:

```sudo apt-get update```
```sudo apt-get upgrade```

2. Change port from 22 to 2200:

```sudo nano /etc/ssh/sshd_config```

Add the line "Port 2200" to the list of ports

Restart SSH service:

```sudo service ssh restart```

3. Confugre the UFW:

```sudo ufw default deny incoming```
```sudo ufw default allow outgoing```
```sudo ufw allow ssh```
```sudo ufw allow 2200```
```sudo ufw allow 22```
```sudo ufw allow 80```
```sudo ufw allow 123```
```sudo ufw enable```

Check status of UFW by typing ```sudo ufw status```. ALl the ports should show.

4. Create user grader

```sudo adduser grader```

Give user grader a password and finish creating the user.

5. Give grader permission to sudo:

```sudo visudo```

Copy the line: ```root  ALL=(ALL:ALL) ALL``` onto a new line and replace root with grader.

6. Create SSH key pair for grader:
On your lcoal machine terminal window: ```ssh-keygen```
Give a name for the file.
Enter a passphrase.
Then open the newly created pub file: ```cat .ssh/your-file-name.pub```
Copy the contents of the file.

On your server signed in as grader: 
```mkdir .ssh```
```touch .ssh/authorized_keys```
```nano .ssh/authorized_keys```
Paste the contents of the .pub file and save.
```chmod 700 .ssh```
```chmod 644 .ssh/authorized_keys```

7. Force Key Based Authorization:
```sudo nano /etc/ssh/sshd_config```
Change the line "PasswordAuthentication yes" to "PasswordAuthentication no".

8. Configure local time to UTC
```cat /etc/timezone```
If this returns time in UTC, then nothing needs to be done. If not:
```sudo dpkg-reconfigure tzdata```
Choose None of the above and then choose UTC.

9. Install and Configure Apache:
Install Apache: ```sudo apt-get install apache2```
Install mod_wsgi: ```sudo apt-get install libapache2-mod-wsgi python-dev```
Enable mod_wsgi: ```sudo a2enmod wsgi```

10. Clone GitHub repository [here](https://github.com/aapujji/Udacity-Full-Stack-Nanodegree---Item-Catalog)
