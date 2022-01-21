### First Step: 
Create a virtual machine on ESXI
### Second Step: 
Install debian Server and configure it
### Third Step:
Install and Setup wireguard on a debian server
### Fourth Step:
Install and config Prometheus and alertmanager its exporters and grafana on docker containers
##
NOTE: Some of these configs are samples and we should replace them with right value

at first we need to install a debian server on a VM
you can create a VM on virtualbox or a Vmware (workstation or ESXI)
this jobs is tested in a ESXI vm.
to create a VM on a Vmware ESXI in server/create_vm/. run:

NOTE: Edit "vm.tf" in this directory and put right values that belongs to your peod-env.
```bash
terraform init 
terraform plan
terraform apply
```
## vm.tf:

```yml
terraform {
  required_providers {
    vsphere = {
      source = "hashicorp/vsphere"
      version = ">= 2.0"
    }
  }
}
provider "vsphere" {
  user           = "VM user"
  password       = "VM password"
  vsphere_server = "VM Address"
  allow_unverified_ssl = true
}


data "vsphere_datacenter" "dc" {
  name = "arvan"
}
data "vsphere_datastore" "datastore" {
  name          = "datastore1"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
data "vsphere_network" "network" {
  name          = "VM Network"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

data "vsphere_resource_pool" "pool" {
  name          = ""
  datacenter_id = data.vsphere_datacenter.dc.id
}
resource "vsphere_virtual_machine" "vm" {
  name             = "your vm name"
  resource_pool_id = "${data.vsphere_resource_pool.pool.id}"
  datastore_id     = "${data.vsphere_datastore.datastore.id}"

  num_cpus = numberOfCpus
  memory   = MemmorySize
  guest_id = "other3xLinux64Guest"

  network_interface {
    network_id = "${data.vsphere_network.network.id}"
  }

  disk {
    label = "disk0"
    size  = SizeofVMdisk 
  }
}


```
##
then you need to Download [debian](https://cdimage.debian.org/debian-cd/11.2.0-live/amd64/iso-hybrid/) image and install it on your machine from your VMWare administration panel
##
So now run this commands on your newly installed debian:
```bash
apt update
apt install openssh-server python3 python3-pip build-essential libssl-dev libffi-dev python-dev vim 
pip3 install docker-compose docker
```
now on your local machine run:
```bash
ssh-keygen -t rsa
```
then: 
```bash
ssh-copy-id  user@remote_server(VM)
```
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






