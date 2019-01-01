# aws-lightsail-linux-server-configuration

Project 6 of the Udacity Full Stack Developer Nanodegree

_hostname_:   ec2-18-191-144-56.us-east-2.compute.amazonaws.com  
_ip address_: 18.191.144.56

## Step 1: AWS Lightsail Registration
Follow the default setup for a Ubuntu instance on AWS at https://lightsail.aws.amazon.com/  
Upon completion, download the default .pem private key provided by Amazon on your Account Page, and use this to SSH login as ubuntu user.  

## Step 2: Update Packages
Because we have just initialized a new machine in the cloud, it is safe to immediately update all packages on the OS. Do this by running the following:
```
sudo apt update
sudo apt upgrade
```
At this point, if the Ubuntu prompt states that a restart is required, run
```
sudo reboot
```
and then log back in from your host machine via ssh (remember to use the -i parameter with your .pem key)

## Step 3:  Setup grader account and keys for grader
Becase we are logged in as the Ubuntu user currently, we have access to `sudo` and should use it at this point to create the `grader` account. Run the following to create the `grader` account as well as grant it `sudo` access:
```
sudo adduser grader
```
I then verified this user creation by using `finger`. First, I installed `finger` by running
```
sudo apt install finger
```
and then I checked the grader user by running `finger grader` and can confirm that a user account and a `/home/grader` dir were created for the `grader` user.  
However, at this point, the `grader` user does not sudo access yet. Run the following in the remote terminal:
```
sudo vim /etc/sudoers.d/grader
```
and press `i` to enter `insert` mode and type the following text:
```
grader ALL=(ALL) ALL
```

## Step 4: Create SSH keys for `grader`
IMPORTANT: at this stage, make sure to exit the remote Ubuntu instance, and in your local host machine, run
```
ssh-keygen
```
to generate the public-private key pair for `grader`.  
Now that we have the key pairs, log back into the remote instance as `ubuntu` user and as `sudo`, run the following in the `/home` directory:
```
sudo mkdir grader/.ssh
sudo touch grader/.ssh/authorized_keys
```
Now back on your local host machine, `cat` the contents of the `.pub` key that was created in the previous step and copy it to your clipboard. And over on your remote machine, using either `nano` or `vim`, `sudo` edit the contents of `authorized_keys` and paste the contents of the `.pub` key from the local host machine. Save and close the file.   
At this point, confirm that you can now login to the remote instance as `grader` using the `ssh -i` command and passing with it the file of the private key that was generated in the previous step.  

## Step 5: Configure SSH Port 2200 and Enforce Authentication and Disable Root
At this stage, using the `ubuntu` user again, log back into the remote instance if not already logged in, and run `sudo vim /etc/ssh/sshd_config`.  
Uncomment the "Port" line and change 22 to 2200. Uncomment the "PermitRootLogin" line and modify its parameter to "no." Lastly, ensure that the "PasswordAuthentication" line states "no" and if it does not, please change it to "no."  

Save the file and run `sudo service ssh restart` to ensure the changes are made effective.

ONE IMPORTANT STEP TO NOTE HERE: With Amazon Lightsail, we will also need to go to the "Networking" tab for our instance, and ensure that we create an account Firewall rule. Clikc on "Add another" and select "Custom" for Application, "TCP" for Protocol, and use "2200" as the Port range. Save the rule.  

Now, we can exit the current SSH session and log back in but this time with the added parameter of `-p 2200` and if everything was successful, we will be able to log back in as both `ubuntu` and `grader` users.  


## Step 6: Configuring Firewall (UFW)
Log back in to the remote AWS instance and run the following:
```
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
```
After verifying that these are all entered correctly, only then proceed to run the next line:
```
sudo ufw enable
```
Press `y` and `Enter` when the confirmation dialog is presented. 
Exit the SSH session and log back in to confirm that the rules are working and that UFW status is active. 

## Step 6: Web Server and Dependency Setup
Back in the remote instance, run:
```
sudo apt install apache2
sudo apt install git (if necessary)
sudo apt install libapache2-mod-wsgi
sudo apt install postgresql
```

## Step 7: Configure Pip, Virtualenv, Flask, and Dependencies
Back in the remote instance terminal, run the following:
```
sudo apt install python-pip
sudo pip install virtualenv
```  
Now navigate to `/var/www` and `sudo mkdir catalog`.  Then run the following:
```
sudo chown grader:grader catalog
cd catalog
touch catalog.wsgi
```
Edit the `catalog.wsgi` file using either `nano` or `vim` and add the following:  
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
Then save and exit the file.   
While still inside the `catalog` directory, make sure to
TODO
TODO
TODO
change the primary Flask file `app.py` to `__init__.py` by running `mv app.py __init__.py`  
Then run the following to create the right Python environment along with all the dependencies needed to run our Flask app:
```
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
ls -al (to confirm venv and its all file permissions)
```
Now we can install flask and all the other dependencies needed to run our webapp:
```
pip install Flask httplib2 oauth2client sqlalchemy psycopg2
```

Now, modify the create and modify our configuration file by running
`sudo vim /etc/apache2/sites-available/catalog.conf`
and pasting in the following contentns:
```
<VirtualHost *:80>
    ServerName 35.167.27.204
    ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
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
Save and exit the file. 

Now, enable the virtual host by running `sudo a2ensite catalog` followed by `sudo service apache2 restart`.

