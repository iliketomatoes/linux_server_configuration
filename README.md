# Udacity - Linux server configuration project

## Create a new user named "grader" and grant this user sudo permissions.

1. Log into the remote VM as *root* user through ssh.
2. `$ adduser grader`.
3. Create a new file under the suoders directory: `$ nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.

## Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.
3. Install *finger*, a utility software to check users' status: `$ apt-get install finger`.

## Configure the local timezone to UTC

1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
2. Install *ntp daemon ntpd* for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.

Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime).

## Configure key-based authentication for the *grader* user

1. Generate an encryption key *on your local machine* with: `$ ssh-keygen -f ~/.ssh/udacity_grader`.
2. Log into the remote VM as *root* user through ssh and create the following file: `$ touch /home/grader/.ssh/authorized_keys`.
3. Copy the content of the *udacity_grader.pub* file from your local machine to the */home/grader/.ssh/authorized_keys* you just created on the remote VM.
	1. Change permission: `$ sudo chmod 700 /home/grader/.ssh`.
	2. Change permission: `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
	3. Change owner: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_grader grader@52.34.208.247`.

## Enforce key-based authentication
1. Edit *sshd_config*: `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and set it to *no*.
2. `sudo service restart`.

## Change the SSH port from 22 to 2200
1. Edit *sshd_config*: `$ sudo nano /etc/ssh/sshd_config`. Find the *Port* line and set it to *2200*.
2. `sudo service restart`.
3. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_grader -p 2200 grader@52.34.208.247`.

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013).

## Disable ssh login for *root* user
1. Edit *sshd_config*: `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and set it to *no*.
2. `sudo service restart`.

Source: [Askubuntu](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server).

## Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw allow 2200/tcp`.
2. `$ sudo ufw allow 80/tcp`.
3. `$ sudo ufw allow 123/udp`.
4. `$ sudo ufw enable`.