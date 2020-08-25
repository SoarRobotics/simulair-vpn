# Start VPN Session
clone .ovpn file

    # git clone https://github.com/tahameg/temp.git
install openvpn3
first ensure that your apt supports the https transport 

    # apt install apt-transport-https
Install keys:

     # wget https://swupdate.openvpn.net/repos/openvpn-repo-pkg-key.pub
     # apt-key add openvpn-repo-pkg-key.pub
   
Install repos:

    # wget -O /etc/apt/sources.list.d/openvpn3.list https://swupdate.openvpn.net/community/openvpn3/repos/openvpn3-$DISTRO.list
    # apt update

Install openvpn3

    # apt install openvpn3
cd into user config

    # cd temp/files
  
import session file

    # openvpn3 config-import --config Luka.ovpn
start session

    # openvpn3 session-start --config Luka.ovpn
# SSH into the server
Install putty

    # sudo add-apt-repository universe
    # sudo apt update
    # sudo apt install putty
   
   Open the app
   

    # putty
![enter image description here](https://i0.wp.com/itsfoss.com/wp-content/uploads/2018/12/putty-interface-ubuntu.jpeg?w=800&ssl=1)
   

 - Go Connection > SSH > Auth 
 - Browse the Server.ppk file in the "temp"
   repo  
  - Go to the "session" in putty 
  - Type IP ( make sure the ssh is
   selected)
   

    ubuntu@18.157.120.51
