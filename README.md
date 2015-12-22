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

### 4 - Configure key-based authentication for the *grader* user

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

**Notes**: It doesn't make much sense to use *fail2ban* when the ssh key-based authentication is enforced. But it is still useful for other things, like smtp/imap-logins.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), [Reddit](https://www.reddit.com/r/linuxadmin/comments/2lravs/fail2ban_does_not_detect_my_ssh_privatekey/).

### 10 - Configure cron scripts to automatically manage package updates

1. Install *unattended-upgrades* if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. To enable it, do: '$ sudo dpkg-reconfigure --priority=low unattended-upgrades'.