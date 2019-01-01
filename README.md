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
and then I checked the grader user by running `finger grader`.
