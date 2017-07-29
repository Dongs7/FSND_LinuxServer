# FSND P7 - Linux Server Configuration
Linux Server


## Requirements
* AWS Lightsail
* Catalog App - <a href="https://github.com/Dongs7/FSND_Catalog">FSND_Catalog</a>

## Project Detail

### Server Information

Created a static IP and attached to the instance

- IP Address : 35.167.201.154
- URL : http://ec2-35-167-201-154.us-west-2.compute.amazonaws.com
- SSH port : 2200

### Update packages

Successfully updated all packages currently installed by

```
$ sudo apt-get update
$ sudo apt-get upgrade

```

### Change default SSH port/ Setting UFW

Changed SSH port from 22 (default) to 2200. In order to do this change,
1. Make sure to allow connection from port 2200:
 ```
 $ sudo ufw allow 2200/tcp
 ```
2. Open and edit sshd_config file by:

  ```
  $ sudo nano /etc/ssh/sshd_config

  ```
  Then, find the line "Port 22" and change it to "Port 2200".
  Save and exit the nano editor and restart the SSH:

  ```
  $ sudo service sshd restart
  ```
Now we can connect to the SSH using port 2200.

3. UFW Setting

```
 $ sudo ufw default deny incoming   // Deny all incoming connections
 $ sudo ufw default allow outgoing  // Allow all outgoing connections
 $ sudo ufw allow www               // Allow connections for HTTP (port 80)
 $ sudo ufw allow ssh               // Allow connections for SSH
 $ sudo ufw allow ntp               // Allow connections for NTP (port 123)
 $ sudo ufw enable                  // Apply and enable UFW
```
You can check your UFW setting by:
```
 $ sudo ufw status
```


### Give grader sudo access

1. First we need to create user 'grader':
```
 $ sudo adduser grader
```

2. Give sudo privilege to the user 'grader':
```
 $ sudo usermod -a -G sudo grader
```

Now the user 'grader' have sudo privilege.

### SETUP SSH-KEY for 'grader'
  - I used ssh-keygen to generate key for the user 'grader'.
    - once the key is generated, copy the key to the clipboard

1. On the linux instance, create a new folder named .ssh under /home/grader:
```
$ sudo mkdir /home/grader/.ssh                 // Create a new folder .ssh
$ sudo chown grader:grader /home/grader/.ssh   // Change the owner of the created folder to grader
$ sudo nano /home/grader/.ssh/authorized_keys  // Create a file named 'authorized_keys'
```
When the nano editor opens, paste the key generated from the ssh-keygen, then save the file and exit. Then:

```
 $ sudo chown grader:grader /home/grader/.ssh/authorized_keys  // Give file ownership to the user 'grader'
 $ sudo chmod 644 /home/grader/.ssh/authorized_keys            // Change file permission
```

2. Now the user 'grader' can connect to the linux instance using:
```
 $ sudo ssh -i ~/grader grader@35.167.201.154 -p 2200 -vv
```

### Disable root login & Enforce key-based authentication

Open and edit sshd_config file:
```
 $ sudo nano /etc/ssh/sshd_config
```

- Disable root login
Find  the line "PermitRootLogin prohibit-password", and change it to "PermitRootLogin no"

- Disable password-login authentication (Enforce key-based authentication)
Find the line "PasswordAuthentication yes", and change this to "PasswordAuthentication no"
<small>It is possible that the system already disables this. </small>

### Change Timezone to UTC

Change timezone to UTC:

```
 $ date
 ```
 If not UTC, then
 ```
 $ sudo timedatectl set-timezone UTC
```

### Install Apache/Python mod_wsgi and PostgreSQL

- Install Apache
```
 $ sudo apt-get install apache2
```

- Install Python mod_wsgi
```
 $ sudo apt-get install libapache2-mod-wsgi
```
Restart apache service to get mod-wsgi to work:
```
 $ sudo apache2ctl restart
```

- Install PostgreSQL
```
 $ sudo apt-get install postgresql postgresql-contrib
```

### Configure PostgreSQL

- Create a new database user named catalog and give limited permissions
```
 $ sudo -u postgres createuser -P catalog
 $ sudo -u postgres createdb -O catalog catalog
```


### Catalog App

- Clone the catalog app from the previous project and install required packages.

Installed packages:

Python 2.7.12
Flask
SQLalchemy
Flask-Login
Flask-Bootstrap
Flask-Uploads
Flask-WTF
OAuth2Client
Twitter Bootstrap
Requests
passlib
bcrypt

### Setup mod_wsgi for the App

Following is the app structure:

[var]
|-----[www]
|----------[app]
|---------------[catalog]
|------------------------[instance]         - contains app config files  
|------------------------[project]    
|--------------------------------[api]      - contains api related files  
|--------------------------------[catalog]  - contains catalog related files  
|--------------------------------[template] - contains html files  
|--------------------------------[users]    - contains user related files  
|--------------------------------[db]       - contains model, db files  
|--------------------------------[static]   - contains static files (css/js/images..)/ upload folder
|------------------------__init__.py        - contains run script  
|----------------catalog.wsgi               - wsgi file for this application

- In order to make this app run, we create the catalog.wsgi file:
```
# catalog.wsgi

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/app/catalog/")

from project import app as application

```
- Also, we need to modify few lines of code to connect database properly
Since we are using PostgreSQL, we need to change:
```
engine = create_engine('sqlite:///catalog.db')
```

from models.py and db_helper.py in DB folder to:

```
engine = create_engine('postgresql://catalog:<password>@localhost/catalog')
```

so that we can access database using PostgreSQL.

### Google OAuth2 Setting

- Since the app is supporting Google OAuth2 authentication,
'Authorized javascript origin' and 'redirect url' need to be updated properly.
The Lightsail url can be used for both entries.


### Apache2 Configure

A vhost configuration needs to be created to serve the app properly.
The file needs needs to be created in '/etc/apache2/sites-available' folder.
The config file for this app is 'catalog.conf'

Inside of the config file, we have:
```
<VirtualHost *:80>
    ServerAdmin test@site.com

    # Our server name           
    ServerName http://35.167.201.154/
    ServerAlias

    # Create error/access logs in '/var/www/app/log/' folder
    ErrorLog /var/www/app/logs/error.log
    CustomLog /var/www/app/logs/access.log combined

    # Reload service whenever this file is modified
    WSGIScriptReloading On

    # This file executes when users connect the app using the ip or url
    #WSGIScriptAlias / /var/www/app/catalog/catalog.wsgi

    # This app runs under www-data user in www-data group
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/app/catalog.wsgi

    <Directory /var/www/app/>
        Require all granted
    </Directory>

    # Setting an alias for the static folders
    Alias /static/ /var/www/app/catalog/project/static/

    <Directory /var/www/app/catalog/project/static/>
	     Require all granted
    </Directory>

    <DirectoryMatch /\.git/>
        AllowOverride None
        Require all denied
    </DirectoryMatch>
</VirtualHost>
```

After the above steps, use following code to make our app enable:
```
 $ sudo a2ensite catalog
```
Then, we need to restart apache service to apply any changes:
```
 $ sudo apache2ctl restart
```

Now users can connect the application using
- http://35.167.201.154/
- http://ec2-35-167-201-154.us-west-2.compute.amazonaws.com

I just initiated category list. Logged in users can add, edit or delete their items.
