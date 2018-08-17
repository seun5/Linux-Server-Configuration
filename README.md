# Linux Server Configuration
Udacity Full Stack Web Developer Nano Degree Project #6

This project will use the item catalog application that was created in a previous project and use AWS Linux virtual machine to host to website online


## IP & SSH Port
IP Address: 54.244.205.68
SSH port: 2200

## Application URL
http://ec2-54-244-205-68.us-west-2.compute.amazonaws.com/



## Tasks

### Configuration of UFW

Set up the configurations specified by the rubric for the project. 
Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

'''
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow http
sudo ufw allow ntp
sudo ufw enable
'''
After Setting up the firewall, check by using

'''sudo ufw status'''

### Change SSH Port to 2200

Edit file '/etc/ssh/sshd_config' using your preferred text editor to change port from 22 to 2200
'''
sudo vim /etc/ssh/sshd_config
sudo service ssh restart
'''

### Update installed packages

Update and upgrade current packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install postgresql postgresql-contrib
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2 
sudo apt-get install git

```


### Add user grader and allow sudo 

'sudo adduser grader'
'sudo usermod -aG sudo grader'

### Set-up SSH keys for user grader

The ssh key is given in the notes to grader

'ssh -i [ssh key] -p 2200 grader@54.244.205.68'

### Disable Root Login
'sudo vim /etc/ssh/sshd_config'
Change line 
PermitRootLogin without-password to PermitRootLogin no

### Set Up Database
'sudo -u postgres createuser -P catalog'
'sudo -u postgres createdb -O catalog catalog'

### Clone Item Catalog App from Github

Clone the repository from Github using clone command and make .git inaccessable
'''
cd /var/www
sudo mkdir catalog
git clone https://github.com/seun5/Finance-Buddy.git catalog
sudo vim .htaccess
'''
Add line RedirectMatch 404 /\.git

### Adjust app to prepare for Deployment

Rename app to __init.py
'sudo mv application.py __init__.py'

Change engine from sqlite to postgresql for 'model.py' and '__init.py'
'engine = create_engine('postgresql://catalog:DB-PASSWORD@localhost/catalog')'


#### Update Google OAuth
- Any reference to 'client_secrets.json' is now '/var/www/catalog/catalog/client_secrets.json'
- Add the URL and IP address in the javascript_origins field in the Google Developers Console settings 
- Make sure that the client id in 'template/login.html' is the same as the client id in 'client_secrets.json'

#### Create WSGI File
Create a empty WSGi File
'sudo vim /var/www/catalog/catalog.wsgi'
Copy the Following
'''
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/fullstack-nanodegree-vm/catalog/catalog/")

from catalog import app as application

application.secret_key = 'YOUR_SECRET_KEY'
'''

#### Configure Apache2

'sudo nano /etc/apache2/sites-available/catalog.conf'


'''
<VirtualHost *:80>
		ServerName http://54.244.205.68
		ServerAlias HOSTNAME ec2-54-244-205-68.us-west-2.compute.amazonaws.com/
		ServerAdmin admin@54.244.205.68
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
'''


Launch your app
'''
sudo a2dissite 000-default.conf
sudo a2ensite catalog.conf
sudo service apache2 restart
'''


## References:

- Udacity Discussion Forums on the project
- [Flask Application on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [Grader SSH Access Folder Permission](https://www.digitalocean.com/community/questions/error-permission-denied-publickey-when-i-try-to-ssh)
- [Add Sudo User](https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart)
