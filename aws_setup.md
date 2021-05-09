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


