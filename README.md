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
###### vm.tf:

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
### Download debian image
then you need to Download [debian](https://cdimage.debian.org/debian-cd/11.2.0-live/amd64/iso-hybrid/) image and install it on your machine from your VMWare administration panel
### Install dependencies and prerequisites
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
By doing this you can run ansible-playbook and manage your server remotely without password.
### Installing Ansible
install ansible on your local machine using commandline:
```bash
apt-get install ansible
```
on /etc/ansible/hosts define your servers that you wanna do your tasks on
##### Installing docker on remote machine using ansible-playbook
clone the project and in server/ansible-playbooks/. directory run this command to install docker on remote_server:
```bash
ansible-playbook install-docker.yml
```
###### install-docker.yml:
```yml
- hosts: webservers
  become: yes
  gather_facts: false
  tasks:
  - name: Install docker packages
    remote_user: root
    apt:
      name: ['apt-transport-https','ca-certificates', 'curl', 'software-properties-common']
      state: present
    tags:
      - docker
  - name: Add Docker s GPG key for ubuntu from official site
    remote_user: root
    apt_key:
      url: https://download.docker.com/linux/ubuntu/gpg
      state: present
    tags:
      - docker
  - name: Verify gpg key with the fingerprint
    remote_user: root
    apt_key:
      id: 0EBFCD88
      state: present
    tags:
      - docker
  - name: Configure Docker for ubuntu stable repository
    remote_user: root
    apt_repository:
      repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
      state: present
      update_cache: yes
    tags:
      - docker
  - name: Update apt packages
    remote_user: root
    apt:
      update_cache: yes
    tags:
      - docker
  - name: Install docker-ce
    remote_user: root
    apt:
      name: docker-ce
      state: present
      update_cache: yes
    tags:
      - docker
  - name: Add "ahmad-test" user to "docker" group
    remote_user: root
    user:
      name: "ahmad-test"
      group: "docker"
      append: yes
    tags:
      - docker
  - name: Install docker-compose from official github repo
    remote_user: root
    get_url:
      url : https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Linux-x86_64
      dest: /usr/local/bin/docker-compose
      mode: 'u+x,g+x'
```
##### Installing grafana, peometheus, alertmanager, nginx_exporter, node_exporter, Blackbox_exporter on docker containers using ansible-playbook
Then: 
```bash
ansible-playbook jobs.yml
```
this is the agenda of jobs.yml file:
1. Connect to remote server
2. Create docker network
3. Download image and create alertmanager container
4. prometheus container
5. node_exporter container
6. grafana container
7. nginx_exporter container
8. And blackbox_exporter container
10. Copy Alertmanager and Prometheus config files and rules.yml from ../prometheus-configs/ to remote_server
11. Restart prom and alrtmngr containers
12. Add wireguard apt-repo 
13. Install Wireguard
14. And copy Wireguard config files
###### jobs.yml:
```yml
- hosts: webservers
  become: yes
  gather_facts: false
  tasks:
  - name: network
    docker_network:
      name: pnet1
      ipam_options:
        subnet: '178.18.0.0/16'
        gateway: 178.18.0.1

  - name: copy alertmanager.yml
    copy:
      src: ../prometheus-configs/alertmanager.yml
      dest: /etc/alertmanager/
  - name: copy rules.yml
    copy:
      src: ../prometheus-configs/rules.yml
      dest: /etc/prometheus/
  - name: copy promehteus.yml
    copy:
      src: ../prometheus-configs/prometheus.yml
      dest: /etc/prometheus/

  - name: "install alertmanager on a container"
    docker_container:
      name: alertmanager
      image: prom/alertmanager
      ports:
        - 9093:9093
      volumes:
        - /etc/alertmanager/:/etc/alertmanager/
      restart: true
      command:
        - '--config.file=/etc/alertmanager/alertmanager.yml'
        - '--storage.path=/alertmanager'
      networks: 
        - name: pnet1
          ipv4_address: 178.18.0.10
  - name: "install prometheus on a container"
    docker_container:
      name: prometheus
      image: prom/prometheus
      volumes:
        - /etc/prometheus/:/etc/prometheus/
        - prometheus_data:/prometheus
      command:
        - '--config.file=/etc/prometheus/prometheus.yml'
        - '--storage.tsdb.path=/prometheus'
        - '--web.console.libraries=/usr/share/prometheus/console_libraries'
        - '--web.console.templates=/usr/share/prometheus/consoles'
      ports:
        - 9090:9090
      links:
        - "alertmanager:alertmanager"
      restart: true
      networks: 
        - name: pnet1
          ipv4_address: 178.18.0.11

  - name: "install node_exporter  on a container"
    docker_container:
      name: node-exporter
      image: prom/node-exporter
      volumes:
        - /proc:/host/proc:ro
        - /sys:/host/sys:ro
        - /:/rootfs:ro
      command: 
        - '--path.procfs=/host/proc' 
        - '--path.sysfs=/host/sys'
        - --collector.filesystem.ignored-mount-points
        - "^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)"
      ports:
        - 9100:9100
      restart: true
      networks: 
        - name: pnet1
          ipv4_address: 178.18.0.12

  - name: "install grafna on a container"
    docker_container: 
      name: grafana
      image: grafana/grafana
      user: "472"
      ports:
        - 3000:3000
      volumes:
        - grafana_data:/var/lib/grafana
        - ./grafana/provisioning/:/etc/grafana/provisioning/
      restart: true

  - name: "install nginx_exporter on a container"
    docker_container:
      name: nginx-exporter
      image: nginx/nginx-prometheus-exporter:0.10.0
      ports:
        - 9113:9113
      restart: true
      command: 
        - '-nginx.scrape-uri=http://200.100.100.207:8089/stub_status'
      networks: 
        - name: pnet1
          ipv4_address: 178.18.0.13

  - name: "install blackbox_exporter on a container"
    docker_container:
      name: blackbox-exporter
      image: prom/blackbox-exporter
      ports:
        - "9115:9115"
      restart: true
      networks: 
        - name: pnet1
          ipv4_address: 178.18.0.14

  - name: Add wireguard apt-repo
    apt_repository:
      repo: deb http://deb.debian.org/debian buster-backports main contrib non-free
      state: present
      update_cache: yes

  - name: install wireguard
    apt:
      state: present
      name:
       - wireguard

  - name: copy wireguard config file to server
    copy:
      src: ../../wireguard/wg0-server.conf
      dest: /etc/wireguard/ 

```
## Configure wireguard
to config wireguard server you must change some of settings that wrote to wg0-server.conf file.
and for your client you must change settings in wg0-client.conf file.
to get PublicKey and PrivateKey of server and client use commands belloew:
```bash
wg genkey | tee server-private.key | wg pubkey > server-public.key
wg genkey | tee client-private.key | wg pubkey > client-public.key
```
files server-private.key , server-public.key , client-public.key and client-private.key will generate.
you can use 'cat' command to see and copy keys to your wg0.conf file:
- e.g: cat server-private.key

the paths of wireguard config files in both server and client are in /etc/wireguard/. directory.
you should config your firewall (if your using it).
in firewall we must allow our traffic comes through <Server Internet PublicIP>:<Port of our vpn> and translate it to our wireguard server.






