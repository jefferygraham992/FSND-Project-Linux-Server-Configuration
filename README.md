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
    * Change the line below `# What ports, IPs and protocols we listen for` from Port 22 to Port 2200
4. Restart the sshd service by running the following command:
   * `sudo service sshd restart`
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
   * `sudo ufw default deny incoming`
   * `sudo ufw default allow outgoing`
   * `sudo ufw allow 2200/tcp`
   * `sudo ufw allow www`
   * `sudo ufw allow ntp`
   * `sudo ufw enable`



### Settings


## Author

* **Jeff Graham** - *Initial work* - [jefferygraham992](https://github.com/jefferygraham992)
