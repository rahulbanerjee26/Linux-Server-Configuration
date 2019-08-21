
  

<h1>Linux Web Server Configuration  </h1>

<h2> Nanodegree : Full Stack Web Developer, Udacity </h2>

## Description

 You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

<h2> Server Info </h2>
<b> IP Address: </b> 3.121.206.97 <br>
<b>URL:</b> http://ec2-3-121-206-97.eu-central-1.compute.amazonaws.com/
<b> SHH PORT </b> 2200



  <h2>CONFIGURATION STEPS:</h2>
  

  <h3> <u> Get your server </u> </h3>
	
**Setup Server**
 1. Go to <a href="https://lightsail.aws.amazon.com"> Amazon LightSail
    </a>
 
 2. Create an account/sign in to your existing account
 3. Click Create instance and choose `Linux/Unix`,`OS only`  and `Ubuntu 16.04LTS`
 4. Choose the cheapest Plan (3.5 USD/month) 
 5. Give the instance a host name
 6. Click Create and wait a few minutes for it to start up
 7.  Once the instance is created, ssh in and you'll be logged in as the ubuntu user.

**SSH into Server**

 8. Go to your account and click on `SSH Keys`. Download the default key. The file name will be similar to `LightsailDefaultKey-eu-central-1.pem`
 9. In the terminal Navigate to file's directory and restrict file permission by typing `chmod 600 LightsailDefaultPrivateKey-*.pem`
 10. Rename file to `lightsail_key.rsa`
 11. SSH into your instance by typing `ssh -i lightsail_key.rsa ubuntu@3.121.206.97` (Change IP address accordingly) 


  <h3> <u> Secure your server </u> </h3>
  
**Update all currently installed packages**

 12. Type the following: 

     `sudo apt-get update`
     `sudo apt-get upgrade`
    `sudo apt-get update && sudo apt-get dist-upgrade`




**Change the SSH port from *22* to *2200***

 13. Edit the `sshd_config` file by typing:  `sudo nano /etc/ssh/sshd_config`
 14. Search for the port number and change it from 22 to 2200
 15. Restart SSH with `sudo service ssh restart`
 
 **Configure the Uncomplicated Firewall (UFW)**
 

 16. Only allow incoming connections for `SSH (port 2200), HTTP (port 80), and NTP (port 123)`.
 17.  Check the ufw status `sudo ufw status`. It should be inactive. 
 18.  Deny all incoming request `sudo ufw default deny incoming`
 19.  Allow all outgoing requests `sudo ufw default allow outgoing`
 20.  Allow the required incoming requests: 
 21. `sudo ufw allow 2200/tcp` to allow ssh requests
 22. `sudo ufw allow 80/tcp` to allow http requests
 23. `sudo ufw allow 123/udp` to allow ntp requests 
 24. `sudo ufw deny 22`  to deny requests from port 22
 25. `sudo ufw enable` to set status to active
 26. `sudo ufw status` Check the status. It should look something like this 
	 

    Status: active
    	   To                         Action      From
        --                         ------      ----
        2200/tcp                   ALLOW       Anywhere
        80/tcp                     ALLOW       Anywhere
        123/udp                    ALLOW       Anywhere
        22                         DENY        Anywhere
        2200/tcp (v6)              ALLOW       Anywhere (v6)
        80/tcp (v6)                ALLOW       Anywhere (v6)
        123/udp (v6)               ALLOW       Anywhere (v6)
        22 (v6)                    DENY        Anywhere (v6)

 27. Edit the rules accordingly in Networking in your lightsail account 
 
 <h3> <u> Give Grader Access </u> </h3>
 

  **Create a new user account named `grader`** 
  

 28.  `sudo adduser grader`

 **Give `grader` the permission to `sudo`**
 

 29. sudo nano /etc/sudoers.d/grader
 30.  Type the following line `grader ALL=(ALL:ALL) ALL`

**Create an SSH key pair for `grader` using the `ssh-keygen` tool**
    
 31. In your local machine, start up git bash and type `ssh-keygen`. Set a file directory and view the contents of the .pub file using the cat command. Copy the content
 32. log in as grader and  create a folder ssh in home directory. `mkdir .ssh` 
 33. Create a file authorized_keys `touch .ssh/authorized_keys`
 34. Paste the content into the file and save it. `nano .ssh/authorized_keys`
 35. Restrict file permissions 
          
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
   

 36. To disable password based login/ disable root login: 
		

    sudo nano /etc/ssh/sshd_config
   Search for password based authentication and change it to no
   Search for PermitRootLogin and change it to no
   

    sudo service ssh restart
  

 37. `exit` and `ssh -i ~/.ssh/grader -p 2200 grader@3.121.206.97`

<h3> <u> Prepare to deploy your project. </u> </h3>

**Configure the local timezone to UTC**

 1. `sudo dpkg-reconfigure tzdata`   Choose your timezone
 
  **Install Apache to serve a Python mod_wsgi application**
  
 2. Install Apache `sudo apt-get install apache2`
	  Check your public IP and it should display a webpage. 
 3. `sudo apt-get install python-setuptools libapache2-mod-wsgi` Install python mod-wgsi module

**Install and Configure PostgreSql**

 4. Type `sudo apt-get install postgresql` to install postgresql
 5.  Log in as postgres `sudo su - postgres`
 6. Enter psql shell by typing `psql`
 7.  Type the following Commands in the shell
 8.   `CREATE DATABASE Catalog;`
 9.  `CREATE USER Catalog;`
 10. `ALTER ROLE catalog WITH PASSWORD 'grader';`
 11. `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
 12.  Quit the shell `\q`
 13. Exit postgres by typing `exit`



 
 **Install Git**
 
 1. Type `sudo apt-get install git` to install git
 2. Configure Email Id and Username:
 3.  `git config --global user.name <username>` 
 4.  `git config --global user.email <email>`
 5.  change directory to /var/www  `cd /var/www`
 6. Create  a new folder. `sudo mkdir catalogApp`
 7. cd into the folder `cd catalogApp` 
 
 **Clone and setup the project**
 1. clone the required repo `sudo git clone https://github.com/rahulbanerjee26/Item-Catalog.git` Rename file to catalogApp 
 2. Go into the repo using cd and   rename  `app.py`  to  `__init__.py`  `sudo mv app.py __init__.py` 
 3. In `__init__.py` and `databaseSetup.py` files, remove the sqlite connection and connect to postgresql database.  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

**Create and Configure a New Virtual Host**

 

 1. `sudo apt-get install python-virtualenv` to install the virtual environment
 2. cd into your repo and type `sudo virtualenv -p python3 venv3`
 3. `sudo chown -R grader:grader venv3/` to change permission
 4. `. venv3/bin/activate` to activate the environment
 5. Install Flask and other dependencies 
 6. `sudo apt-get install python-pip`
 7. `sudo pip install Flask`
 8. `sudo pip install sqlalchemy oauth2client httplib2 requests`
 9. `sudo apt-get install postgresql libpq-dev postgresql-client postgresql-client-common`
 10.`deactivate` 


 10. Add the following code in catalogApp.conf to edit: `sudo nano /etc/apache2/sites-available/catalogApp.conf`  Change info as required

    <VirtualHost *:80>
            ServerName 3.121.206.97
            ServerAlias http://ec2-3-121-206-97.eu-central-1.compute.amazonaws.com/
            ServerAdmin banerjeer2611@gmail.com
            WSGIScriptAlias / /var/www/catalogApp/catalogapp.wsgi
            <Directory /var/www/catalogApp/catalogApp/>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static /var/www/catalogApp/catalogApp/static
            <Directory /var/www/catalogApp/catalogApp/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

 11.  `sudo a2ensite catalogApp.conf`

 12.  Setup the dataBase `sudo python databaseSetup.py`

 13. Reload apache2 `sudo service apache2 reload`

**Create .wgsi file**

 13.  `sudo nano  /var/www/catalogApp/catalogapp.wsgi`
 14. Paste the following: 
	
 

    activate_this = '/var/www/catalogApp/catalogApp/venv3/bin/activate_this.py'
    with open(activate_this) as file_:
        exec(file_.read(), dict(__file__=activate_this))
    
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalogApp")
        from catalogApp import app as application
        application.secret_key = 'my_secret_key'


    
    

 15. Save the file 
 16. Go to `__init__.py`  and change the path of the `client_secrets.json` from relative to absolute
 17. Run `sudo service apache2 restart`
 18. If you get an error, `sudo tail /var/log/apache2/error.log`

<h3><u> Resources </u>  </h3>

 1. [https://www.fullstackpython.com/blog/postgresql-python-3-psycopg2-ubuntu-1604.html](https://www.fullstackpython.com/blog/postgresql-python-3-psycopg2-ubuntu-1604.html)
 2. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 3. 

<h3><u> Author</u>  </h3>

<b>Rahul Banerjee</b>

