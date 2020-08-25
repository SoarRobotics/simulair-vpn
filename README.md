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
