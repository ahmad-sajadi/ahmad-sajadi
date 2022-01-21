First Step: Create a virtual machine on ESXI

Second Step: Install debian Server and configure it

Third Step: Install and Setup wireguard on a debian server

Fourth Step: Install and config Prometheus and alertmanager its exporters and grafana on docker containers

NOTE: Some of these configs are samples and we should replace them with right value

at first we need to install a debian server on a VM
you can create a VM on virtualbox or a Vmware (workstation or ESXI)
this jobs is tested in a ESXI vm.
to create a VM on a Vmware ESXI in server/create_vm/. run:
NOTE: Edit "vm.tf" in this directory and put right values that belongs to your peod-env.
- terraform init     <------------- for initialization
- terraform plan     <------------- for check our file is working correctly
- terraform apply    <------------- create vm

then you need to Download debian image and install it on your machine from your VMWare administration panel

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

clone the project and in server/ansible-playbooks/. directory run this command to install docker on remote_server:
- ansible-playbook install-docker.yml

then: 
- ansible-playbook jobs.yml

this is the agenda of jobs.yml file:
1. Connect to remote server
2. Create docker network
3. Download image and create alertmanager container
4. prometheus container
5. node_exporter container
6. grafana container
7. nginx_exporter container
8. And blackbox_exporter container

then:

10. Copy Alertmanager and Prometheus config files and rules.yml from ../prometheus-configs/ to remote_server
11. Restart prom and alrtmngr containers
12. Add wireguard apt-repo 
13. Install Wireguard
14. And copy Wireguard config files

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






