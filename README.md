### Linux Server Configuration

- This project involved configuring a Ubuntu Linux server instance on Amazon Lightsail and preparing it to host web applications.

#### Configuration details

#### Secure server

###### Update all currently installed packages
    Login to the server and execute the following commands to update packages installed
    - sudo apt-get update
    - sudo apt-get upgrade

---
   ###### Configure the Lightsail firewall
    Change the Port to 2200 in the sshd_config file using the following commands:
    sudo nano /etc/ssh/sshd_config````
    sudo service ssh restart
---
   ###### Configure the Uncomplicated Firewall
    - sudo ufw status (For checking status)
    - sudo ufw default deny incoming
    - sudo ufw default allow outgoing
    - sudo ufw allow ssh
    - sudo ufw allow 2200/tcp
    - sudo ufw allow www
    - sudo ufw allow ntp
    - sudo ufw enable

#### Add new user - grader
- sudo adduser grader
- sudo cat /etc/passwd (Confirm addition)
- nano /etc/sudoers.d/grader
- Add the text: grader ALL=(ALL) ALL
- Create an SSH key pair for grader on the local machine:
    ssh-keygen
- On the server, paste the public key from the earlier step in to the authorized_keys file:
    cd /home/grader/
    mkdir .ssh
    nano .ssh/authorized_keys
    chmod 600 .ssh/authorized_keys
    chmod 700 .ssh

#### Configure local timezone to UTC
Verified using the following command:
sudo dpkg-reconfigure tzdata

#### Prepare to deploy your project

   ###### Install Apache
    sudo apt-get install apache2

   ###### Install WSGI
    sudo apt-get install libapache2-mod-wsgi
    Follow the directions in the terminal
    sudo a2enmod wsgi

   ######  Install git
    sudo apt-get install git

   ######  Install Postgresql
    - sudo apt-get install postgresql
    - To not allow remote connections:
        - sudo cat /etc/postgresql/9.5/main/pg_hba.conf
        - Verify if the configuration table matches the following:
            local   all             postgres                                peer
            local   all             all                                     peer
            host    all             all             127.0.0.1/32            md5
            bhost    all             all             ::1/128                 md5
    - sudo -u postgres psql (Access psql without switching accts)
    - Create a new database and a DB user named catalog using the following commands:
        - CREATE USER catalog WITH PASSWORD 'catalog'\g
        - ALTER USER catalog CREATEDB\g
        - CREATE DATABASE catalog WITH OWNER catalog\g
        - REVOKE ALL ON SCHEMA public FROM public\g
        - GRANT ALL ON SCHEMA public TO catalog\g
        - \l (see list of Dbs)
        - SELECT usename FROM pg_user\g (verify list of users)

   ###### Install required python packages
    sudo apt-get install python-sqlalchemy
    sudo apt-get install python-requests
    sudo apt-get install python-oauth2client
    sudo apt-get install python-psycopg2
    sudo apt-get install python-flask
    
   ###### Setup Item Catalog project from the Github repository
   Clone the Catalog repository into /var/www/catalog/catalog folder

#### Deploy the Item Catalog project
- Update application.py,database_init.py and database_setup.py to use PostgreSQL instead of SQLite:
 engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
- Update the relative link for client_secret.json to absolute in application.py:
client_id = json.loads(open('/var/www/catalog/catalog/client_secret.json', 'r').read())['web']['client_id']

- Add new endpoint URLs for the server to enable Oauth on https://console.developers.google.com/. Generate a new updated client_secrets.json file.
- Add a new __init__.py file in /var/www/catalog/catalog folder with the following content:
    - from application import app

- Create tables and add data. In the project folder, run the following commands:
    - python database_setup.py
    - python database_init.py 
    
- Create a WSGI for the Catalog project
    - cd /var/www/catalog
    - sudo nano catalog.wsgi
    - Add the following content to the file:
    ~~~~
        #!/usr/bin/python
    
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")
        
        from catalog import app as application
    ~~~~
- Make a WSGI configuration for the Catalog project
    - sudo nano /etc/apache2/sites-available/catalog.conf
    - Add the following contents to the file:
    ~~~~
        <VirtualHost *:80>
            ServerName 34.207.86.0
            ServerAdmin your_email@live.com
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
	~~~~
- Enable Catalog configuration:
    - sudo a2ensite catalog
- Restart web server:
    - sudo service apache2 restart
- Access your application from the browser: http://34.207.86.0/catalog

#### List of third party resources used
Besides the Udacity course material for Configuring Linux Web Servers, I referred the following resources for the project: 
- https://help.ubuntu.com/community/UbuntuTime
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

