AWS

## SETTING UP YOUR INSTANCE

When setting up an EC2 instance you need to remember to use an UBUNTU Linux machine. If you set up an AMI with Amazon Linus you won't be able to use certain commands like apt-get. 

Select an UBUNTU AMI with the settings you will need. Almost always you use one with root device type EBS and Virtualization type HVM. Ubuntu Server XXX.version. LTS

Need will most likly need at least 8 cores, 32 G RAM, 250G space and a root with 50Gb
ALWAYS make sure you check protection for accidental protection. 

Configure a security group. SSH (port 22) and HTTP (port 80) with default options. 
Protocol is always TCP. 

Once your create a PEM file you cannot download again. 

## CONNECTING TO YOUR INSTANCE

Download your keyfile (PEM)

```
chmod 400 $$$NAME_OF_AWS_KEY.pem
#this gives permission to your PEM file. if you don't use it chmod you wont be able to access the pem file. 

ssh -i $$$NAME_OF_AWS_KEY.pem ubuntu@$$$IP_address_yourinstance
#ssh is the command to start a shell
#if your connected then your prompt will change color. REMEMBER that you can ONLY use ubuntu as the username if you install ubuntu and not amazon linus as AMI. 

exit #to disconnect but will not terminate your instance! 

```
## SETTING UP APACHE WEB ACCESS

If you want to easily retrieve or share data created on your instance, one option is to start an Apache web service on the instance so that you can browse the contents of certain directories remotely in a web browser. Note that when launching the instance our security group was configured to allow http access via port 80 so that this would work.

First you need to install Apache (in case it does not come with it)

```
sudo apt-get install apache2 -y
```

After installation completed, verify the Apache2 version.

```
sudo apache2ctl -v
```

The output looks like this:
Server version: Apache/2.4.41 (Ubuntu)
Server built:   2019-04-02T20:30:26

Step 2. Testing Apache2 Web Server
```
sudo service apache2 status
```
This will take you to a website that shows that it is working but you cannot access it. You need to change permisions in your EC2 instance as follows:

Edit config to allow files to be served from outside /usr/share and /var/www

```
sudo nano /etc/apache2/apache2.conf
#nano is the tool to edit text files in linux
#you will be accessing the aapche2.conf file
```
then add the following code to teh apache.conf file
LITERALLY COPY PAST BELOW THE AREA WHERE SIMILAR TEXT SHOWS. You can always use command+F to find FollowSymLinks or something like that or simply scroll until you find it. 

```
<Directory /home/ubuntu/>
       Options Indexes FollowSymLinks
       AllowOverride None
       Require all granted
</Directory>
```

Then Edit vhost file

```
sudo vim /etc/apache2/sites-available/000-default.conf
```

this will allow you to selecdt what directory should be accessible via entering the IP address in your webbrowser

find the section where it says DocumentRoot and change it to ```/home/ubuntu```

then restart apache

```
sudo service apache2 restart
```

you can now go to your webbrowser www.IPaddressCopiedFromAWS.com and you should be able download files. this is useful when you wanna download the files you processed if you wanna use your files in R studio in your computer instead of using R in your EC2


## Basic introduction to the EC2 instance

```
#How are storage volumes mounted?
lsblk

#How much storage space is being used for various mount points?
df -h

#Detailed description of hardware
lshw

#See active resource usage right now with "top".  Use "1" to see all CPUs individually.  Use "q" to exit
top
```

## SET UP ENVIROMENTAL VARIABLES

Environmental variables 

## TO DOWNLOAD FROM GOOGLE DRIVE

To download from Gdrive you need a special script. I used one python script that make thing easier called gdown (https://github.com/wkentaro/gdown)

to install 

```
pip install gdown
```

then download using the FILEID from the shareable link. for example in a link:
https://drive.google.com/file/d/1nucLF3mkIB7F7tmRfM3dpEp1Js__DgSD/view?usp=sharing

the FILED is between file/d/ and /view? thus it would be 1nucLF3mkIB7F7tmRfM3dpEp1Js__DgSD

then use that with the following command

```
gdown https://drive.google.com/uc?id=1nucLF3mkIB7F7tmRfM3dpEp1Js__DgSD
#were you copy and past the FILEID after id=
```







