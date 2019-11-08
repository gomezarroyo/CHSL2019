AWS

#SETTING UP YOUR INSTANCE

Need 8 cores, 32 G RAM, 250G space and a root with 50Gb
ALWAYS make sure you check protection for accidental protection. 

Configure a security group. SSH and HTTP. Protocol TCP. 

Once your create a PEM file you cannot download again. 

#CONNECTING TO YOUR INSTANCE

Download your keyfile (PEM)

```
chmod 400 cshl_2019_student.pem
ssh -i cshl_2019_student.pem ubuntu@IPaddress_yourinstance
#if your connected then your prompt will change color. 

exit #to disconnect but will not terminate your instance! 

```
