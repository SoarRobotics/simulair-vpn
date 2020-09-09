# Overview
Simulair clients can communicate with the simulation environment via Ros2 nodes and topics. We are connecting users to the server via VPN to enable this feature. We have set up openvpn server on our experimental AWS instance with UDP Multicast enabled, so connected users can make topic search on the server. In this document we have explained the necessary steps for setting up the OpenVPN Server and client. Also how to ssh into our experiment server, tweak OpenVPN Server Configurations, test UDP multicasting connectivity, connect local ros2 environment to the server and test the connectivity. Enjoy!! 
# References

While writing this instructions document, I got help from the following links.
- [\[1\]](https://hackernoon.com/using-a-vpn-server-to-connect-to-your-aws-vpc-for-just-the-cost-of-an-ec2-nano-instance-3c81269c71c2)
- [\[2\]](https://www.cyberciti.biz/faq/ubuntu-18-04-lts-set-up-openvpn-server-in-5-minutes/)

# Prerequisites
- Ubuntu 18.04 operating system.
- OpenSSH(comes built-in with ubuntu 18.04)
# 1- Create OpenVPN Server 
We are creating our VPN Server on an AWS EC2 instance with Ubuntu 18.04 ami. We will be setting up the minimal requirements for enabling the mentioned functionality above. For preventing the DNS leaks and providing a more robust connectivity, please check-out [this](https://openvpn.net/community-resources/how-to/#creating-configuration-files-for-server-and-clients) and [this](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-18-04)
## Create the AWS Instance 
Firstly, we should create a new Inbound Security Group for our server. On the dashboard, go to Services > EC2. Than, on the left panel, go to Security Groups and click go to "Create security group" 
![enter image description here](https://i.ibb.co/8xxfWkB/1.png)

After entering a name and description to the new security group. I named it as "Custom VPN Security Group". In the Inbound rules, add 4 rules for: 
| Type | Protocol | Port |
| ------ | ------ | -----|
| ssh | TCP |22|
| https| TCP | 443 |
| Custom TCP| TCP |943 |
| Custom UDP |  UDP|1194|

Node that port 1194 is UDP and the rest is TCP. It will look like this. 
![enter image description here](https://hackernoon.com/hn-images/1*IcPlpDswN8ohxky3Gp-FIg.png)

Source should be set as "anywhere". 
Go to "instances" in the left panel and click to "Create new instances". In the AMI selection panel, search for ubuntu. Select "Free tier eligible" LTS version of ubuntu 18.04.
![enter image description here](https://i.ibb.co/HTkkQ3h/2.png)

Now you shoul configure your instance according to your need. Since we are also setting up Ros2 on our server, we have set up our experimental server on **t3a.small** machine. Just select one and you are good to go. Add some storage and go to "Configure security group" menu. 
On the "**Assign a security group:**" menu, select "Select an **existing** security group" option.
Select the security group you have created and also **default** security group.

![enter image description here](https://i.ibb.co/KrJnNSt/3.png)

If everything looks cool, you can click to review and launch button for launching your instance. While creating the instance, create a new ssh key and download the "***.pem**" file. Now we will create an elastic ip for connecting to our instance.
Goto **elastic ips** page from the left menu. Click to **Allocate Elastic IP adress**. Follow the steps for creating a new elastic ip and associate it with your instance. If everything is alright, you can go to "**Instances**" page and review it by clicking on its "**Instance ID**".
You will have a Public IP which is the elastic IP that we have just associated with this server and a Private IP. Write them down, they will be needed for the server configurations.

## SSH into your instance
Open a Go to the directory where your **.pem** file rests
Type the following command for establishing ssh connection into your instance. Make sure you have changed *<PUBLIC_IP>* field with public ip of your instance and *<PEM_FILE_NAME>* with name of your pem file.

    ssh -i <PEM_FILE_NAME>.pem ubuntu@<PUBLIC_IP>
 if you get an error like
 

    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0644 for 'amazonec2.pem' are too open.
    It is recommended that your private key files are NOT accessible by others.
    This private key will be ignored.
    bad permissions: ignore key: amazonec2.pem
    Permission denied (publickey).

you can fix it with the following command and try again.

    chmod 400 <PEM_FILE_NAME>.pem
You should be successfuly established SSH connection now. 
## Install OpenVPN Server
!! Follow these steps for ssh prompt of your instance !!
#### Step 1 - Update your system
Run following commands

    $ sudo apt update  
    $ sudo apt upgrade

 #### Step 2 – Download and run openvpn-install.sh script

    $ wget https://git.io/vpn -O openvpn-install.sh
   Setup permissions using the chmod command:

    $ chmod +x openvpn-install.sh
   Now all you have to do is:
   

    $ sudo ./openvpn-install.sh
This script will ask for 6 inputs befaore it starts installing the OpenVPN Server.

![enter image description here](https://www.cyberciti.biz/media/new/faq/2018/09/Ubuntu-18.04-LTS-Setup-OpenVPN-Server-In-5-Minutes.png)

 1. The first field will be Private IP Address of your  instance. ( This may not be asked)
 2. Second is Public IP of your instance.
 3. Type **1** for selecting the UDP protocol
 4. Port must be 1194
 5. Always use 1.1.1.1 or Google DNSs for sake of speed.
 6. type a user name for the first user.

For this case, desktop.ovpn file will be created besides the " openvpn-install.sh " script. Pull this file by typing following command on your local machine's command prompt.You should run this script again for creating new users. ( All clients must log in with different ovpn files)

    sudo scp -i <directory to "Server.pem" file> ubuntu@<Public IP of instance>:~/<user name>.ovpn <where you want it to be downloaded>

  #### Step 3 - Manage Server Service
  You can use following commands for starting, stopping, restarting the OpenVPN service.

    $ sudo systemctl stop openvpn-server@server.service # <--- stop server  
    $ sudo systemctl start openvpn-server@server.service # <--- start server  
    $ sudo systemctl restart openvpn-server@server.service # <--- restart server  
    $ sudo systemctl status openvpn-server@server.service # <--- get server status

## Configure OpenVPN Server
**In your local device**, locate to root directory on where you have cloned this repo.
cd into **configuration** folder

    ~/Simulair-VPN$ cd configuration
Send server.conf file to the "home" directory of your remote instance with ssh. Type following command on your local machine prompt.

    $ sudo scp -r -P 22 -i *<path to your .pem file>* server.conf ubuntu@<Public IP of Server>:~/

Make sure you have changed the generic fields. In my case I have typed:

    $ sudo scp -r -P 22 -i ~/Downloads/temp/experiment-server/Server.pem server.conf ubuntu@<my public ip>:~/

Now go to ssh prompt of your remote instance. You will see server.conf file is loaded in ' ~/ ' or ' /home ' directory. 
Open this file for editing.

    $ sudo nano server.conf
goto end of the file where you will see the following line.
![enter image description here](https://i.ibb.co/qJWT3Rf/4.png)

The ip that comes after the route keyword defines ip space to be routed in. This must be first 2 fields of Private IP of your instance. E.g, if your private ip is 134.12.4.2, than you should change this line to 

    push "route 134.12.0.0 255.255.0.0"
Do ctrl + x and ---> press Y ---> press Enter 

Move this file to /etc/openvpn. (if exists, remove  all .conf files inside /etc/openvpn directory before this step)

    ~$ sudo move server.conf /etc/openvpn
restart the OpenVPN service

    $ sudo systemctl restart openvpn-server@server.service

## Connect to VPN Server

Go to the directory where you have pulled your ovpn file. Follow below instructions in the client machine(your local device)
first ensure that your apt supports the https transport

    $ apt install apt-transport-https

Install keys:
  
   

    $ wget https://swupdate.openvpn.net/repos/openvpn-repo-pkg-key.pub
    $ apt-key add openvpn-repo-pkg-key.pub

Install repos:

    $ wget -O /etc/apt/sources.list.d/openvpn3.list https://swupdate.openvpn.net/community/openvpn3/repos/openvpn3-bionic.list
    $ apt update

Install openvpn3

    $ apt install openvpn3

import session file

    $ openvpn3 config-import --config <your ovpn file>

start session

    $ openvpn3 session-start --config <your ovpn file>

If these runs without any errors, you should have successfully established your vpn session. Type ifconfig in your client machine for checking if you have assigned an IP.
![enter image description here](https://i.ibb.co/bbFNF1P/5.png)
This sections shows my ip in the VPN network.
## Testing the UDP Multicast Connectivity 
Ros2 uses UDP ports for making topic search inside the local network. Our VPN server must support UDP Multicasting for this kind of operation. We will use 
