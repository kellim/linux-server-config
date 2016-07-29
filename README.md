# Linux Server Info
* IP Address: 52.38.197.19
* SSH Port: 2200
* URL: http://ec2-52-38-197-19.us-west-2.compute.amazonaws.com/

## Summary of Configuration Steps and Software Installed
* Launched Virtual Machine with info provided by Udacity. 
* Created a user named `grader` on the remote server and gave the user sudo privleges.
    *   To add the user named `grader`, ran the command `adduser grader` on the remote server. Gave user full name of "Udacity Grader" and left all other defaults the same.
    * Gave the `grader` permission to sudo by running the command `nano /etc/sudoers.d/grader` and then added this line to the file before saving: `grader  ALL=(ALL) ALL`. 
* Created ssh key for `grader`, copying the public key to the remote server.
    * In a terminal window on my local machine, ran the command `ssh-keygen` and saved the key as `/Users/my-user-account-name/.ssh/udacity_grader`, entering a passphrase when prompted.
    * On the remote server terminal where I was logged in as `root`, switched to the grader user by running `su grader`.
    * Made a .ssh folder in the `grader` user's directory by running the command `mkdir .ssh`
    * Created an `authorized_keys` folder in the `.ssh` folder that was just created with the command `touch .ssh/authorized_keys`
    * Copied the public key that was generated on my local machine by running `cat .ssh/udacity_grader.pub` and then copying the results.
    * On the remote server terminal, ran `cd ~` to go to the home directory for user `grader`.
    * Edited the `.ssh/authorized_keys` file by running the command `sudo nano .ssh/authorized_keys` and then pasted the copied public key in the file, then saved it.
    * Updated permissions on the `.ssh` directory with the command `chmod 700 .ssh`
    * Tried to update permissions on `.ssh/authorized_keys` with the command `chmod 644 .ssh/authorized_keys` but got the error `chmod: changing permissions of ‘.ssh/authorized_keys’: Operation not permitted`
    * Pressed `ctrl-d` to switch back to the `root` user and then ran the command `chmod 644 /home/grader/.ssh/authorized_keys`.
    * Logged out of remote server by pressing `ctrl-d`.
    * Logged back into remote server as `grader` with the command `ssh -i ~/.ssh/udacity_grader grader@52.38.197.19` and entered my passphrase when prompted with the message `Enter your password for the SSH key "udacity_grader"`. It responded with an `Identity Added` message and also said `connection closed`.
    * I ran the command `ssh -i ~/.ssh/udacity_grader grader@52.38.197.19` a second time and successfully logged in as `grader`.
* Updated all currently installed packages by running these commands: 
    * `sudo apt-get update`
    * `sudo apt-get upgrade`
* To make the server more secure, edited `sshd_config` with a text editor using the command `sudo nano /etc/ssh/sshd_config` and made the following changes before saving the file: 
    * Changed the SSH port from 22 to 2200 by updating the line `Port 22` to be `Port 2200`.
    * Updated the line `PermitRootLogin without-password` to `PermitRootLogin no` so that the `root` user cannot login.
    * Added the following 2 lines to the bottom of the file:
        * `AllowUsers grader` - only allow the user `grader` to login.
        * `UseDNS no`
    * Double checked that the file had the line `PasswordAuthentication no` which means users can only login with ssh and not just a password.
* Ran `sudo ssh reload` which gave an error `sudo: unable to resolve host ip-10-20-XX-XXX`. I updated the file `/etc/hosts` using the command `sudoedit /etc/hosts` and added the line `10.20.XX.XXX ip-10-20-XX-XXX`.   Ran `sudo reload ssh` again and it did not give an error. 
<br>Note: Some of the numbers in the IP addresses above have been replaced with 'X' for security purposes.
* In a new terminal window, logged into the remote server as `grader` again with the command `ssh -i ~/.ssh/udacity_grader grader@52.38.197.19 -p 2200` and entered passphrase when prompted. 
* Logged out of the previous connection to the remote server I was using by pressing `ctrl-d` with its terminal window open, then closed the window.
* Setup the ufw firewall on the remote server logged in as `grader`.
    * Ran the command `sudo ufw default deny incoming` to block all incoming requests.
    * Ran the command `sudo ufw default allow outgoing` as a default rule for outgoing connections.
    * Ran the following commands to open the needed ports only:
        * `sudo ufw allow 2200/tcp`
        * `sudo ufw allow www`
        * `sudo ufw allow ntp`
    * Enabled the firewall with the command `sudo ufw enable`.
    * Verified the firewall is active and checked the allowed ports and other settings with the command `sudo ufw status verbose`.
    * Pressed `ctrl-d` to logout of the remote server and made sure I can log back in after enabling firewall using the command `ssh -i ~/.ssh/udacity_grader grader@52.38.197.19 -p 2200`.
* Configured the local timezone on the remote server to UTC by running the command `dpkg-reconfigure tzdata`. Selected these options when prompted:
    * Geographic Area: None of the above
    * Time zone: UTC
* Installed and configured Apache on the remote server to serve a Python mod_wsgi application.
    * Installed Apache with the command `sudo apt-get install apache2`.
    * Verified installation by going to `http://52.38.197.19` in a web browser and the "Apache2 Ubuntu Default Page" appeared.
    * Installed mod_wsgi with the command `sudo apt-get install libapache2-mod-wsgi`
    * Edited the file `/etc/apache2/sites-enabled/000-default.conf` with the command `sudo nano /etc/apache2/sites-enabled/000-default.conf` and added the line `WSGIScriptAlias / /var/www/html/myapp.wsgi` above the line that says `</VirtualHost>`.
    * Restarted Apache with the command `sudo apache2ctl restart`.
* Installed and configured PostgreSQL on the remote server using the following commands:
    * `sudo apt-get install postgresql postgresql-contrib` - install PostgreSQL
    * `sudo su postgres` - switch to postgres user.
    * `psql -d postgres -U postgres` - Login to database as postgres user.
    * `CREATE USER catalog WITH PASSWORD 'PasswordGoesHere'` - create catalog user. Replaced actual password above with <em>PasswordGoesHere</em>. Permissions to be updated in a later step.
    * Note that the assignment mentions to "Do not allow remote connections" and according to [documentation](https://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html), the default behavior does not allow remote TCP/IP connections.
    * Pressed `ctrl-d` to leave the postgres prompt and then ran `su grader` to log back in as user `grader`. 
* Installed git, cloned and setup my Catalog App project so that it functions correctly when visiting my server's IP address in a browser.
    * Installed and configured git using these commands:
        * `sudo apt-get install git`
        * `git config --global user.name "My Name"`
        * `git config --global user.email "myemail@domain.com"`
    * Setup app directory on remote server and moved to that directory with the following commands:
        * `cd /var/www`
        * `sudo mkdir linkcollector`
        * `cd linkcollector`
    * Cloned the GitHub repo for the app with the following command:
        * `sudo git clone https://github.com/kellim/link-collector.git linkcollector`
    * Rename app.py to __init__.py with the following commands:
        * `cd linkcollector` - first go to the new directory created by cloning the repo.
        * `sudo mv app.py __init__.py`
    * In my Google Developer console for this app, made the following updates to use OAuth on the remote server.:
        * Added `http://ec2-52-38-197-19.us-west-2.compute.amazonaws.com` and `http://52.38.197.19` to Authorized JavaScript Origins.
        * Added `http://ec2-52-38-197-19.us-west-2.compute.amazonaws.com/oauth2callback` to Authorized Redirect URIs.
        * Removed all localhost entries from Authorized JavaScript Origins and Authorized Redirect URIs.
        * Downloaded a new `client_secrets.json` file from Google and saved it to my local machine.
    * On the remote server, ran the command `sudo nano client_secrets.json` and copied contents of the client_secrets.json file from my local PC and pasted into this file, then saved.
    * On the remote server, ran the command `sudo nano fb_client_secrets.json` and copied contents from the file of the same name on my local PC, pasted into this file, and saved.
    * On [developers.facebok.com](http://developers.facebook.com), changed the Valid OAuth redirect URIs for the app to be `http://ec2-52-38-197-19.us-west-2.compute.amazonaws.com`.
    * Install Virtual Environment, Flask, and Flask extensions with the following commands:
        * `sudo apt-get install python-pip` - install pip first as it is used to install everything else. 
        * `sudo pip install virtualenv` - install virtual environment
        * `sudo virtualenv vlc` - created virtual environment called vlc
        * `source vlc/bin/activate` - activate the virtual environment
        * `sudo pip install Flask` - install Flask
        * `sudo pip install Flask-WTF` - install Flask-WTF
        * `sudo pip install Flask-Bootstrap` - install Flask Bootstrap
        * `sudo pip install sqlalchemy-utils` - install SQLAlchemy
        * `sudo apt-get install python-psycopg2` - install psycopg2
    * Enabled and configured my virtual host with these comannds:
        * `sudo nano /etc/apache2/sites-available/linkcollector.conf`
        * Added the following code to the file before saving (code is adapted from [How to deploy a Flask application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps))
            ```
            <VirtualHost *:80>
            		ServerName 52.38.197.19
		            ServerAlias ec2-52-38-197-19.us-west-2.compute.amazonaws.com
            		ServerAdmin webmaster@localhost
            		WSGIScriptAlias / /var/www/linkcollector/linkcollector.wsgi
            		<Directory /var/www/linkcollector/linkcollector/>
            			Order allow,deny
            			Allow from all
            		</Directory>
            		Alias /static  /var/www/linkcollector/linkcollector/static
            		<Directory /var/www/linkcollector/linkcollector/static/>
            			Order allow,deny
            			Allow from all
            		</Directory>
            		ErrorLog ${APACHE_LOG_DIR}/error.log
            		LogLevel warn
            		CustomLog ${APACHE_LOG_DIR}/access.log combined
            </VirtualHost>
            ```
        * `sudo a2ensite linkcollector` - enable virtual host.
    * Create the .wsgi file in `cd /var/www/linkcollector` with the following commands:
        * `cd /var/www/linkcollector`
        * `sudo nano linkcollector.wsgi`
        * Added the following code to the file before saving (code is adapted from [How to deploy a Flask application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)) and the secret key has been changed below for security purposes.
            ```
            #!/usr/bin/python
            import sys
            import logging
            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0,"/var/www/linkcollector/")
            
            from linkcollector import app as application
            application.secret_key = 'SecretKeyGoesHere'
            ```
    * Restart Apache with the command `sudo service apache2 restart`
    * Finished PostgreSQL setup:
        * Ran the command `sudo nano models.py` and edited line that said `engine = create_engine('postgresql:///links')` to be `engine = create_engine('postgresql://catalog:PASSWORD_GOES_HERE@localhost/links')`. Note that the real password isn't shown here.
        * Ran the command `sudo nano add_test_data.py` and made the same changes to the file as I did to `models.py` above.
        * Then did the same for `__init__.py`, running `sudo nano __init__.py`.
        * Ran these commands to create a superuser account in PostgreSQL for `grader` user I'm logged in as:
            * `sudo -u postgres createuser --superuser $USER`
            * `sudo -u postgres createdb $USER`
        * Ran `psql` to go to psql prompt.
        * Ran `create database links;` to create the database.
        * Press `ctrl-d` to exit psql command prompt.
        * Ran `python models.py` to add tables to database.
        * Ran `psql` to go to psql prompt again
        * Ran the following commands to grant access to tables in the links database to the catalog user:
            * `GRANT SELECT, UPDATE, INSERT, DELETE ON users TO catalog;`
            * `GRANT SELECT, UPDATE, INSERT, DELETE ON collection TO catalog;`
            * `GRANT SELECT, UPDATE, INSERT, DELETE ON category TO catalog;`
            * `GRANT SELECT, UPDATE, INSERT, DELETE ON link TO catalog;`
        * Press `ctrl-d` to exit psql prompt.
        * Ran `python add_test_data.py` to add test data to database.
      * Tested app in browser, and getting error `No module named oauth2client.client` so ran command `sudo pip install --upgrade google-api-python-client --ignore-installed six` to resolve it.
      * Ran `sudo nano __init__.py` and made the following updates:
          * Deleted this code at the bottom of the file:
              ```
              if __name__ == '__main__':
                  app.secret_key = secret.SECRET_KEY
                  app.debug = True
                  app.run(host='0.0.0.0', port=5000)
              ```
      * Deleted the line `import secret`
      * Deleted the `secret.py.config` file with the command 'rm secret.py.config' (because the secret key is now in the wsgi file)
      * Updated the 4 lines with filenames `client_secrets.json` and `fb_client_secrets.json` to have the full path to the files: `/var/www/linkcollector/linkcollector/client_secrets.json` and `/var/www/linkcollector/linkcollector/fb_client_secrets.json`.
      * Tested app in browser and it works.

## Resources Used
* https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04
* http://www.ducea.com/2006/06/18/linux-tips-password-usage-in-sudo-passwd-nopasswd/
* http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none
* http://stackoverflow.com/questions/33441873/aws-error-sudo-unable-to-resolve-host-ip-10-0-xx-xx
* https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04
* http://askubuntu.com/questions/709843/how-to-configure-ufw-to-allow-ntp-to-work
* https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
* http://killtheyak.com/use-postgresql-with-django-flask/
* https://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04
* http://initd.org/psycopg/docs/install.html
* https://launchschool.com/blog/how-to-install-postgres-for-linux
* http://www.tutorialspoint.com/postgresql/postgresql_create_database.htm
* https://www.postgresql.org/docs/current/static/ddl-priv.html
* https://www.postgresql.org/docs/9.1/static/sql-createdatabase.html
* https://www.postgresql.org/docs/current/static/sql-grant.html
* https://discussions.udacity.com/t/no-module-named-oauth2client-client/168463
* https://discussions.udacity.com/t/client-secret-json-not-found-error/34070
