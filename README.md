# FSND-Project-Linux-Server-Configuration

Students will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host their web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

##Settings
* IP Address: 34.199.55.49
* SSH Port: 2200
* URL: [http://ec2-34-199-55-49.compute-1.amazonaws.com/](http://ec2-34-199-55-49.compute-1.amazonaws.com/)
* Logging In:
  1. Save private key to your local machine in the `~/.ssh` folder
    * `mv ~/DIRECTORY_WHERE_KEY_IS DOWNLOADED/linuxServerConfiguration.rsa` `~/.ssh/`
  2. SSH into server:
    * `ssh grader@34.199.55.49 -p 2200 -i ~/.ssh/linuxServerConfiguration.rsa`


## Secure the server
1. Update all currently installed packages
   * `sudo apt-get update`
   * `sudo apt-get dist-upgrade`
2. Remove unnecessary packages
   * `sudo apt-get autoremove`
3. Change the SSH port from 22 to 2200
   * `sudo nano /etc/ssh/sshd_config`
    * Change the line below _What ports, IPs and protocols we listen for_ from Port 22 to Port 2200
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


## Set timezone to UTC
1. Configure the local timezone to UTC
   * `sudo  timedatectl set-timezone Etc/UTC` (Note: By default timedatectl syncs the time once on boot and later on uses socket activation to recheck once network connections become active.)


## Configure catalog database
   1. Install PostgreSQL
      * `sudo apt-get install postgresql postgresql-contrib`
   2. Switch to postgres user and open psql prompt.
      * `sudo -u postgres psql postgres`
   3. Create a user named catalog with a password:
      * `CREATE USER catalog WITH PASSWORD 'YOUR_PASSWORD';`
   4. Allow catalog user the ability to login
      * `ALTER ROLE catalog WITH LOGIN;`
   5. Give a user the ability to create new databases
      * `ALTER USER catalog CREATEDB;`
   6. Create a database named catalog
      * `CREATE DATABASE catalog WITH OWNER catalog;`
   7. Revokes the ability to create objects for all users
      * 'REVOKE ALL ON SCHEMA public FROM public;'
   8. Grant privileges to catalog user
      * `GRANT ALL ON SCHEMA public TO catalog;`
   9. Disconnect from psql
      * `\q`
   10. Restart postgres
      * `sudo service postgresql restart`


## Install and configure Apache to serve item catalog application
Reference: [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
   1. Install Apache using your package manager with the following command:
      * `sudo apt-get install apache2`
   2. Install mod_wsgi:
      * `sudo apt-get install libapache2-mod-wsgi python-dev`
   3. Enable mod_wsgi with the following command:
      * `sudo a2enmod wsgi`
   4. Move to the /var/www directory:
      * `cd /var/www`
   5. Create the application directory structure using mkdir as shown:
      * `sudo mkdir catalog`
   6. Move inside this directory using the following command:
      * `cd catalog`
   7. Clone git repository for item-catalog project:
      * `sudo git clone https://github.com/jefferygraham992/item-catalog.git`
   8. Move to the catalog project directory:
      * `cd NameOfCatalogProjectDirectory`
   9. Rename project.py to __init__.py
      * `mv project.py to __init__.py`
   10. Set up a virtual environment. This will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations
      1. Install *pip*, in order to install *virtualenv*
         * `sudo apt-get install python-pip`
      2. Install *virtualenv*:
         * `sudo pip install virtualenv`
      3. Create a virtual environment *venv*
         * `sudo virtualenv venv`
      4. Activate virtual environment
         * `source venv/bin/activate`
      5. Install necessary packages to run application:
         * `sudo pip install Flask`
         * `sudo pip install sqlalchemy`
         * `sudo pip install psychopg2`
         * `sudo pip install oauth2client`
         * `sudo pip install requests`
      6. Test to see if the installation is successful and the app is running: (*It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app.*)
         * `sudo python __init__.py`
      7. To deactivate the environment, give the following command:
         * `deactivate`
   11. Configure the virtual host:
      1. `sudo nano /etc/apache2/sites-available/catalog.conf`
   12. Add the following lines of code to the file to configure the virtual host. Be sure to change the ServerName to your domain or cloud server's IP address:
   ```
   <VirtualHost *:80>
      ServerName IP address of your server
      ServerAdmin Your email address
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
   13. Enable the virtual host:
      * `sudo a2ensite catalog`
   14. Move to the /var/www/catalog directory and create a file named catalog.wsgi
      * `cd /var/www/catalog`
      * `sudo nano catalog.wsgi`
   14. Add the following lines of code to the catalog.wsgi file:
   ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalog/")

      from catalog import app as application
      application.secret_key = 'your secret key'
   ```
   15. Restart Apache:
      * `sudo service apache2 restart `


## Populate database
Reference: [Engine Configuration¶](http://docs.sqlalchemy.org/en/latest/core/engines.html)
   1. Change the line creating the database to use postgres instead of sqlite in the `database_setup.py`, `__init__.py` and `thomastrains.py`
      * `engine = create_engine('sqlite:///thomascatalog.db')` becomes `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
   2. Use the full path to `client_secrets.json` and `fb_client_secrets.json` in the `project.py` file
   3. Setup the database
      * `sudo python database_setup.py`
   4. Populate the database
      * `sudo python thomastrains.py`
   5. Run the application
      * `sudo python __init__.py`


## Update OAuth in google and facebook's developer settings.
   * Go to Google Developer Console and set javascript origins and redirect URLS using the hostname.
   * Go to facebook developer apps, select this app and set the URL in both basic and advanced settings.
*Note:* The url for the application can be found by running: `nslookup <IP addess of server>`


## Extra steps to harden Ubuntu server
Reference: [How to secure an Ubuntu 16.04 LTS server](https://www.thefanclub.co.za/how-to/how-secure-ubuntu-1604-lts-server-part-1-basics)
1. Secure shared memory
   Shared memory can be used in an attack against a running service, so it is always best to secure that portion of memory. You can do this by modifying the /etc/fstab file.
   1. Open the /etc/fstab file.
      * `sudo nano /etc/fstab`
   2. Add the following line to the bottom of that file:
      * `tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0`
   3. Save and close the file. Reboot the server for changes to take place:
      * `sudo reboot`

2. Add security banner
   1. Edit the `/etc/issue.net` file
      * `sudo nano /etc/issue.net`
   2. Edit the file to add a suitable warning.
   3. Disable the banner message from motd
      1. Open the `/etc/pam.d/sshd` file
         * `sudo nano /etc/pam.d/sshd`
      2. Comment out the following two lines
         * `session optional pam_motd.so motd=/run/motd.dynamic`
         * `session optional pam_motd.so noupdate`
   4. Edit the `/etc/ssh/sshd_config` file
      * `sudo nano /etc/ssh/sshd_config`
      * Uncomment the following line: `Banner /etc/issue.net`
   5. Restart the ssh server
      * `sudo service ssh restart`

3. Harden Apache SSL - disable SSL v2/v3 support
   * The SSL v2/v3 protocol has been proven to be insecure.
   * Disable Apache support for the protocol & force the use of the newer protocols
   1. Open `/etc/apache2/mods-available/ssl.conf`
      * `sudo nano /etc/apache2/mods-available/ssl.conf`
   2. Change this line `SSLProtocol all -SSLv3` to this `SSLProtocol all -SSLv2 -SSLv3`
   3. Restar Apache server
      `sudo service apache2 restart`

4. Harden the networking layer
   Prevent source routing of incoming packets (and log all malformed IPs) on your Ubuntu Server
   1. Open `/etc/sysctl.conf` file
      * `sudo nano /etc/sysctl.conf`
   2. Uncomment or add the following lines:
   ```
   # IP Spoofing protection
   ​net.ipv4.conf.all.rp_filter = 1
   ​net.ipv4.conf.default.rp_filter = 1
   ​
   ​# Ignore ICMP broadcast requests
   ​net.ipv4.icmp_echo_ignore_broadcasts = 1
   ​
   ​# Disable source packet routing
   ​net.ipv4.conf.all.accept_source_route = 0
   ​net.ipv6.conf.all.accept_source_route = 0
   ​net.ipv4.conf.default.accept_source_route = 0
   ​net.ipv6.conf.default.accept_source_route = 0
   ​
   ​# Ignore send redirects
   ​net.ipv4.conf.all.send_redirects = 0
   ​net.ipv4.conf.default.send_redirects = 0
   ​
   ​# Block SYN attacks
   ​net.ipv4.tcp_syncookies = 1
   ​net.ipv4.tcp_max_syn_backlog = 2048
   ​net.ipv4.tcp_synack_retries = 2
   ​net.ipv4.tcp_syn_retries = 5
   ​
   ​# Log Martians
   ​net.ipv4.conf.all.log_martians = 1
   ​net.ipv4.icmp_ignore_bogus_error_responses = 1
   ​
   ​# Ignore ICMP redirects
   ​net.ipv4.conf.all.accept_redirects = 0
   ​net.ipv6.conf.all.accept_redirects = 0
   ​net.ipv4.conf.default.accept_redirects = 0
   ​net.ipv6.conf.default.accept_redirects = 0
   ​
   ​# Ignore Directed pings
   ​net.ipv4.icmp_echo_ignore_all = 1
   ```
   3. Restart the service
      * `sudo sysctl -p`

5. Prevent IP Spoofing
   *Note: IP spoofing, also known as IP address forgery or a host file hijack, is a hijacking technique in which a cracker masquerades as a trusted host to conceal his identity, spoof a Web site, hijack browsers, or gain access to a network.*
   1. Open the `/etc/host.conf` file
      * `sudo nano /etc/host.conf`
   2. Add or edit the following lines:
      ```
      order bind,hosts
      nospoof on
      ```

6. Restrict Apache Information Leakage
   *Note: Information Leakage is an application weakness where an application reveals sensitive data, such as technical details of the web application, environment, or user-specific data. Sensitive data may be used by an attacker to exploit the target web application, its hosting network, or its users.*
   1. Edit the Apache2 configuration security file
      * `sudo nano /etc/apache2/conf-available/security.conf`
   2. Add or edit the following lines and save :
      ```
      ServerTokens Prod
      ServerSignature Off
      TraceEnable Off
      Header unset ETag
      Header always unset X-Powered-By
      FileETag None
      ```
   3. Restart Apache server.
      * `sudo service apache2 restart`

7. Install ModSecurity on your server
   *Note:*ModSecurity is a toolkit for real-time web application monitoring, logging, and access control.*
   1. Install the dependencies
      * `sudo apt-get install libxml2 libxml2-dev libxml2-utils`
      * `sudo apt-get install libaprutil1 libaprutil1-dev`
   2. Install ModSecurity
      * `sudo apt-get install libapache2-modsecurity`
   3. Verify ModSecurity was installed
      * `apachectl -M | grep --color security`
   4. Rename the recommended configuration file
      * `sudo mv /etc/modsecurity/modsecurity.conf{-recommended,}`
   5. Reload Apache server.
      * `sudo service apache2 reload`
   6. Configuring mod_security
      *Note: Out of the box, modsecurity doesn't do anything as it needs rules to work. The default configuration file is set to DetectionOnly which logs requests according to rule matches and doesn't block anything. This can be changed by editing the modsecurity.conf file:*
      1. Modify `modsecurity.conf` file
         * `sudo nano /etc/modsecurity/modsecurity.conf`
      2. Change this line `SecRuleEngine DetectionOnly` to this `SecRuleEngine On`
   7. Set up rules  
      *Note: *To make your life easier, there are a lot of rules which are already installed along with mod_security. These are called CRS (Core Rule Set)*
      1. Remove the default CRS .
         * `sudo rm -rf /usr/share/modsecurity-crs`
      2. Clone latest version of mod_security CRS
         * `sudo git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git /usr/share/modsecurity-crs`
      3. Rename the example setup file
         * `cd /usr/share/modsecurity-crs`
         * `mv crs-setup.conf.example crs-setup.conf`
      4. Enable these rules by configuring `/etc/apache2/mods-enabled/security2.conf` file
         * `sudo nano /etc/apache2/mods-enabled/security2.conf`
      5. Change the file as shown
         ```
         <IfModule security2_module>
            SecDataDir /var/cache/modsecurity
            IncludeOptional /etc/modsecurity/*.conf
            IncludeOptional "/usr/share/modsecurity-crs/*.conf"
            IncludeOptional "/usr/share/modsecurity-crs/rules/*.conf
        </IfModule>
         ```
      6. Restart apache service
         * `systemctl restart apache2`

8. Install ModEvasive on your server
   *Note: Mod_evasive is an Apache module that can be used to protect against various kinds of attacks on the Apache web server including DDoS, DoS and brute force*
   1. Install mod_evasive
      * `sudo apt-get install libapache2-mod-evasive`
   2. Verify installation
      * `sudo apachectl -M | grep evasive`
   3. Configure mod_evasive
      1. Enable mod_evasive by editing `evasive.conf` file
         * `sudo nano /etc/apache2/mods-enabled/evasive.conf`
         Make changes below:
         ```
         <IfModule mod_evasive20.c>
             DOSHashTableSize 3097
             DOSPageCount 2
             DOSSiteCount 50
             DOSPageInterval 1
             DOSSiteInterval 1
             DOSBlockingPeriod 60
             DOSEmailNotify <someone@somewhere.com>
         </IfModule>
         ```
   4. Create a log directory for mod_evasive
      * `sudo mkdir /var/log/mod_evasive`
      * `sudo chown -R www-data:www-data /var/log/mod_evasive`
   5. restart Apache
      * `sudo systemctl restart apache2`

9. Scan logs and ban suspicious hosts with Fail2Ban
   *Note: Fail2ban scans log files and bans IPs that show the malicious signs -- too many password failures, seeking for exploits, etc*
   1. Install Fail2ban
      * `sudo apt-get install fail2ban`
   2. Configuring Fail2Ban
      1. Configure `fail2ban.local`
         *Note: The file `fail2ban.conf` contains the default configuration profile. The default settings will give you a sane and working setup so this is the best place to start. If you want to make any changes, it’s best to do it in a separate file, `fail2ban.local`, which overrides `fail2ban.conf`.*
         1. Rename a copy `fail2ban.conf` to `fail2ban.local`
            * `sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local`
         2. Edit the definitions in `fail2ban.local` to match your desired configuration
      2. Configure `jail.local`
         *Note: The `jail.conf` file will enable Fail2ban for SSH by default for Ubuntu. All other protocols and configurations (HTTP, FTP, etc.) are commented out. If you want to change this, it’s recommended to create a `jail.local` for editing just like you did `with fail2ban.local`.*
         1. Rename a copy `jail.conf` to `jail.local`
            * `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
   3. Restart Fail2ban
      * `sudo service fail2ban restart`
   4. Chack the status:
      * `sudo fail2ban-client status`

10. Install PSAD Intrusion Detection
    *Note: psad is a collection of three lightweight system daemons (two main daemons and one helper daemon) that run on Linux machines and analyze iptables log messages to detect port scans and other suspicious traffic.*
   1. Download and install the latest version of PSAD
      ```
      sudo su
      mkdir /tmp/.psad
      cd /tmp/.psad
      wget http://cipherdyne.org/psad/download/psad-2.4.3.tar.gz
      tar -zxvf psad-2.4.3.tar.gz
      cd psad-2.4.3
      ./install.pl
      cd /tmp
      rm -R .psad
      exit
      ```
   2. Edit the PSAD configuration file
      * `sudo nano /etc/psad/psad.conf`
   3. Add iptables LOG rules for both IPv4 and IPv6
      *Note: Add rules that instruct iptables to log packets that are not part of legitimate traffic. Psad can be configured to only analyze those iptables messages that contain specific log prefixes (which are added via the --log-prefix option)*
      ```
      iptables -A INPUT -j LOG
      iptables -A FORWARD -j LOG
      ip6tables -A INPUT -j LOG
      ip6tables -A FORWARD -j LOG
      ```
   4. Reload and update PSAD
      ```
      psad -R
      psad --sig-update
      psad -H
      ```
      * Check the status of PSAD
        * `psad --Status`

11. Check for rootkits w/ RKHunter and CHKRootKit
    *Note: A rootkit is a collection of computer software, typically malicious, designed to enable access to a computer or areas of its software that would not otherwise be allowed (for example, to an unauthorized user) and often masks its existence or the existence of other software.*
   1. Install RKHunter and CHKRootKit
      * `sudo apt-get install rkhunter chkrootkit`
   2. Run chkrootkit
      * `sudo chkrootkit`
   3. Update and run RKHunter
      ```
      sudo rkhunter --update
      sudo rkhunter --propupd
      sudo rkhunter --check
      ```

12. Scan open ports with Nmap
    *Note: Nmap ("Network Mapper") is a free and open source utility for network discovery and security auditing*
   1. Install Nmap
      * `sudo apt-get install nmap`
   2. Scan your system for open ports
      * `sudo nmap -v -sT localhost`
   3. SYN scanning with the following (*SYN scanning is a tactic that a malicious hacker (or cracker) can use to determine the state of a communications port without establishing a full connection. This approach, one of the oldest in the repertoire of crackers, is sometimes used to perform denial-of-service (DoS) attacks*)
      * `sudo nmap -v -sS localhost`

13. Analyse system LOG files with LogWatch
    *Logwatch is a customizable log analysis system. Logwatch parses through your system's logs and creates a report analyzing areas that you specify.*
    1. Install Logwatch
       * `sudo apt-get install logwatch libdate-manip-perl`
    2. View log files
       * `sudo logwatch | less`


## Author

* **Jeff Graham** - *Initial work* - [jefferygraham992](https://github.com/jefferygraham992)
