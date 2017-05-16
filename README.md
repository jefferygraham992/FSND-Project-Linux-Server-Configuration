# FSND-Project-Neighborhood Map

Students will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host their web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Secure the server
1. Update all currently installed packages
   * `sudo apt-get update`
   * `sudo apt-get dist-upgrade`
2. Remove unnecessary packages
   * `sudo apt-get autoremove`
3. Change the SSH port from 22 to 2200
   * `sudo nano /etc/ssh/sshd_config`
    * Change the line below `*# What ports, IPs and protocols we listen for*` from Port 22 to Port 2200
4. Restart the sshd service by running the following command:
   * `sudo service sshd restart`
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
   * `sudo ufw default deny incoming`
   * `sudo ufw default allow outgoing`
   * `sudo ufw allow 2200/tcp`
   * `sudo ufw allow www`
   * `sudo ufw allow ntp`
   * `sudo ufw enable`

## Give grader access
1. Create a new user account named grader
   * `sudo adduser grader`
2. Give grader the permission to sudo
   * `sudo usermod -aG sudo grader`

## Create an SSH key pair for grader using the ssh-keygen tool
1. Change to grader account
   * `su – grader`
2. Create the key pair on your local machine with the following command:
   * `ssh-keygen` (Save the file to default directory for key pairs)
3. Create a directory on the server to store keys
   * `mkdir .ssh`
4. Create a file to store keys
   * `touch .ssh/authorized_keys`
5. Edit the authorized_keys file with your favorite text editor and paste the public key for your key pair into the file
   * On local machine: `cat .ssh/<NameofFileContainingPublicKey>.pub` and copy key info
   * On the server: `nano .ssh/authorized_keys` & paste the public key for your key pair into the file
6. Change the file permissions of the .ssh directory to 700 (this means only the file owner can read, write, or open the directory).
   * `chmod 700 .ssh`
7. Change the file permissions of the authorized_keys file to 644 (this means only owner can write, others can read).
   * `chmod 644 .ssh/authorized_keys`




### Settings


## Author

* **Jeff Graham** - *Initial work* - [jefferygraham992](https://github.com/jefferygraham992)
