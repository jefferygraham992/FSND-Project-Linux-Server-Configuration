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


   Reference: [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## Populate database
   1. Change the line creating the database to use postgres instead of sqlite in the `database_setup.py`, `__init__.py` and `thomastrains.py`
      * `engine = create_engine('sqlite:///thomascatalog.db')` becomes `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
   2. Use the full path to `client_secrets.json` and `fb_client_secrets.json` in the `project.py` file
   3. Setup the database
      * `sudo python database_setup.py`
   4. Populate the database
      * `sudo python thomastrains.py`
   5. Run the application
      * `sudo python __init__.py`


Reference: [Engine Configuration¶](http://docs.sqlalchemy.org/en/latest/core/engines.html)





### Settings


## Author

* **Jeff Graham** - *Initial work* - [jefferygraham992](https://github.com/jefferygraham992)
