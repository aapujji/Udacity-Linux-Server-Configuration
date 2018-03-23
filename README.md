# Udacity-Linux-Server-Configuration

## Description

IP Address: 18.188.114.75
Port: 2200

URL: http://ec2-18.188-114-75.us-east-2a.compute.amazonaws.com

## Server Setup


### Step 1: Update currently installed packages:

```
sudo apt-get update
sudo apt-get upgrade
```


### Step 2: Create user grader

```
sudo adduser grader
```

Give user 'grader' a password and finish creating the user.


### Step 3: Give grader permission to sudo:

```
sudo visudo
```

Copy the line: ```root  ALL=(ALL:ALL) ALL``` onto a new line and replace root with grader.


### Step 4: Create SSH key pair for grader:

#### On your lcoal machine terminal window: 
```ssh-keygen```

Enter the path for the file: ```"C:/Users/YOUR-USER-NAME/.ssh/app-server"```
Note: THe path to the file must be this path, but you can name the file anything you want. You don't have to name it app-server.

Enter a passphrase.

Then open the newly created pub file: ```cat .ssh/your-file-name.pub```

Copy the contents of the file.

#### On your server: 

```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```

Paste the contents of the .pub file and save.

```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
sudo su - ubuntu
```


### Step 5: Force Key Based Authorization:

```
sudo nano /etc/ssh/sshd_config
```

Change the line "PasswordAuthentication yes" to "PasswordAuthentication no".


### Step 6: Configure local time to UTC

```
cat /etc/timezone
```

If this returns time in UTC, then nothing needs to be done. If not:
```
sudo dpkg-reconfigure tzdata
```

Choose None of the above and then choose UTC.


### Step 7: Install and Configure Apache:

Install Apache and mod_wsgi: 
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
```

Enable mod_wsgi: 
```
sudo a2enmod wsgi
```


### Step 8: Download Item Catalog Project
Install git: 
```
sudo apt-get install git
```

Create the project directory:
```cd /var/wwww
mkdir catalog
cd catalog
sudo git clone https://github.com/aapujji/Udacity-Full-Stack-Nanodegree---Item-Catalog.git
```

Rename the folder that you cloned:
```
sudo mv ./Udacity-Full-Stack-Nanodegree---Item-Catalog ./catalog
```

Rename the application file: 
```
sudo mv /var/www/catalog/catalog/itemcatalog.py /var/www/catalog/catalog/__init__.py
```


### Step 9: Install Flask and other dependencies
```
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
sudo pip install Flask
sudo pip install sqlalchemy psycopg2 requests oauth2client httplib2
```


### Step 10: Configure and Enable Virtual Host:
```
sudo nano \etc\apache2\sites-available\catalog.conf
```

Add the following lines of code to the file and save:
```
<VirtualHost *:80>
		ServerName 18.188.114.75
    ServerAlias ec2-18.188.114.75.us-east-2a.compute.amazonaws.com
		ServerAdmin admin@18.188.114.75
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/Udacity-Full-Stack-Nanodegree---Item-Catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catalog/Udacity-Full-Stack-Nanodegree---Item-Catalog/static
		<Directory /var/www/catalog/Udacity-Full-Stack-Nanodegree---Item-Catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the virtual host: 
```
sudo a2ensite catalog
```


### Step 11: Create the .wsgi File
```
cd /var/www/catalog
sudo nano catalog.wsgi
```

Add the following lines to the wsgi file and save:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```


### Step 12: Restart Apache
```
sudo service apache2 restart
```


### Step 13: Install PostgreSQL and create database:
```
sudo apt-get install postgresql
```

To make sure that remote access is not allowed:
```
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

Make sure the IP address is pointing to 127.0.0.1. If it is, then remote access is disabled.

Create database:
```
sudo su - postgres
```

Type in ```psql```

Then enter the following:
```
CREATE USER catalog WITH PASSWORD 'catalog';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
```
Exit out of psql and postgres:
```
\q
exit
```


### Step 14: Update Project Files
Update database_setup.py, populatedb.py, and __init__.py with the new engine connection:
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

Run the following lines: 
```
sudo python database_setup.py
sudo python populatedb.py
```

Update __init__.py to make sure that Google login will work:
```
CLIENT_ID = json.loads(
    open('/var/www/catalog/Udacity-Full-Stack-Nanodegree---Item-Catalog/client_secrets.json', 'r').read())['web']['client_id']
```


### Step 15: Change port from 22 to 2200:

```
sudo nano /etc/ssh/sshd_config
```

Comment out hte line "Port 22" in the list of ports
Add the line "Port 2200" to the list of ports

Restart SSH service:

```
sudo service ssh restart
```

### Step 16: Confugre the UFW:

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
sudo ufw enable
```

Check status of UFW by typing ```sudo ufw status```. ALl the ports should show.


## Resources

1. [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

2. [Harushimo](https://github.com/harushimo/linux-server-configuration)
