# Project Description

This project takes a baseline Amazon Lightsail Linux Server, and prepares it to host your own web applications.
1. Configuring the machine to serve as an Apache web server
2. Securing the web server from a number of attack vectors
3. Installing and configuring a Postgresql database server
4. Deploying a Flask-Python web application onto it


### Public IP for the web application: 34.210.193.162


### Public URL for the web application: http://ec2-34-210-193-162.us-west-2.compute.amazonaws.com



## Obtaining an Amazon Lightsail instance


1. Create an AWS account


![alt text](aws_login.jpg)


2. Create a Lightsail instance. Choose OS Only, and Ubuntu.


![alt text](ubuntu_instance.jpg)


3. Give the instance a hostname, and start up the instance. It may take a few minutes.

4. Once you have an instance, you will be given a public IP address. Write it down.



## SSH access to the instance


1. Download your SSH private key from your AWS Lightsail Account page. It should end with .pem


![alt text](ssh_key.jpg)


2. Open a command prompt on your local computer, then move the private key file into ~/.ssh folder. For example:

```
$ mv ~/MyFolder/Lightsailkey.pem ~/.ssh
```

3. Grant access to the key file

```
$ chmod 400 ~/.ssh/Lightsailkey.pem
```

4. SSH into the server instance from your local computer. Replace YourPublicIP with your real IP given by Amazon.

```
$ ssh -i ~/.ssh/Lightsailkey.perm ubuntu@YourPublicIP
```



## Change port from 22 to 2200


1. On your Amazon Lightsail instance page, find Networking tab. Under Firewall, add Custom, TCP, 2200.

![alt text](firewall.jpg)

2. On your command prompt, as you still login as ubuntu, modify sshd_config file to accept Port 2200.

```
$ sudo nano /etc/ssh/sshd_config
```

Change Port 22 to Port 2200, and save the change (Ctrl+X, Y, Enter).

Then restart the SSH service:

```
$ sudo service ssh restart
```

3. Now try to SSH into the instance using Port 2200

```
$ ssh -i ~/.ssh/Lightsailkey.pem -p 2200 ubuntu@YourPublicIP
```



## Disable SSH for root user


```
$ sudo nano /etc/ssh/sshd_config
```

Find the PermitRootLogin, and change it to no and save.

Then restart the SSH service:

```
$ sudo service ssh restart
```



## Configure firewall on the instance


1. Still login as ubuntu, first check the firewall status. It should be inactive.

```
$ sudo ufw status
```

2. Configure firewall

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow ssh
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw deny 22
```

Note: Please be very careful here. Make sure to allow ssh and 2200 BEFORE you deny 22 and enable the firewall so you don't lose your ssh access to your instance.

3. Enable firewall. After enabling, you can check the status again.

```
$ sudo ufw enable
$ sudo ufw status
```



## Create a new user grader for the instance


1. Still login as ubuntu, Add the grader user

```
$ sudo adduser grader
```

You will be asked for a password. Enter it.

2. Grant sudo to grader

```
$ sudo nano /etc/sudoers.d/grader
```

Type grader ALL=(ALL:ALL) NOPASSWD:ALL, then save the file. (Ctrl+X, Y, Enter)



## Make key based authentication for the new user grader


1. Start a new command prompt on your local computer (Not login to the AWS instance), generate a SSH key pair.

```
$ ssh-kengen
```

Enter the file in which to save the keys. For example: /c/Users/MyName/.ssh/udacity

2. In the command prompt for the AWS instance, if you already login as ubuntu, login as grader use password

```
$ su - grader
```

Enter password for grader when you created the grader user. Now, you are logged in as grader.

3. Copy the public key for grader from your local machine onto the AWS instance.

```
$ mkdir .ssh
$ touch ./ssh/authorized_keys
$ nano .ssh/authorized_keys
```

From the command prompt for your local machine, get the public key file

```
$ cd ~/.ssh
$ cat udacity.pub
```

Right click on the content of the .pub file, select, and copy.

Back to the command prompt for the AWS instance (logged in as grader), paste the .pub content into the newly created authorized_keys file, then save the file. (Ctrl+X, Y, Enter)

4. Grant access to .ssh folder and authorized_keys file.

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

5. Force key based authentication for grader

While logged in as grader

```
$ sudo nano /etc/ssh/sshd_config
```

Change PasswordAuthentication to no, then save the change. (Ctrl+X, Y, Enter)

6. Then restart the SSH service:

```
$ sudo service ssh restart
```



## Configure timezone to UTC


```
$ sudo dpkg-reconfigure tzdata
```

Select None of the above, then UTC.



## Update currently installed packages on the AWS instance


```
$ sudo apt-get update
$ sudo apt-get upgrade
```



## Install Apache and WSGI on the AWS instance


```
$ sudo apt-get install apache2
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
```

Note: if your application is in Python 3, then install wsgi for python 3

```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi-py3
```

Then restart the Apache service:

```
$ sudo service apache2 restart
```



## Install and configure PostgreSQL on the AWS instance


1. Install PostgreSQL

```
$ sudo apt-get install postgresql
```

2. Check no remote connection allowed in pg_hba.conf file (9.5 is the PostgreSQL version, yours may be different.)

```
$ cat /etc/postgresql/9.5/main/pg_hba.conf
```

3. Create database user catalog and database catalog (replace CreatePassword with your real password)

```
$ sudo su - postgres
$ psql
postgres=# create user catalog with password 'CreatePassword';
postgres=# create database catalog owner catalog;
postgres=# \q
postgres@ip-xxx-xx-x-xx:~$ exit
```



## Install Git and clone your web application on the AWS instance


1. Install git

```
$ sudo apt-get install git
```

2. Clone your web application (In this case, I am cloning my catalog project. The url is your github url for the cloned project)

```
$ cd /var/www
$ sudo mkdir FlaskApp
$ cd FlaskApp
$ sudo git clone https://github.com/mathildaj/Catalog.git catalog
```

3. Make .git unaccessible from the browser

In /var/www/FlaskApp directory:

```
$ nano ~/.htaccess
```

Enter RedirectMatch 404 /\.git, then save (Ctrl+X, Y, Enter)

Copy the .htaccess from home directory to FlaskApp directory

```
$ sudo mv ~/.htaccess .htaccess
```



## Setup your web application to run on the AWS instance


1. In /var/www directory:

```
$ sudo chown -R ubuntu:ubuntu FlaskApp
```

1. In  /var/www/FlaskApp directory:

```
$ cd catalog
$ sudo mv views.py __init__.py
```

2. Change the database engine to run on PostgreSQL instead of SQLite:

In /var/www/FlaskApp/catalog directory:

```
$ sudo nano __init__.py
```

Then change engine = create_engine('sqlite:///catalog.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog'), and save.

Make the same change to database engine on database_setup.py, and database_init.py.

3. Make sure __init__.py file has the correct Google Client ID and Secret. They are needed for Google Oauth login.
   You can also use a separate client_secret.json file to store the Google credentials.

4. Install all the necessary packages for the app:

Go back to the home directory: (cd ~)

```
$ sudo apt-get install python-pip
$ sudo pip install sqlalchemy
$ sudo pip install Flask
$ sudo pip install httplib2
$ sudo pip install requests
$ sudo pip install oauth2client
$ sudo pip install Flask-OAuthlib
$ sudo pip install --user flask-oauthlib
$ sudo pip install psycopg2
```

5. Run the initial database setup:

```
$ cd /var/www/FlaskApp/catalog
$ sudo python database_setup.py
$ sudo python database_init.py
$ sudo pip install httplib2
$ sudo pip install requests
$ sudo pip install oauth2client
$ sudo pip install Flask-OAuthlib
$ sudo pip install --user flask-oauthlib
```



## Create a virtual host on the AWS instance


1. Create a configuration file for your website

```
$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

Add the following to the file, and save. (replace your_public_ip with your real IP, and your_aws_instance_url with your reaal URL. For example: ec2-xx-xxx-xxx-xxx.us-west-2.compute.amazonaws.com)

```
<VirtualHost *:80>
	ServerName Your_Public_IP
	ServerALias Your_AWS_instance_URL
	ServerAdmin admin@Your_Public_IP
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

2. Enable the virtual host and disable the default:

```
$ sudo a2ensite FlaskApp
$ sudo a2dissite 000-default.confSite 000-default disabled
$ sudo service apache2 reload
```


## Create the WSGI file for the web application

In /var/www/FlaskApp directory:

```
$ sudo nano flaskapp.wsgi
```

Enter the following to the file, and save.

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Your_secret_key'
```


## Restart Apache web server and run the application

```
$ sudo service apache2 restart
```

Open a browser, and type:

http://ec2-xx-xxx-xxx-xxx.us-west-2.compute.amazonaws.com

Now the application is up and running!

## License

None

## Acknowledgements

* https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps


