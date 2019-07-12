# Linux Server Configuration'

This project is part of Udacity's [Full Stack Web Developer Nano Degree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). The project takes a baseline installation of a Linux server to host your web applications while securing, installing and configuring a database server.

The project ip: [3.93.39.142](http://3.93.39.142/)

The project url: [http://3.93.39.142.xip.io/](http://3.93.39.142.xip.io/)

## Tools

- [Amazon Lightsail](https://aws.amazon.com/lightsail)
- The [Catalog Project](https://github.com/arrionc/catalogApp) from the previous section in the course
- [PostgreSQL](https://www.postgresql.org/)

## Setting up an Instance in Amazon Lightsail

- Login to Amazon Lightsail
- Click `Create Instance`
- Select Platform `Linux/Unix` 
- Select a blueprint `OS Only`
- Select `Ubuntu 16.04 LTS`
- Choose your instance plan - choose the cheapest one
- Give a name to your instance 
- Create Instance

## Set up the SSH Key
- Download the default private key from your Lightsail Account Page.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`
- In your local terminal run the following command `chmod 600 ~/.ssh/lightsail_key.rsa`
- Connect to the instance via the terminal `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@3.93.39.142`

## Change SSH port from 22 to 2200
- Edit the sshd_config file by running `sudo nano /etc/ssh/sshd_config`
- Change port from 22 to 2200
- Save the change by Control+X and confirm with Y
- Restart SSH with sudo service ssh restart 

## Configure Uncomplicated Firewall (UFW)
- The firewall for Ubuntu needs to only allow incoming connections for SSH(port 2200), HTTP(port 80) and NTP (port 123).
- Run the following commands: 
```
sudo ufw status         # the status should be inactive.
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp # Allow incoming tcp packets on port 2200.
sudo ufw allow www      # Allow HTTP traffic in.
sudo ufw allow 123/udp  # Allow incoming udp packets on port 123.
sudo ufw deny 22        # Deny incoming requests on port 22 sudo ufw enable         #Turn on UFW
sudo ufw status         #UFW is now active and should list all the allowed ports  
```
- Exit the SSH connection, type `exit`
- Go to the AWS Lightsail instance and in the Networking tab edit the Firewall rules:
```
Application        Protocol      Port range
HTTP                TCP           80
Custom              UDP           123
Custom              TCP           2200
```

- After enabling ufw and opening ports 80, 2200 and 123 run:
`ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@3.93.39.142`


## Create user grader and give sudo access 
- Create user run `sudo adduser grader`
- Create a new directory in sudoer directory run `sudo nano /etc/sudoers.d/grader`
- Add `grader ALL=(ALL:ALL) ALL` in nano editor
- Save and exit using CTRL+X and confirm with Y 
- Verify the grader has sudo permissions. Run `su -grader` and then `sudo -l` 
- The grader should have ALL permissions.

## Create SSH key pair for grader 
 - In your local terminal run `ssh-keygen`
 - Copy the generated ssh file with extension .pub
 - Login to the virtual machine as the grader `su - grader `
 - Create directory `mkdir .ssh`
 - Create file `touch .ssh/authorized_keys`
 - Run `sudo nano ~/.ssh/authorized_keys` and paste the contents from the .pub file into this file.
 - Save and exit 
 - Give permissions `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
 - Check `/etc/ssh/sshd_config` file make sure `PasswordAuthentication`is set to no
 - Restart `sudo service ssh restart`
 - Check `sudo nano /etc/ssh/sshd_config` find `PermitRootLogin` and change it to `no`
- On the local machine run `ssh grader@3.93.39.142 -p 2200 -i ~/.ssh/linuxProject`

## Update all packages

- Run `sudo apt-get update` and `sudo apt-get upgrade`

## Set up Local Time Zone 
- While logged in as grader, configure the time zone: `sudo dpkg-reconfigure tzdata`

## Install Apache and wsgi module 
- Run `sudo apt-get install apache2` to install apache
- Run `sudo apt-get install python-setuptools libapache2-mod-wsgi-py3` to install mod-wsgi module
- Start the server `sudo service apache2 start`
- Enable `mod-wsgi` run `sudo a2enmod wsgi`

## Install Git and clone the catalog project
- Run `sudo apt-get install git`
- Configure your username and email run `git config --global user.name <yourusername>` and `git config --global user.email <youremail>`
- Clone the project run `cd /var/www and sudo mkdir catalog`
- Change the owner to grader `sudo chown -R grader:grader catalog`
- Run `sudo chmod catalog` to give a permission to clone the project.
- Switch to the catalog directory and clone the Catalog project.
- `cd catalog` and `git clone https://github.com/arrionc/catalogApp.git`
- Create catalog.wsgi file.
- Run `sudo nano catalog.wsgi` and add the following code:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalogApp/")
sys.path.insert(1, "/var/www/catalog/")

from catalogApp import app as application
application.secret_key = '...'
```

## Install and configure PostgreSQL
- While logged in as grader, install PostgreSQL: `sudo apt-get install postgresql`

- Check PostgreSQL does not allow remote connections. In the `/etc/postgresql/9.5/main/pg_hba.conf` file
- Switch to the postgres user: `sudo su - postgres`
- Open PostgreSQL interactive terminal with psql.
- Create the catalog user with a password and give them the ability to create databases run the following commands:
```
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'yourownpassword';
postgres=# ALTER ROLE catalog CREATEDB;
```
- You can check the created role by running `\du` and the catalog role should be listed with an attribute to `Create DB`

- Exit psql: `\q`

- Switch back to the grader user type `exit` in the terminal
- Create a new Linux user called catalog: `sudo adduser catalog`
- Enter password and fill out information.
- Give to catalog user the permission to sudo. Run: `sudo visudo`
- Search for the lines that looks like this:
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
- Give catalog sudo permission by adding it to the file
```
catalog  ALL=(ALL:ALL) ALL
```
- Save and exit using CTRL+X and confirm with Y.
- Change user to catalog `su -catalog`
- While logged in as catalog, create a database: `createdb catalog`
- Run `psql` and then run `\l` to see that the new database has been created. 
- Exit psql: `\q`
- Switch back to the grader user type `exit` on your terminal

## Install virtual environment and Flask framework
- Install pip, `sudo apt-get install python3-pip`
- Run `sudo apt-get install python-virtualenv` to install virtual environment
- Create a new virtual environment with `sudo virtualenv -p python3 venv3`
- Change the ownership to grader with: `sudo chown -R grader:grader venv3/ `
- Activate the environment `. ven3/bin/activate`
- Install all needed dependencies:
```
pip3 install flask
pip3 instal sqlalchemy
pip3 install --upgrade oauth2client
pip3 install httplib2
pip3 install requests
sudo apt-get install libpq-ved
pip3 install psycopg2
``` 
- Deactivate virtual environment type `deactivate` in the terminal.

## Configure Apache
- Create a config file `sudo nano /etc/apache2/sites-available/catalog.conf`
- Add the following lines:
```
WSGIPythonHome "/var/www/catalog/catalogApp/venv3/bin/python3"
WSGIPythonPath "/var/www/catalog/catalogApp/venv3/lib/python3.5/site-packages"

<VirtualHost *:80>
    ServerName Ubuntu-1
    ServerAlias 3.93.39.142
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalogApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalogApp/static
    <Directory /var/www/catalog/catalogApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
```
- Disable Apache site `sudo a2dissite 000-default.conf`
- Reload Apache `sudo servie apache2 reload`
- Enable virtual host `sudo a2ensite catalog`
- Reload Apache `sudo service apache2 reload`

## Modify filenames to deploy on AWS.

- Rename `application.py` file to `__init__.py` by running `mv application.py __init__.py`
- In `__init__.py`, replace:
```
# app.run(host="0.0.0.0", port=8000, debug=True)
app.run()
```
- In the `database_setup.py` file, the `__init__.py` file and the `starterItems.py` file replace:
```
# engine = create_engine("sqlite:///winecatalog.db")
engine = create_engine('postgresql://catalog:yourownpassword@localhost/catalog')
```
- In the `/var/catalog/catalogApp directory`, activate the virtual environment `. venv3/bin/activate`
- Populate the database `python3 starterItems.py`
- Deactivate the virtual environment `deactivate`
- Restart Apache `sudo service apache2 reload`
- Open browser to `3.93.39.142`
- Application should be app and running 
- A useful command to catch errors in apache `sudo tail /var/log/apache2/error.log`

## Log in with Google OAuth
- It's important to change your settings for a successful login
- Go to your google cloud platform under `API & Services` go to `Credentials`
- Under `OAuth consent screen` and then under `Authorized Domains` add `xip.io`
- Go back to `Credentials` click on your app and change the `Authorized JavaScript Origins` to `http://3.93.39.142.xip.io`
- Change the `Authorized redirect URI's` to `http://3.93.39.142.xip.io/oauth2callback`
- Save and download the new JSON file 
- Open the JSON File copy its contents
- In the project directory `/var/www/catalog/catalogApp` edit the `client_secrets.json file` delete its contents and paste the contents from the new JSON file that was downloaded.
- Open `__init__.py` file and change
```
CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']
APPLICATION_NAME = "Item Catalog Application"
```
to
```
CLIENT_ID = json.loads(
    open('/var/www/catalog/catalogApp/client_secrets.json', 'r').read())['web']['client_id']
APPLICATION_NAME = "Item Catalog Application"
```
- Run `sudo service apache2 restart`
- Open browser to `http://3.93.39.142.xip.io`
- The Web App should now have google signin enabled

## References 
- [Stack Overflow](https://stackoverflow.com/)
- [How to Create a Server on Amazon Lightsail](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail
)
- [xip.io](http://xip.io/)
- GitHub Repositories
    - https://github.com/boisalai/udacity-linux-server-configuration
    - https://github.com/adityamehra/udacity-linux-server-configuration
    - https://github.com/aaayumi/linux-server-configuration-udacity
    







