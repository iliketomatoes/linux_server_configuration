# Udacity - Linux server configuration project

### 1 - Create a new user named *grader* and grant this user sudo permissions.

1. Log into the remote VM as *root* user through ssh.
2. `$ sudo adduser grader`.
3. Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.
4. In order to prevent the "sudo: unable to resolve host" error, edit the hosts:
	1. `$ sudo nano /etc/hosts`.
	2. Add the host: `127.0.1.1 ip-10-20-37-65`.

### 2 - Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.
3. Install *finger*, a utility software to check users' status: `$ apt-get install finger`.

### 3 - Configure the local timezone to UTC

1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
2. Install *ntp daemon ntpd* for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.

Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime).

### 4 - Configure the key-based authentication for *grader* user

1. Generate an encryption key *on your local machine* with: `$ ssh-keygen -f ~/.ssh/udacity_grader`.
2. Log into the remote VM as *root* user through ssh and create the following file: `$ touch /home/grader/.ssh/authorized_keys`.
3. Copy the content of the *udacity_grader.pub* file from your local machine to the */home/grader/.ssh/authorized_keys* file you just created on the remote VM. Then change some permissions:
	1. `$ sudo chmod 700 /home/grader/.ssh`.
	2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	3. Finally change the owner from *root* to *grader*: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_grader grader@52.34.208.247`.

### 5 - Enforce key-based authentication
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and edit it to *no*.
2. `sudo service ssh restart`.

### 6 - Change the SSH port from 22 to 2200
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *Port* line and edit it to *2200*.
2. `sudo service ssh restart`.
3. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_grader -p 2200 grader@52.34.208.247`.

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013).

### 7 - Disable ssh login for *root* user
1. `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit it to *no*.
2. `sudo service ssh restart`.

Source: [Askubuntu](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server).

### 8 - Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw allow 2200/tcp`.
2. `$ sudo ufw allow 80/tcp`.
3. `$ sudo ufw allow 123/udp`.
4. `$ sudo ufw enable`.

### 9 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

Install *fail2ban* in order to mitigate brute force attacks by users and bots alike.

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install fail2ban`.
3. We need the *sendmail* package to send the alerts to the admin user: `$ sudo apt-get install sendmail`.
4. Create a file to safely customize the *fail2ban* functionality: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` .
5. Open the *jail.local* and edit it: `$ sudo nano /etc/fail2ban/jail.local`. Set the *destemail* field to admin user's email address.

**Notes**: It doesn't make much sense to use *fail2ban* when the ssh key-based authentication is enforced. Though it is still useful for other things, like smtp/imap-logins.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), [Reddit](https://www.reddit.com/r/linuxadmin/comments/2lravs/fail2ban_does_not_detect_my_ssh_privatekey/).

### 10 - Configure cron scripts to automatically manage package updates

1. Install *unattended-upgrades* if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. To enable it, do: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`.

### 11 - Install Apache, mod_wsgi

1. `$ sudo apt-get install apache2`.
2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
3. `$ sudo service apache2 start`.

### 12 - Install Git

1. `$ sudo apt-get install git`.
2. Configure your username: `$ git config --global user.name <username>`.
3. Configure your email: `$ git config --global user.email <email>`.

### 13 - Clone the Catalog app from Github

1. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
2. Change owner for the *catalog* folder: `$ sudo chown -R grader:grader catalog`.
3. Move inside that newly created folder: `$ cd /catalog` and clone the catalog repository from Github: `$ git clone https://github.com/iliketomatoes/catalog.git catalog`.
4. Make a *catalog.wsgi* file to serve the application over the *mod_wsgi*. That file should look like this:

```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

5. Some tweaks were needed to deploay the catalog app, so I made a *deployment* branch which slightly differs from the *master*. Move inside the repository, `$ cd /var/www/catalog/catalog` and change branch with: `$ git checkout deployment`.

### 14 - Install virtual environment, Flask and the project's dependencies

1. Install *pip*, the tool for installing Python packages: `$ sudo apt-get install python-pip`.
2. If *virtualenv* is not installed, use *pip* to install it using the following command: `$ sudo pip install virtualenv`.
3. Move to the *catalog* folder: `$ cd /var/www/catalog`. Then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
4. Activate the virtual environment: `$ source venv/bin/activate`.
5. Install Flask: `$ sudo pip install Flask`.
6. Install all the other project's dependencies: `$ sudo pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`.
7. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).

### 15 - Configure and enable a new apache virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apach2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName  ec2-52-34-208-247.us-west-2.compute.amazonaws.com
    ServerAdmin admin@52.34.208.247
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Specifying what Python to use through the *WSGIDaemonProcess* line can save you from a big mess. In this case we are explicitly saying to use the virtual environment and its packages to run the application.

3. Enable the new virtual host: `$ sudo a2ensite catalog`.

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

### 16 - Install and configure PostgreSQL



