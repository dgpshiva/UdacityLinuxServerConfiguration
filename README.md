# UdacityLinuxServerConfiguration

This repository is created for the Udacity Linux Server Configuration project submission. It details the steps that were followed and the final configuration of the server on which the app was deployed.

## Table of content
- [Server and Deployment details](#server-and-deployment-details)
- [Configuration change details](#configuration-change-details)
- [Changes on Google console](#changes-on-google-console)
- [References](#references)

## Server and Deployment details
- Public IP address of the remote server is 18.222.22.198
- SSH has been configured over port 2200
- Full url path to reach the application is [http://18.222.22.198.xip.io](#http://18.222.22.198.xip.io)

## Configuration change details
* Disable ```root``` login
    - The PermitRootLogin property in the ssh config file was edited using the command ```sudo nano /etc/ssh/sshd_config``` and set to ```no``` 
    
* Packages update
    - First login was made into the server using the ```ubuntu``` user
    - The sources list was then updated using the command ```sudo apt-get update```
    - All existing packages were then upgraded using the command ```sudo apt-get upgrade```
    - Packages specific to this ubuntu distribution were also upgraded using the command ```sudo sudo apt-get dist-upgrade```
    - Packages that are not being used were then removed using the command ```sudo apt-get autoremove```
    - ```finger``` package was then installed using the command ```sudo apt-get install finger```

* New User addition
    - ```grader``` user was then created using the command ```sudo adduser grader```
    - ```grader``` user was then given ```sudo``` access by creating the file ```/etc/sudoers.d/grader``` with the entry ```grader ALL=(ALL) NOPASSWD:ALL```

* Setting up key based auth
    - Public and Private keys were then generated on the local machine using the command ```ssh-keygen```
    - The Public key for ```grader``` user was placed in a file under his user folder at ```/home/grader/.ssh/authorized_keys```
    - Password login to the server was disabled by opening the file ```sudo nano /etc/ssh/sshd_config``` and editing the ```PasswordAuthentication``` property to ```no```
    - The ```ssh``` service was restarted so that this change is picked up using the command ```sudo service ssh restart```

* Firewall setup
    - Default firewall settings was first applied
        - ```sudo ufw default deny incoming```
        - ```sudo ufw default allow otugoing```
    - ```ssh``` was allowed using the command ```sudo ufw allow ssh```
    - Port for ```ssh``` was changed from default 22 to 2200 by again editing the ```sudo nano /etc/ssh/sshd_config``` file and setting the ```Port``` property to 2200
    - The ```ssh``` service was again restarted for this change to take into effect ```sudo service ssh restart```
    - The following services were then allowed
        - ```sudo ufw allow www```
        - ```sudo ufw allow ntp```
    - Check was made using ```sudo ufw status``` to make sure all firewall rules are applied as required

* Connection using new user account
    - Disconnected from the server
    - Connected back using the new ```grader``` user account using the command ```ssh grader@18.222.22.198 -p 2200 -i <path to private key generated in above step>(~/.ssh/<key file name>) ```

* Setting local time zone on the server to UTC
    - The command ```sudo dpkg-reconfigure tzdata``` was used to set the local time zone on server to UTC
    - This was helpful when looking at apache server logs while debugging

* Application specific packages install
    - The following were installed for the web app deployment
        - ```sudo apt-get install apache2```
        - ```sudo apt-get install apt-get libapache2-mod-wsgi```
        - ```sudo apt-get install postgresql```
        -  ```sudo apt-get install git```

* Web app set up
    - The folder was created ```/var/www/ItemCatalog```
    - The Item Catalog Application repo code was cloned into the folder
    - Connection strings in the app code were changed to use ```postgresql ```
    - The ```application.py``` file name was changed to ```__init.py__```
    - Python ```pip``` was installed using the command ```sudo apt-get install python-pip```
    - ```virtualenv``` was installed ```sudo pip install virtualenv```
    - New virtual environment was created and activated under ```/var/www/ItemCatalog/ItemCatalogApplication``` folder using the commands
        - ```sudo virtualenv flaskenv```
        - ```source flaskenv/bin/activate ```
    - Required python packages were installed into this environment
        - ```pip install Flask```
        - ```pip install httplib2```
        - ```pip install oauth2client```
        - ```pip install requests```
        - ```pip install sqlalchemy```
        - ```pip install flask_httpauth```
    - The virutal env was deactivated using the command ```deactivate```
    - New file named ```ItemCatalogApp.conf``` was created under _/etc/apache2/sites_available/ItemCatalogApp.conf_ with the contents
        ```
            <VirtualHost *:80>
                ServerName 18.222.22.198
                ServerAdmin admin@18.222.22.198
                WSGIScriptAlias / /var/www/ItemCatalogApp/itemcatalogapp.wsgi

                <Directory /var/www/ItemCatalogApp/ItemCatalogApplication>
                        Order allow,deny
                        Allow from all
                </Directory>

                Alias /static /var/www/ItemCatalogApp/ItemCatalogApplication/static
                <Directory /var/www/ItemCatalogApp/ItemCatalogApplication/static/>
                        Order allow,deny
                        Allow from all
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
            </VirtualHost>
        ```
    - New file named ```itemcatalogapp.wsgi``` was created under the _ItemCatalog_ folder with the contents
        ```
            #!/usr/bin/python
            import sys
            import logging

            activate_this =             '/var/www/ItemCatalogApp/ItemCatalogApplication/flaskenv/bin/activate_this.py'
            execfile(activate_this, dict(__file__=activate_this))

            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0,"/var/www/ItemCatalogApp")

            from ItemCatalogApplication import app as application
            application.secret_key = 'super_secret_key'```
        ```
    - The virtual host was enable using the command
        - ```sudo a2ensite ItemCatalogApp.conf```
    - The default host was disabled using the command
        - ```sudo a2dissite 000-default.conf```
    - The Apache web server was restarted using the command ```sudo service apache2 restart```

## Changes on Google console
- The path ```http://18.222.22.198.xip.io``` was added as Authorized Javascript origin on Google Developer Console for this app
- The path ```http://18.222.22.198.xip.io/v1/categories/``` was added as a Authorized Redirect URI

## References
- The [blog post](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) was really helpful is getting a better idea of steps to perform to deploy the app
