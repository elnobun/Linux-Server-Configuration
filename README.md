# Linux-Server-Configuration-FSND-P5
Linux distribution on a virtual machine, prepared to host web applications, install updates and securing it from a number of attack vectors.

This Project is in fulfilment of the Project 5 [Udacity Full Stack Web Developer Nanodegree Program](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). It deploys [Project 3](https://github.com/elnobun/Item-Catalog-Movie-Collection-App-/tree/master/vagrant) "Item Catalog - Movie Collection App" on the virtual machine, and is reacheable under the AWS - server at http://ec2-52-39-26-86.us-west-2.compute.amazonaws.com.

This Project is managed by Ellis Enobun

* [Project Access](#project-access)
* [Project Overview](#project-overview)
* [Installed Packages](#installed-packages)
* [Configuration Summary](#configuration-summary)
* [Project Step by Step Walkthrough](#project-step-by-step-walkthrough)

## Project Access
#### IP address

**`52.39.26.86`**

#### SSH Port

**`2200`**

#### `grader` password and passphrase:

**`Udacity268`**

#### Web Application URL

http://52.39.26.86/  

* AWS-Server: 
http://ec2-52-39-26-86.us-west-2.compute.amazonaws.com
 
## Project Overview 

I configured a secured remote virtual machine to host a databse server, and web appllications which are data driven. This was accomlished through a provided Linux distribution by [Udacity](https://www.udacity.com), and an installed Apache2 server to serve Python Flask application that connects to PostgresSQL database.

Result: Unregistered user page, running on http://52.39.26.86
![public_page](https://cloud.githubusercontent.com/assets/15114201/14767906/7ff35b5a-0a21-11e6-84c5-cd0f2336e635.png)
Registered user can edit or delete data entry. Page running on http://52.39.26.86/collection/1/movie/
![private_page](https://cloud.githubusercontent.com/assets/15114201/14767907/822fd9a2-0a21-11e6-9c28-cf0c242e113c.png)

## Installed Packages

Package Name | Description
--------------: | :------------
**finger:** | Displays an easy to read information about a user
**apache2** | HTTP Server
**libapache2-mod-wsgi** | hosts Python applications on Apache2 server
**ntp** | Synchronizes time over a network
**postgresql** | Postgresql Database server
**git** | Version control system tools
**python-setuptools** | An easy-install package to facilitate installing Python packages
**sqlalchemy** | ORM and SQL tools for Python
**flask** | Microframework for web applications
**python-psycopg2** | PostgreSQL adapter for Python
**oauth2** | Authorization framework for third-party login (Google and Facebook)
**google-api-python-client** | Google API for OAuth login
**fail2ban** | Protection against suspicious site activity by IP banning
**Glances** | Application monitor for host bugs

## Configuration Summary

- Setup Virtual Machine and SSH into the server.
- A new system user `grader` was created with permission to sudo.
- All cuurently installed packages were updated and upgraded.
- CRON tasks added to `update` and `upgrade` installed packages.
- Changed SSH Port from `22` to `2200` and configure SSH access.
- Configured `UFW`to only allow incoming connections for `SSH(Port:2200)`, `HTTP(Port:80)` and `NTP(Port:123)`.
- Configured local Time Zone to `UTC`.
- Installed and configure `Apache` to serve a `Python mod_wsgi` application.
- Installed Git and Setup Environment for delopying Flask Application.
- Install and configure `PostgreSQL` with default settings to *not* allow remote connection.
- Created a new user `catalog`, added user to PostgreSQL databse with limited permissions to catalog application database.
- Get OAUTH-LOGINS (Google+ and Facebook) working.
- Installed and Configured `Fail2ban` intrusion protection that bans suspicious IPs.


----------


## Project Step By Step Walkthrough:


----------


### A - Development Environment: Setup Virtual Machine and SSH into the server
Reference: [Udacity](https://www.udacity.com/account#!/development_environment)

1. Hit "Create new development environment".

2. Take note of your `Public IP address`, and download `private key` at the bottom.

3. Move the private key file into `~/.ssh` folder with this command in Terminal:
    
    ```
    $ mv ~/Downloads/udacity_key.rsa ~/.ssh
    ```
    
    where ~ is your environment's home directory). If you downloaded the file to the downloads folder, just execute the above command in your terminal.

4. Allow owner the right to "read" and "write" the file:

    ```
    $ chmod 600 ~/.ssh/udacity_key.rsa
    ```
    
5. SSH into the instance:

    ```        
    $ ssh -i ~/.ssh/udacity_key.rsa  root@YOUR-PUBLIC-IP-ADDRESS 
    ```


----------


### B - User Management: Create a new user with the permission to sudo.
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

*Note:* In some cases, you would have to use the `sudo` commnand to be able to perform some functions depending on the administrative priviledge of your local computer. 

1. Create a new user: `grader`:

    ``` 
    root@ip-10-20-30-101:~# adduser grader
    ```
    
    OPTIONAL: You can confirm that the new user has been created by installing `finger`.
    
    ``` 
    root@ip-10-20-30-10:~# apt-get install finger
    root@ip-10-20-30-10:~# finger grader
   ```
   
2. Give the new user sudo permission:

    (a) - Open sudoer configuration using visudo command:
    
    ```
        root@ip-10-20-30-10:~# visudo
    ```
    A nano editor will open up. scroll down to the line that reads:

	```
		root ALL=(ALL:ALL) ALL
   ```
    (b) -  Below that line, add the new user: `grader` like so:
    
    ```
        root ALL=(ALL:ALL) ALL
        grader ALL=(ALL:ALL) ALL
    ```
    Save changes by pressing `ctrl+x, y` then Enter key.  
    
    (c.) -  You can list all the users present root.
    
    ```
        root@ip-10-20-30-10:~# cut -d: -f1 /etc/passwd
    ```


----------


### C - Update and upgrade currently installed package

1. Update all available packages: This will provide a list of packages to be upgraded.

    ```
     root@ip-10-20-30-10:~# sudo apt-get update
    ```
    
2. Upgrade packages to newrer versions:
    
    ``` 
    root@ip-10-20-30-10:~# sudo apt-get upgrade
    ```
    
3. Add CRON script to manage `update` and `upgrade` installed packages.

    (a) -  Install unattended-upgrade packages:
    
    ```
        root@ip-10-20-30-10:~# sudo apt-get install unanttended-upgrades
    ```
    (b) -  Enable the unattended-upgrade packages:
  
    ```
       root@ip-10-20-30-10:~# sudo dpkg-reconfigure -plow unattended-upgrades
    ```


----------


    
### D - Change SSH Port from `22` to `2200` and Configure SSH access:
Reference :[Ask Ubuntu](http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server)

1. Change the SSH configuration file:

    (a) -  Access the config file using nano editor:
    
    ```
        root@ip-10-20-30-10:~# nano /etc/ssh/sshd_config
    ```
    while in the nano editor:
  
    (b) - Change `Port 22` to `Port 2200`.

    (c.) - Change `PermitRootLogin without-password` to `PermitRootLogin no`

    (d) - Change `PasswordAuthentication no` to `PasswordAuthentication yes`. This is temporal.

    (e) - At the end of the file, add `UseDNS no` and `AllowUsers grader`. This will allow grader SSH login access.

    (f) - Exit nano editor: `ctrl+x, y then enter`, and Restart SSH service for changes.
    
    ``` 
    root@ip-10-20-30-10:~# sudo service ssh restart
    ```
    Now you can connect to `grader` from local Machine using : 

    ```
    YOUR LOCAL MACHINE:~$ ssh grader@52.39.26.86 -p 2200
    ```
2. Create a SSH Key Pair:

    (a) - Switch to your local machine by using the `exit` command: 
    
    ```
    root@ip-10-20-30-10:~# exit
    ```
    
    Now enter this command to generate a SSH key pair.
   
   ```
    YOUR LOCAL MACHINE:~$ ssh-keygen
   ```
    Now you are asked to give a filename for the key pair. You are presented with something like this: `/Users/xxx/.ssh/id_rsa `. You can change the `id_rsa` to  whatever filename you want. In my case, I used the default filemname (id_rsa). But note that this is the default directory that key pairs should exist. So that filename should be kept safe. 
   
   You will also be prompted to enter a `passphrase` which will prevent unauthroized access to the files.
 
   Now you will see two files. Assuming you used the default `id_rsa`, you will see that ssh-keygen has generated `id_rsa` (private_key) and `id_rsa.pub` (public_key) file. The file `id_rsa.pub` will be placed on the server. *Note:* The key contained in the `id_rsa`file, which is your private key, must be kept safe. This is the key you should give to allow access to your files.  

    (b) - Switch to the remote server as `grader` and create a directory called `.ssh`
    
    ```
    grader@ip-10-20-30-101:~$ mkdir .ssh
    ```
   
   Create a new file within the `.ssh` directory called `authorized_keys`. A special file that will store the public keys.
   
    ```
    grader@ip-10-20-30-101:~$ touch .ssh/authorized_keys
    ```
    
    (c.) - Switch back to your Local Machine, and copy the contents of `id_rsa.pub`:
    
    ```
    YOUR LOCAL MACHINE:~$ sudo cat ~/.ssh/ida_rsa.pub
    ```
    
    (d) - Switch back to your Remote Server, edit authorized_keys file and paste the content of id_rsa.pub inside. Save file.
    
    ```
    grader@ip-10-20-30-101:~$ sudo nano .ssh/authorized_keys
    ```
    
    (e) - Set specific file permission on `SSH` and `authorized_keys` directories:
    
    ```
    grader@ip-10-20-30-101:~$ chmod 700 .ssh
    grader@ip-10-20-30-101:~$ chmod 644 .ssh/authorized_keys
    ```
    (f) - SSHD Configuration:
    
    ```
    grader@ip-10-20-30-101:~$ sudo nano /etc/ssh/sshd_config
    
    Change the `PasswordAuthentication yes` to `PasswordAuthentication no`
    ```
    
    To remove the `sudo: unable to resolve host...` warning after using sudo, follow these steps:
    
    - Open `sudo nano /etc/hostname`. 
    - You will see something like this `ip-10-20-25-101`. Copy that 
    - open `sudo nano /etc/hosts` and on the first line, append the hostname right before 127.0.0.1 localhost like so:
    
	    ```
	    52.39.26.86 ip-10-20-25-101
		127.0.0.1 localhost
	    ```
    
    You can also simplify your SSH login. Instead of having to type a long ssh login  command: `$ ssh grader@52.39.26.86 -p 2200`, you can simply type in `$ grader` to quickly connect to your Server. Here is how to do it:

    - Navigate to your `.bash_aliases` file from your local computer:
    
        ```
        YOUR LOCAL COMPUTER:~$ sudo nano ~/.bash_aliases
        ```

    - Add your SSH path like so:

        ```
        alias grader='sudo ssh grader@52.38.238.110 -p 2200'
        ```
    
    - Save the file: `ctrl+x, y then enter`, and restart the aliases file:

        ```
        YOUR LOCAL CMOPUTER:~$ source ~/.bash_aliases
        ```

    - Now you can simply do this to enter your Server:

        ```
        YOUR LOCAL COMPUTER:~$ grader
        ```        

    To handle the message: `System restart required...` after login:

    - List all the packages causing the reboot:
    
        ```    
        grader@ip-10-20-30-101:~$ cat /var/run/reboot-required.pkgs
        ```
        
    - List all high security issues:
    
        ```
        grader@ip-10-20-30-101:~$ xargs aptitude changelog < /var/run/reboot-required.pkgs | grep urgency=high
        ```
        
    - reboot your system if necessary: `sudo shutdown -r now`


----------


### E - Configure UFW to only allow incoming connections for SSH(Port:2200), HTTP(Port:80) and NTP(Port:123).

1. Check the status of UFW. Make sure it is `inactive`:
    
    ```        
    grader@ip-10-20-30-101:~$ sudo ufw status
    ```
    
2. Deny all incoming connections as default so that we can allow the ones we need.

    ```
    grader@ip-10-20-30-101:~$ sudo ufw default deny incoming
    ```
    
3. Allow incoming TCP connection on SSH(Port:2200), HTTP(Port:80), NTP(Port:123)

    ```
    grader@ip-10-20-30-101:~$ sudo ufw allow 2200/tcp
    grader@ip-10-20-30-101:~$ sudo ufw allow 80/tcp
    grader@ip-10-20-30-101:~$ sudo ufw allow 123/udp
    ```    
4. Enable the firewall: 

    ```
    grader@ip-10-20-30-101:~$ sudo ufw enable
    ```


----------


    
### F - Configure local Time Zone to UTC
Reference: [Ubuntu](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

1. Open Timezone selection dialog:

    ```
    grader@ip-10-20-30-101:~$ sudo dpkg-reconfigure tzdata
    ```

2. Choose and type `None of the above`, then choose `UTC`.

3. Setup `ntp daemon` to improve time sync:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install ntp
    ```


----------


### G - Install and Configure Apache to serve a Python mod_wsgi application.

1. Install Apache web Server:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install apache2
    ```
    
2. In your browser (Chrome preferably), type in your public ip  address: `http://52.39.26.86`, and it should return - `It works!` Ubuntu page.
        
3. Install `mod_wsgi`, and `python-setuptools` helper package. This will serve Python apps from Apache:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
    ```

	If  you are getting the `libapache2-mod error`, do this:
	
	```
	grader@ip-10-20-30-101:~$ sudo apt-get remove --purge libapache2-mod-wsgi
	```
	Now, install `libapache2-mod-wsgi` again.
    
4. Configure Apache to handle requests using the `WSGI` module

    ```
    grader@ip-10-20-30-101:~$ sudo nano cat/etc/apache2/sites-enabled/000-default.conf
    ```
    
    Add the following line: `WSGIScriptAlias / /var/www/html/myapp.wsgi` at the end of the `<VirtualHost *:80> block, right before the closing </VirtualHost>`. Now save and quit the nano editor.
        
    Restart Apache: `sudo apache2ctl restart`

5. Create the `myapp.wsgi` file that was added to the deafult-conf file: (You can skip this stpe if you want. We just want to test that apache has been rightly configuredto read python files.)

    ```
    grader@ip-10-20-30-101:~$ sudo nano /var/www/html/myapp.wsgi
    ```
    You can test the app by adding the following script in the opened nano editor to be sure that apache has been rightly configured to recognize python applications:
    
    ```python
    def application(environ, start_respose):
        status = '200 ok'
        output = 'Hello World - Its Working'
        
        response_headers=[('content-type','text/plain'),('content-length', str(len(output)))]
        start_response(status, response_headers)
        return [output]
    ```
    
    After you save the file, refresh/reload your browser and you should see `Hello World - Its working`. This same method will be used for our Catalog app configuration process.
    
6. Restart Apache server to load mod_wsgi.

    ```
    grader@ip-10-20-30-101:~$ sudo service apache2 restart
    ```
    
7. To remove the message: `Could not reliably determine the server's fully qualified domain name...`:

    (a) -  Create an Apache config file with the domain name:
        
    ```    
    grader@ip-10-20-30-101:~$ echo "ServerName 52.39.26.86" | sudo tee /etc/apache2/conf-available/fqdn.conf
    ```
    
    (b) -  Enable the file:
    
    ```
    grader@ip-10-20-30-101:~$ sudo a2enconf fqdn
    ```


----------


    
### H - Install Git and Setup Environment for delopying Flask Application. 
Reference: [Github](https://github.com/elnobun/Item-Catalog-Movie-Collection-App-/tree/master/vagrant)

1. Install Git:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install git
    ```
    
    You can set up your name and email address for the commits to your account.

    ```
    grader@ip-10-20-30-101:~$ git config --global user.name "YOUR NAME"
    grader@ip-10-20-30-101:~$ git config --global user.email "YOUR EAMIL"
    ```
    
2. Setup process for delopying Flask application:
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

    (a) - Add additional Python package to enable Apache serve Flask applications:
    
    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install libapache2-mod-wsgi python-dev
    ```
    
    (b) - Enable `mod_wsgi` if it is not enabled already:
    
    ```
    grader@ip-10-20-30-101:~$ sudo a2enmod wsgi
    ```
    
    (c.) - Navigate to the `www` directory:
    
     ```
    grader@ip-10-20-30-101:~$ cd /var/www
    ```
    
    - Setup a directory folder. You can call it `Catalog`: This will hold our app,
    
        ```
        grader@ip-10-20-30-101:/var/www$ sudo mkdir Catalog
        ```
    - cd `Catalog` and make another directory called `catalog`.
    
        ```
        grader@ip-10-20-30-101:/var/www$ cd Catalog
        grader@ip-10-20-30-101:/var/www/Catalog$ sudo mkdir catalog
        ```
    
    - cd `catalog` and make a directory called `static templates` 
    
        ```        
        grader@ip-10-20-30-101:/var/www/Catalog$ cd catalog
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo mkdir static templates
        ```
        
    - Inside `catalog` folder, create a flask applicaion logic file called `__init__.py` through the nano editor: *Note* `__init__.py` is written with double underscore like so: `_\_\init\_\_.py`.
    
        ```
        grader@ip-10-20-25-175:/var/www/Catalog/catalog$ sudo nano __init__.py
        ```
        
    - Inside the __init__.py  nano editor, paste this code:
    
        ```python
        from flask import Flask
        app = Flask(__name__)
        @app.route("/")
        def hello():
            return "Hello, Catalog app coming up soon!"
        if __name__ == "__main__":
        app.run()
        ```
        
       You can use `ls` or `ls -al` to view the content of your file path. 
       
       Next we  will test our Python Flask file:
        
3. Flask Installation and Virtual Environment configuration:
    
    (a) -  Install `pip` (good practice)
    
    ```            
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-pip
    ```
    
    (b) -  Install virtual environment (virtualenv):
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install virtualenv
    ```
    
    You can set the virtual environment name to a shorter name. e.g `venv`
    
    ```    
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo virtualenv venv
    ```
    
    If you are getting the `Error: locale.Error...` reponse, do this:

	```
	grader@ip-10-20-30-101:~$ export LC_ALL = C
	```
    
    - Enable all permissions for the new virtual environment `venv`. By doing so, `sudo` would not be used inside the environment.
    
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo chmod -R 777 venv
        ```
    - Now activate the Virtual Environment:
    
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ source venv/bin/activate
        
         You  will see this:
        (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$
        ```
    - Inside the Virtual Environment, install Flask
    
        ```        
        (venv) grader@ip-10-20-30-101:~$ /var/www/Catalog/catalog$ pip install Flask
        ```    
    - Run the __init__.py file (our python test app)
    
        ```
        (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ python __init__.py
        ```
        
    - If everything is ok, it will run alright.
    
    - Deactivate the Virtual environment:
        
        ```
        (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ deactivate
        ```
4.  Configure and Enable a New Virtual Host that will house our `.wsgi` file we are to create, just like we did while testing our `myapp.wsgi` file.

    (a) - Create vitual host config file:
    
    ```        
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf
    ```
    - In the newly created `catalog.conf` file, paste in the following lines of code.
        ```
        <VirtualHost *:80>
            ServerName 52.39.26.86
            ServerAdmin admin@52.39.26.86
            WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
            <Directory /var/www/Catalog/catalog/>
                Order allow,deny
                Allow from all
            </Directory>
            Alias /static /var/www/Catalog/catalog/static
            <Directory /var/www/Catalog/catalog/static/>
                Order allow,deny
                Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        ```
     - Enable the Virtual Host.
        
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo a2ensite catalog.wsgi         
        ```
    
    (b) - Create the `catalog.wsgi` file that was defined in the host.
    
    - Go back to the `Catalog` folder:
    
        ```
        grader@ip-10-20-30-101:/var/www/Catalog/catalog$ cd /var/www/Catalog
        ```
        
    - Create the `catalog.wsgi` file, using the nano editor:
    
        ```
        grader@ip-10-20-30-101:~/var/www/Catalog$ sudo nano catalog.wsgi
        ```
        
    - Paste the following code inside the `catalog.wsgi` file
    
        ```python
        #!/user/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/Catalog/catalog/")
        
        from catalog import app as application
        application.secret_key = 'Add your secret key'
        ```
        *Note*: The `catalog.wsgi` file, looks into the path `/var/www/Catalog/catalog` for a python file that executes your cloned project 3 file placed in the `/catalog` folder ( we will be doing this next). That file contains the *`app = Flask(__name__)`* expression. If The application that runs your python code is called `catalog.py`, and catalog.py contains that Flask expression, then using `from catalog import app...` is the correct syntax. But if your file that runs your python application is `__init__.py`, and it contains the Flask expression, then it would be proper to do this:
        
        ```python
        #!/user/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/Catalog/catalog/")
        
        from __init__ import app as application
        application.secret_key = 'Add your secret key'
        ```
        In a nutshell, `app` must be imported from whatever application that exeutes your python code.   I figured this out the hard way.
        
    - Restart Apache:
        
        ```
        grader@ip-10-20-30-101:/var/www/Catalog$ sudo service apache2 restart 
        ```
        
5. Clone Your (Project 3 - Item Catalog) respository

    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ git clone https://github.com/elnobun/Item-Catalog-Movie-Collection-App-.git
    ```

6. Move all the contents of your cloned respository directory into `/var/www/Catalog/catalog`, and delete empty directory.

    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ mv Item-Catalog-Movie-Collection-App-/* /var/Catalog/catalog
    ```
    
7. Render your respository inaccessible:

    (a) - Create a `.htaccess file`:
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog$ sudo nano .htaccess
    ```
    
    (b) - Add this to the opened nano .htaccess file : `RedirectMatch 404 /\.git`
    
8.  Install all the neeeded packages and modules for python.

    (a) - FIrst activate your virtual environment:
    
    ```
    grader@ip-10-20-30-101:/var/www/Catalog/catalog$ source venv/bin/activate
    ```
    
    (b) - Install all these packages:
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-setuptools
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install Flask
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ pip install httplib2
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ pip install requests
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install flask-seafurf
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-psycopg2
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install oauth2client
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo pip install sqlalchemy
    ```
    Restart apache: `sudo apache2ctl restart`.


----------


### I - Install and configure PostgreSQL with default settings to not allow remote Connection:
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

1. Install the PostgreSQL database:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install postgresql postgresql-contrib
    ```

2.  Ensure that no remote connections are allowed. It should be default.

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf
    ```
    
3.  Open your project 3 `database_setup.py` file:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano database_setup.py
    ```
    
4.  Effect these changes:

    (a) - Go to the line that have this syntax:
    
    ```python
    engine = create_engine('sqlite:///YOUR-DATABASE-NAME.db')
    ```
    
    (b) - Change the above syntax to a Postgresql database engine like so.
    
    ```python
    engine = create_engine('postgresql://catalog:*DB-PASSWORD*@localhost/catalog')
    ```
    you should put down a password where you have *DB-PASSWORD*. Make sure you rememebr the password because you will need it later.
    
    Also, effect the above changes in your main `app.py` file. I.e the python file you execute to run your project. In my projec 3, my python execution file  is `movie_app.py`.
    
    (c.)  - Rename your `app.py` (mine was movie_app.py), to `__init__.py`
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ mv YOUR_APP.py __init__.py
    ```
    
    *Note*:  If you make that change above, make sure your 'catalog.wsgi` file reflects this change. It should read like so:
    
    ```python
    #!/user/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/Catalog/catalog/")

    from __init__ import app as application
    application.secret_key = 'Add your secret key'
    ```
    
    If you did not make the above changes, then `catalog.wsgi` should have whatever application_name.py in your project 3 file that contains your `app = Flask(__name__)` so that it can be imported as application in the `catalog.wsgi` file.


----------


    
### J - Create a new user: `catalog`, add user to PostgreSQL databse with limited permissions to catalog application database.

1.  Create a user `catalog` for psql:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo adduser catalog
    
    choose a password for that user.
    ```
    
2.  Change to the default user `Postgres`

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo su - postgres
    postgres@ip-10-20-25-175:~$ 
    ```
    
    Connect to the postgres system:
    
    ```
    postgres@ip-10-20-30-101:~$ psql
    ```
    
    You see this:
    
    ```
    psql (9.3.12)
    Type "help" for help.

    postgres=# 
    ```
    
3.  Add the postgres user: `catalog` and setup users parameters.

    (a) - Create user `catalog` with a `login role` and `password`
    
    ```
    postgres=# CREATE USER catalog WITH PASSWORD 'DB-PASSWORD';
    ```
    *Note* : The *DB-PASSWORD*, should be the same password you used to create the postgresql engine in your database_setup.py file.
    
    (b) - Allow the user `catalog` to be able to create databse tables
    
    ```
    postgres=# ALTER USER catalog CREATEDB;
    ```
    You can list the roles available in postgres, and their attribute:
    
    ```
    postgres=# \du
    ```
    
4.  Create a new database called `catalog` for the user: `catalog`:

    ```
    postgres=# CREATE DATABASE catalog WITH OWNER catalog;
    ```
    
5.  Connect to the database:

    ```
    postgres=# \c catalog
    ```
    
6. Revoke all rights on the database schema, and grant access to catalog only.

    ```
    catalog=# REVOKE ALL ON SCHEMA public FROM public;
    catalog=# GRANT ALL ON SCHEMA public TO catalog;
    ```
    
    Exit Postgresql and postgres user:
    
    ```
    postgres=# \q
    postgres@ip-10-20-30-101~$ exit
    ```
    
7. Create Postgresql database schema:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ python database_setup.py
    ```
    
    We can check that it worked. After you run `database_setup.py`, Go back to your postgres schema, and connect to `catalog` database.
    
    ```
    postgres@ip-10-20-30-101:~$ psql
    psql (9.3.12)
    Type "help" for help.

    postgres=# \c catalog
    ```
    
    When you conect to the `catalog` database, you can view all the relations created by your `python database_setup.py` command.
    
    ```
    catalog=# \dt
    
             List of relations
     Schema |    Name    | Type  |  Owner  
    --------+------------+-------+---------
     public | collection | table | catalog
     public | movie      | table | catalog
     public | user       | table | catalog
     (3 rows)
    ```
    
    Now exit postgres, and return to Virtual environment.
    
8.  Restart Apache:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo service apache2 restart
    ```
    
    In your browser, put in your PUBLIC-IP-ADDRESS : `52.39.26.86`. If you follwed the steps accordingly, Your applciation should come up.
    
    If you are getting `Internal server error`, You can access the Apache error log file. To view the last 30 lines in the error log, 
    
    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo tail -30 /var/log/apache2/error.log
    ```


----------


    
### K - Get OAUTH-LOGINS (Google+ and Facebook) working.

1.  To fix the `google: g_client_secrets.json` error, go to the login session of your application, to these sections:

    ```python
    app_token = json.loads(
    open('g_client_secret.json', 'r').read())['web']['client_id']

	oauth_flow:flow_from_clientsecrets('g_client_secret.json', scope='')
    ```
    
    Add `/var/www/Catalog/catalog` to your code path.:
    
    ```python
    app_token = json.loads(
    open(r'/var/www/Catalog/catalog/g_client_secret.json', 'r').read())['web']['client_id']

	
	oauth_flow:flow_from_clientsecrets('/var/www/Catalog/catalog/g_client_secret.json', scope='')
    ```
    
    Do the same thing for the `fb_client_secret.json` file. This is to enable apache locate the file through the proper path. 

	If you are 
  
2.  Go to http://www.hcidata.info/host2ip.cgi to recieve the `Host Name` for your PUBLIC-IP-ADDRESS. The Host Name for mine : `52.39.26.86`, is `ec2-52-39-26.86.us-west-2.compute.amazonaws.com`

3.  Open Apache `catalog.conf` file.

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf
    ```
    
4. Paste this in the nano editor for *catalog.conf*: `ServerAlias ec2-52-39-26.86.us-west-2.compute.amazonaws.com`

    ```
    <VirtualHost *:80>
        ServerName 52.39.26.86
        ServerAdmin admin@52.39.26.86
        ServerAlias ec2-52-39-26.86.us-west-2.compute.amazonaws.com
        WSGIScriptAlias / /var/www/Catalog/catalog.wsgi
        <Directory /var/www/Catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/Catalog/catalog/static
        <Directory /var/www/Catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    
5. Enable virtual host - catalog.conf

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo a2ensite catalog
    ```

6. To get Google+ authorization working, do this:

    (a) - On the Developer Console: http://console.developers.google.com, select your Project.
    
    (b) - Navigate to `Credentials`, and edit your `OAuth 2.0 client IDs` like so:
    
    ![oauth2](https://cloud.githubusercontent.com/assets/15114201/14838612/b00cfad8-0c12-11e6-96c3-b6fbd89bb086.png)
    
7. To get Facebook authorization working, do this:

    (a) - Go to Facebood developers page: `https://developers.google.com/ and select your app.
    
    (b) - Go to settings and fill in your PUBLIC-IP-ADDRESS like so:
    
    ![facebook oauth](https://cloud.githubusercontent.com/assets/15114201/14838811/9821d2de-0c14-11e6-96a1-3e4624d688a0.png)
    
8. You can install Monitor application:

    ```
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sudo apt-get install python-pip build-essential python-dev
    (venv) grader@ip-10-20-30-101:/var/www/Catalog/catalog$ sido pip install Glancers
    ```


----------


    
### L - Install and Configured Fail2ban intrusion protection that bans suspicious IPs.
Reference: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04)
    
1.  Install Fail2ban application:

    ```
    grader@ip-10-20-30-101:~$ sudo apt-get install fail2ban
    ```
    
2.  Copy the default config file

    ```
    grader@ip-10-20-30-101:~$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```
    
3. Open `jail.local` and change the followinf default parameters:

    ```
    grader@ip-10-20-30-101:~$ sudo nano /etc/fail2ban/jail.local
    ```
    
4.  Set the following parameters:

    ```
    set bantime = 1600
    destemail = YOURNAME@DOMAIN or YOUR-EMAIL
    action = %(action_mwl)s
    under [ssh] change port = 2200
    ```

5. Stop the service:

    ```
    grader@ip-10-20-30-101:~$ sudo service fail2ban stop
    ```
    
6. Start the service again:

    ```
    grader@ip-10-20-30-101:~$ sudo service fail2ban start
    ```
    
## FINALLY:
Restart apache2 server, run your app on amazonaws. 

    

