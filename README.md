# Linux Server Configuration Project
For this project, we take a baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, and installing/configuring web and database servers.

## Useful Information:
* IP Address: 18.220.229.44
* SSH Port: 2200
* Application Link: http://ec2-18-220-229-44.us-east-2.compute.amazonaws.com

## Instructions:
#1. Update all currently installed packages:
* ```sudo apt-get update```
* ```sudo apt-get upgrade```
* Install finger: ```sudo apt-get install finger```

#2. Set the Timezone:
* Run: ```sudo timedatectl set-timezone UTC```
* To verify, run: ```timedatectl status```

#3. Change the SSH port from 22 to 2200 and configure the Lightsail firewall to allow it
* Run ```sudo nano /etc/ssh/sshd_config```
* Look for the Port line and edit it to: 2200
Run ```sudo service ssh restart```

#4. Configure Firewall Settings
* Check the status of the firewall by running: ```sudo ufw status```
* ```sudo ufw allow 2200/tcp```
* ```sudo ufw allow 80/tcp```
* ```sudo ufw allow 123/udp```
* ```sudo ufw enable```
* ```sudo ufw status```

#5. Add a new user named grader and enable sudo command:
* Run ```sudo adduser grader```
* Run ```sudo nano /etc/sudoers.d/grader```
* Write the text: ``` grader ALL=(ALL:ALL) ALL``` in the /etc/sudoers.d/grader file

#6. Configure key-based authentication for grader user
* Go back to your local machine and create an encryption key by running: ```ssh-keygen -f ~/.ssh/udacity_key.rsa```
* Open the public key file and copy its content
* Now return to your remote machine and connect to the server via ssh
* Switch user by running: su grader
* Create a new directory called ```.ssh``` in the home directory of grader by running: ```mkdir ~/.ssh```
* Create a new file by running: ```sudo nano ~/.ssh/authorized_keys```
* Insert the content of the public key file
* Ensure that this folder has the correct permissions by running: ```chmod 700 ~/.ssh && chmod 600 ~/.ssh/*```
* Also make sure that key-based authentication is enforced by running: ```sudo nano /etc/ssh/sshd_config```
* Look for the line: PasswordAuthentication yes
* Change it to: PasswordAuthentication no

#7. Disable ssh login for root user
* Run: ```sudo nano /etc/ssh/sshd_config```
* Find the PermitRootLogin line and edit it to ```no```
* Run: ```sudo service ssh restart```

#8. Install Apaache
* ```sudo apt-get install apache2```

#9. Install mod_wsgi and git
* ```sudo apt-get libapache2-mod-wsgi python-dev```
* ```sudo apt-get install git```
* Enable mod: ```sudo a2enmod wsgi```
* Now run: ```sudo service apache2 start```

#10: Install Git and clone project
* Create new directory: ```sudo mkdir /var/www/catalog```
* Set the owners of the files: ```sudo chown -R grader:grader /var/www/catalog```
* Clone project to catalog: ```cd /var/www/catalog```
* ```git clone https://github.com/christopherjeon/item-catalog-project.git catalog```

#11. Create a .wsgi file
* ```sudo nano catalog.wsgi```
* Add the following:

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```
#12. Rename the main file
* ```cd catalog```
* ```sudo mv catalog_project.py __init__.py```

#13. Install virtual environment
* Install pip by running: ```sudo apt-get install python-pip```
* Being with installing virtual environment: ```sudo pip install virtualenv```
* Create new virtual environment: ```sudo virtualenv venv```
* Activate: ```source venv/bin/activate```
* ```sudo chmod -R 777 venv```

#14. Install Flask and other required packages
* Install Flask by running: ```pip install Flask```
* For other packages:```pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils```

#15. Change path of client_secrets.json__
* Edit by running: ```nano __init__.py```
* Change the client_secrets.json path to ```/var/www/catalog/catalog/client_secrets.json```

#16. Configure and enable a new virutal host
* ```sudo nano /etc/apache2/sites-available/catalog.conf```
* Paste this code:

```
<VirtualHost *:80>
    ServerName 18.220.229.44
    ServerAlias ec2-18-220-229-44.us-east-2.compute.amazonaws.com
    ServerAdmin admin@18.220.229.44
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
#17. Install and configure PostgreSQL
* ```sudo apt-get install libpq-dev python-dev```
* ```sudo apt-get install postgresql postgresql-contrib```
* ```sudo su - postgres```
* ```psql```

#18. Create and Set Up database
* ```CREATE USER catalog WITH PASSWORD 'create your password here';```
* ```ALTER USER catalog CREATEDB;```
* ```CREATE DATABASE catalog WITH OWNER catalog;```
* Connect to your database: ```\c catalog```
* ```REVOKE ALL ONE SCHEMA public TO catalog;```
* Log out from PostgreSQL: ```\q```
* Now ```exit```
* Change create engine line in your Python files to: engine = create_engine(‘postgresql://catalog:YOUROASSWORDHERE@localhost/catalog')

#19. Initialize your Python files:
* For me, I initialized my Python files like this:
* ```python lotsofsports.py```
* ```python catalog_database.py```
* ```python __init__.py```

#20. Restart Apache
* Run: ```sudo service apache2 restart```
* Visit: http://ec2-18-220-229-44.us-east-2.compute.amazonaws.com

## Acknowledgments and References:
* https://github.com/callforsky/udacity-linux-configuration
* https://github.com/rrjoson/udacity-linux-server-configuration
* https://github.com/iliketomatoes/linux_server_configuration
