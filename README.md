# FSND P7 - Linux Server Configuration
Linux Server


## Requirements
* AWS Lightsail
* Catalog App - <a href="https://github.com/Dongs7/FSND_Catalog">FSND_Catalog</a>

## Project Detail

### Server Information

Created a static IP and attached to the instance

IP Address : 35.167.201.154
URL : http://ec2-35-167-201-154.us-west-2.compute.amazonaws.com
SSH port : 2200

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

### Install Apache and Python mod_wsgi

- Install Apache
```
 $ sudo apt-get install apache2
```

- Install Python mod_wsgi
```
 $ sudo apt-get install libapache2-mod-wsgi
```
