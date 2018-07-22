# Linux-Server-Configuration

### About
This repo was created for the configuration of linux server and to host a web application. The application uses **python3** and **postgresql** for the web app, **OAuth2** for authentiaction and **Lightsail** for hosting the server.

The application can be viewed at [http://54.71.218.188] or [http://ec2-54-71-218-188.us-west-2.compute.amazonaws.com].

The server can be access using the SSH key as the grader user at port 2200 using the following command.
`ssh grader@54.71.218.188 -p 2200`



### Configuration changes

1. The application uses [Amazon Lightsail](https://lightsail.aws.amazon.com/) for hosting the web server. Download the default private key to your local system and paste it in `~/.ssh/`. Log in remotely from your local system using SSH.


2. Once you have logged into the server, update all currently installed packages and the security updates.
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist upgrade
```

3. Enable the firewall so that it allows only **HTTP** ,**NTP** and **SSH** connections.

- Enable ports *2200* and *123* in the Lightsail portal.

- Configure the firewall so as to accept the connections.
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw allow 22
```

- Enable the firewall on the server
`sudo ufw enable`


4. Make changes to the SSH configuration file.

- Change the default port of SSH to 2200 in the `/etc/ssh/sshd_config` file by editing line 5 to `Port 2200`.
`Port 2200`

- Change the value of PermitRootLogin to *no* to deny remote root access
`PermitRootLogin no`

- Restart the SSH service to put the changes into effect.
`service ssh restart`

- Also block any requests that come on port 22 by denying it on the firewall.
`sudo ufw deny 22`


5. Create a new user **grader**.
``adduser grader``


6. Give grader the permission to sudo.
- Navigate to */etc/sudoers.d* directory and create a file *grader* with the following contents.
`grader ALL=(ALL) NOPASSWD:ALL`


7. Create an SSH key pair for passwordless login for the user grader.
- On your local machine, as the *grader* user, create a key pair using the following command. Put default names for the file names.
`ssh-keygen`

- Navigate to `~/.ssh/` to view the public and private keys generated.

- Copy the contents of public key to the server using the following command.
`ssh-copy-id 54.71.218.188`

- Log in to the server passwordless as *grader*.
`ssh -p 2200 -i ~/.ssh/id_rsa grader@54.71.218.188`

8. Deploy the project
- Install the packages for apache2 and mod_wsgi.
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3.
sudo apt-get install postgresql
```
- Enable mod_wsgi
`sudo a2enmod wsgi`

- Install Postgresql
`sudo apt-gett install postgresql`

- Create a new user *catalog* that has limited permissions to the database and give him sudo access
`sudo adduser catalog` 

- Create a new database *catalog*
`createdb catalog`

- Install git
`sudo apt-get install git`

9. Deploy the Catalog Item project
- Clone the project to var/www/ directory
`sudo git clone https://github.com/boisalai/udacity-catalog-app.git catalog`

- Rename `application.py` to `__init.py__`
`mv application.py __init__.py`

- Inside `__init__.py` change the code that runs the application to `app.run()`

- Inside *database.py* change the engine created to
`engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')`

10. Change the authorized redirect URI and authorized Javascript origins in the [Google Cloud Platform](console.developers.google.com) and update the *client_secrets.json* with the new data.

11. Install a virtual environment
- Install the package
`sudo apt-get install python-virtualenv`

- Inside */var/www/catalog/catalog/* create the virtual environment and give ownership to grader.
```
sudo virtualenv -p python3 venv3
sudo chown -R grader:grader venv3/
```

- Activate the new environment
`. venv3/bin/activate`

- Install pip3 and following dependencies using pip3.
`sudo apt-get install python3-pip`
```
pip install httplib2
pip install requests
pip install oauth2client
pip install sqlalchemy
pip install flask
pip install psycopg2
```

- Check if the application runs successfully locally on the environment
`python3 __init__.py`

- Deactivate the virtual environment
`deactivate`

12. Configuring virtual host
- Add the following entry in */etc/apache2/mods-enabled/wsgi.conf* to use python3
`WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages`

- Add the following details to */etc/apache2/sites-available/catalog.conf*
```
<VirtualHost *:80>
    ServerName 54.71.218.188.xip.io
    ServerAlias http://ec2-54-71-218-188.us-west-2.compute.amazonaws.com
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

- Enable the virtual host
`sudo a2ensite catalog`

- Disable the default host
`sudo a2dissite 000-default.conf`

- Reload apache2
`sudo service apache2 reload`


13. Configuring Flask
- Add the following entry in */var/www/catalog/catalog.wsgi*
```
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
```
- Restart apache2
`sudo service apache2 restart`

14. Populating the Database

- Add the following lines to *lotsofmenus.py*
```
import sys
sys.path.insert(0, "/var/www/catalog/catalog/venv3/lib/python3.5/site-packages") 
```

- Activate the virtual environment and run *lotsofmenus.py* and then deactivate the virtual environment.

15. Change the ownership of directories and restart apache2.
```
sudo chown -R www-data:www-data catalog/
sudo service apache2 restart
```


### Helpful Articles
- [Deploy Flask App](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [boisalai/udacity-linux-server-configuration](https://github.com/boisalai/udacity-linux-server-configuration.git)
- [kongling893/Linux-Server-Configuration-UDACITY](https://github.com/boisalai/udacity-linux-server-configuration.git)
- [Security updates](https://www.digitalocean.com/community/questions/updating-ubuntu-14-04-security-updates)
- [Secure Postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
