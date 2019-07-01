# Linux Server Configuration:

Installation of a Linux distribution on a virtual machine and prepare it to host your web application(Item Catalog). It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

* The EC2 URL is : `http://ec2-13-235-119-8.ap-south-1.compute.amazonaws.com/`
* Local IP address: `http://13.235.119.8/`
* SSH port- `2200`

* Login with: `ssh grader@13.235.119.8 -p 2200 -i ~/.ssh/project3`

## Configuration Steps:
### Step 1 : Launch your Virtual Machine with your Udacity account
* Development Environment Information Details:-
  * Public IP Address -  13.235.119.8
  * Private Key - Can't be shared

### Step-2: Follow the instructions provided to SSH into your server
* `mv ~/Downloads/udacity_key.rsa ~/.ssh/`
* `chmod 600 ~/.ssh/udacity_key.rsa`
* `ssh -i ~/.ssh/udacity_key.rsa root@13.235.119.8`

### Step-3 : Create a new user named grader
* `sudo adduser grader`
* To check the User(grader) information :
```
    sudo apt-get install finger
    finger grader
```
It is give you additional  information(login , name , shell, directory, phone number etc) of User-grader


### Step-4 : Give the grader the permission to sudo
* `sudo visudo`     (edit the sudoers file . it is save to use sudo visudo to edit the sudoers file otherwise file will not be saved)
* add the below line of code after root ALL=(ALL:ALL) ALL
	`grader ALL=(ALL:ALL) ALL` and save it (ctrl-X , then Y and Enter)
* Your new user(grader) is able to execute commands with administrative privileges. ( for example - sudo anycommand)
* You can check the grader entry by below command:
	`sudo cat /etc/sudoers`


### Step-5 : Update all currently installed packages
* `sudo apt-get update` - command will  update list of packages and their versions on your machine.
* `sudo apt-get upgrade` - command will install the packages


### Step-6 : Change the SSH port from 22 to 2200
*  root@ip-172-31-16-246:~# `sudo nano /etc/ssh/sshd_config`
    * change port from `22` to `2200`
    * change `PermitRootLogin without-password` to `PermitRootLogin no` . it is disable root login.
    * change `PasswordAuthentication` from no to yes.
    * add `AllowUsers grader` at end of the file so that we will login through grader.
* restart the SSH service :
    `sudo service ssh restart`

### Step-7 : SSH- Keys
* generate key-pair with ssh-keygen
* Save keygen file into (/home/user/.ssh/project3).and fill the password . 2 keys will be generated,  public key (project3.pub) and identification key(project3).
* Login into grader account using `ssh -v grader@"public_IP_address" -p 2200`.  type the password that you have fill during user creation
  (`sudo adduser grader` step 3) .
  anum@anum:~$ `ssh -v grader@13.235.119.8 -p 2200`
  `grader@ip-172-31-16-246 password :`
* if the password is correct , you will login as grader account:
    `grader@ip-172-31-16-246:~$`
* make a directory in grader account : `mkdir .ssh`
* make a authorized_keys file using `touch .ssh/authorized_keys`
* from your local machine,copy the contents of public key(linuxProject.pub).
* paste that contents on authorized_keys of grader account using `sudo nano authorized_keys` and save it .
* give the permissions : `chmod 700 .ssh`    and `chmod 644 .ssh/authorized_keys`.
*  do `sudo nano /etc/ssh/sshd_config` , change `PasswordAuthentication` to  no .
* `sudo service ssh restart`.
*  `ssh grader@13.235.119.8 -p 2200 -i ~/.ssh/project3` in new terminal .A pop-up window will open for authentication. just fill the password that you have fill during ssh-keygen creation.

**Resources -** [initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04), [udacity course videos](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4331066009/concepts/48010894750923#)

### Step-8:Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
* check the firewall status using `sudo ufw status`.
* block all incoming connections on all ports using `sudo ufw default deny incoming`.
* allow outgoing connections on all ports using `sudo ufw default allow outgoing`.
* allow incoming connection for SSH(port 2200) using `sudo ufw allow 2200/tcp`.
* allow incoming connection for HTTP(port 80) using `sudo ufw allow 80/tcp`.
* allow incoming connection for NTP(port 123) using `sudo ufw allow 123/udp`.
* check the added rules using `sudo ufw show added`.
* enable the firewall using `sudo ufw enable`.
* check whether firewall is enable or not using `sudo ufw status`.

**Resources -** [UFW](https://help.ubuntu.com/community/UFW)

### Step-9: Configure the local timezone to UTC
* configure timezone using `sudo dpkg-reconfigure tzdata` ( select none of the above and then set timezone to UTC)

**Resources -** [timezone to UTC](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)

### Step 10: Install and configure Apache to serve a Python mod_wsgi application:
* install apache using s`udo apt-get install apache2` .
* type `13.235.119.8` (public IP address) on URL . You will see the apache ubuntu default page .
* Install mod_wsgi using `sudo apt-get install libapache2-mod-wsgi` .
* You then need to configure Apache to handle requests using the WSGI module. You’ll do this by editing the /`etc/apache2/sites-enabled/000-default.conf` file. This file tells Apache how to respond to requests, where to find the files for a particular site and much more.
* add the following line at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> line: `WSGIScriptAlias / /var/www/html/myapp.wsgi`

* restart Apache with the `sudo service apache2 restart` command.
* To test if you have your Apache configuration correct you’ll write  a very basic WSGI application.Create the /var/www/html/myapp.wsgi file using the command `sudo nano /var/www/html/myapp.wsgi`. Within this file, write the following application:
```
    def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
```
*  This application will simply print return Hello World! along with the required HTTP response headers. After saving this file you can reload `http://13.235.119.8/` to see your application run in all its glory!

**Resources -** [install apache using linux course videos](https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4340119836/concepts/48189486140923#), [install apache](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)


### Step 11: Install Git
*  Install git using `sudo apt-get install git`
* set up git using :
```
        git config --global user.name "username"
        git config --global user.email "email@domain.com"
```
* check the configurations items using `git config --list`

**Resources -** [install git](https://help.github.com/articles/set-up-git/) , [install git on ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04)

### Step 12: Deploy Flask Application:

This include **six** steps :

#### step-1
* WSGI (Web Server Gateway Interface) is an interface between web servers and web apps for python. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. So the first step to install python-dev (mod-wsgi is already installed )
`sudo apt-get install python-dev`
* To enable mod_wsgi, run `sudo a2enmod wsgi`.

#### step-2
* move to the `/var/www` directory:
* Create the application directory structure using mkdir
    `sudo mkdir catalog`
* Move inside this directory : `cd catalog`
* Create another directory : `sudo mkdir catalog`
* move inside this directory and create two subdirectories named static and templates:
    `cd catalog`
    `sudo mkdir static templates`
* create the __init__.py file that will contain the flask application logic.
    `sudo nano __init__.py`
* Add following logic to the file:
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, everyone!"
if __name__ == "__main__":
    app.run()
```
close and save the file.

#### step-3
* Now , we will create a virtual environment for our flask application.  use pip to install virtualenv and Flask. Install pip :
    `sudo apt-get install python-pip`
* Install virtualenv:   `sudo pip install virtualenv`
* Set enviornment name using : `sudo virtualenv venv`
* Install Flask in that environment by activating the virtual environment using : `source venv/bin/activate`
* Install Flask using : `sudo pip install Flask`
* Run the following command to test if the installation is successful and the app is running:  `sudo python __init__.py`
* It should display "Running on `http://127.0.0.1:5000/"`. If you see this message, you have successfully configured the app.
* To deactivate the environment : `deactivate`

#### step-4
* Run - `sudo nano /etc/apache2/sites-available/catalog.conf`
* configure the virtual host adding your Servername:
```
    <VirtualHost *:80>
        ServerName 13.235.119.8
        ServerAdmin admin@13.235.119.8
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
 Save and close the file.
 * Enable virtual host using :  `sudo a2ensite catalog`


#### step-5
 * Create the wsgi file using:  `cd /var/www/catalog`
 * `sudo nano catalog.wsgi`  and add the code :
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```
* Directory structure should be :
```
|--------catalog
|----------------catalog
|-----------------------static
|-----------------------templates
|-----------------------venv
|-----------------------__init__.py
|----------------catalog.wsgi

```

#### step-6
* Restart Apache :  `sudo service apache2 restart`

**Resources -** [Install flask, Virtual Env](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### Step-13 : Clone the Github Repository
* `sudo mv Item-Catalog_ND-Project /var/www/catalog/catalog/`
* move the Item-Catalog_ND-Project directory to `/var/www/catalog/catalog`.
* To make github repository inaccessible  make a .htaccess file in `/var/www/catalog`.
* paste the content - `RedirectMatch 404 /\.git` in this file and save it .
* You can delete unwanted files in your folder (for example - readme, vagrant folder etc) and your folder should look like :
```
grader@ip-172-31-16-246:/var/www/catalog$ ls
catalog  catalog.wsgi
grader@ip-172-31-16-246:/var/www/catalog$ cd catalog/
grader@ip-172-31-16-246:/var/www/catalog/catalog$ ls
catalog.db           database_setup.pyc      db_items.py  static     venv
client_secrets.json  database_setup.py  database_setup.py.save  __init__.py  templates

```

### Step-14: Install other Packages
* `sudo apt-get install python-pip`
* `source venv/bin/activate`
* `pip install httplib2`
* `pip install requests`
* `sudo pip install --upgrade oauth2client`
* `sudo pip install sqlalchemy`
* `pip install Flask-SQLAlchemy`
* `sudo pip install flask-seasurf`
* If you want to see what packages have been installed with your installer tools :  `pip freeze`

### Step 15:  Install Postgresql
*  Install the Python PostgreSQL adapter psycopg:
        sudo apt-get install python-psycopg2
* Install PostgreSQL:
    `sudo apt-get install postgresql postgresql-contrib`
* To check, no remote connections are allowed :
        sudo vim /etc/postgresql/9.3/main/pg_hba.conf
* open database_setup.py using :
    `sudo nano database_setup.py`
* update the create_engine line:
	`python engine = create_engine('postgresql://catalog:catalog-pw@localhost/catalog')`
* Update the create_engine line in project.py and lotsofmenus.py too.
* move the project.py file to __init__.py file :
    mv application.py __init__.py
* Change to default user postgres:
    `sudo su - postgre`
* Connect to the system:
    `psql`
* Create user catalog:
    `CREATE USER catalog WITH PASSWORD 'catalog-pw';`
* check lists of roles using` \du`
* Allow the user to create database :
    `ALTER USER catalog CREATEDB;`
  and check the roles and attributes using \du.
* Create database using :
   ` CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to database using :
       ` \c catalog`
* Revoke all the rights :
    `REVOKE ALL ON SCHEMA public FROM public;`
* Grant the access to catalog:
    `GRANT ALL ON SCHEMA public TO catalog;`
* Once you execute database_setup.py , again you can login as psql and check all the tables with following commands:
	* connect to database using : `\c catalog`
	* To see the tables in schema :` \dt`
	* to see particular table:` \d [tablename]`
	* to see the entries/data in table :`select * from [tablename];`
	* to drop the table:`drop table [tablename];`
* exit from Postgresql : `\q `then `exit `from postgresql user.
* restart postgresql: `sudo service postgresql restart`

**Resources -** [Install postgresql](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) , [engine configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql)

### Step 16: Run the application:
* Create the database schema:
    `python database_setup.py`
    `python db_items.py`
* Restart Apache : `sudo service apache2 restart`
* in /var/www/catalog/catalog directory : execute -  `python __init__.py`
* type  public IPaddress (`http://13.235.119.8/`) on URL and you will see your Tile Catalog Webpage.


* related to client_secrets.json and fb_client_secrets.json files. You need to give absolute path to these files .
change the ```CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']``` to
    ```CLIENT_ID = json.loads(
    open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']```
    Similarly for `fb_client_secrets.json` file.

* check your errors in /var/log/apache2/error.log files.
         `tail -10 /var/log/apache2/error.log` to see last 10 lines of file.

* Make sure after you recorrect your error , restart the apache2 server.

**Resources -** [Udacity Discussion Forum](https://discussions.udacity.com/t/client-secret-json-not-found-error/34070) , [forum post]( https://discussions.udacity.com/t/how-to-login-to-my-aws-virtual-server-as-new-user-grader/201164/4).

### Step 17: Oauth Login
* go to [hcidata](http://www.hcidata.info/host2ip.cgi) and get the host name of public IP address (13.235.119.8).
    (IP Address) 13.235.119.8 = (Host Name) ec2-35-165-147-241.us-west-2.compute.amazonaws.com
* `sudo nano /etc/apache2/sites-available/catalog.conf` and add the hostname below ServerAdmin:
    paste `ServerAlias ec2-35-165-147-241.us-west-2.compute.amazonaws.com`
*  enable the virtual host : `sudo a2ensite catalog`
* restart the apacheserver : `sudo service apache2 restart`.

* Google Authorization steps:
  * Go to [console.developer](https://console.developers.google.com/)
  * click on Credentails --> edit
  * add you hostname (http://ec2-35-165-147-241.us-west-2.compute.amazonaws.com  ) and public IP address (http://13.235.119.8) to Authorised   JavaScript origins.
  * add hostname (http://ec2-35-165-147-241.us-west-2.compute.amazonaws.com/oauth2callback) to Authorised redirect URIs.
  * update the client_secret.json file too(adding hostname and public IP address).


**Resources -** [Udacity Discussion Forum](https://discussions.udacity.com/t/setting-facebook-valid-oauth-redirect-uris/160617) , [Forum post](https://discussions.udacity.com/t/oauth-course-google-sign-in-doesnt-work/15444/2).
