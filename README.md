First Step: Install and Setup wireguard on a debian server
Second Step: Install and config Prometheus and alertmanager its exporters and grafana on docker containers
NOTE: Some of these configs are samples and we should replace them with right value

at first we need to install a debian server on a VM
you can create a VM on virtualbox or a Vmware (workstation or ESXI)
this jobs is tested in a ESXI vm.
to create a VM on a Vmware ESXI login to your Vmware or VCenter administration panel:
- right click on your host
- click on "new virual machine"
- on the opend panel click on "create a new virtual machine"
- on the next tab select your datacenter and host
- then select datastore
- select your VM compatibility to a VMware version
- select linux on guest OS family drop down list and debian 11 on guest OS version list.
- customize VM's hardware 
- next and finish
now your OS is installed

so now run this commands on your newly installed guest OS:
- apt update
- apt install openssh-server python3 python3-pip build-essential libssl-dev libffi-dev python-dev vim 
- pip3 install docker-compose docker

now on your local machine run:
  ssh-keygen -t rsa
then: 
  ssh-copy-id  user@remote_server(VM)
by doing this you can run ansible-playbook and manage your server remotely without password.

install ansible on your local machine using commandline:
apt-get install ansible

on /etc/ansible/hosts define your servers that you wanna do your tasks on

clone the project and in project's root directory run this command to install docker on remote_server:
- ansible-playbook install-docker.yml

then: 
- ansible-playbook jobs.yml

jobs.yml file will install prometheus and it's exporters on docker containers and then copy config files from root directory of project and copy them
to remote server and then will install wireguard to remote server.

to config wireguard server you must change some of settings that wrote to wg0-server.conf file.
and for your client you must change settings in wg0-client.conf file.
to get PublicKey and PrivateKey of server and client use commands belloew:
- wg genkey | tee server-private.key | wg pubkey > server-public.key
- wg genkey | tee client-private.key | wg pubkey > client-public.key

files server-private.key , server-public.key , client-public.key and client-private.key will generate.
you can use 'cat' command to see and copy keys to your wg0.conf file:
- e.g: cat server-private.key

the paths of wireguard config files in both server and client are in /etc/wireguard/. directory.
you should config your firewall (if your using it).
in firewall we must allow our traffic comes through <Server Internet PublicIP>:<Port of our vpn> and translate it to our wireguard server.






