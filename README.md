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

now on your local machine run:
  ssh-keygen -t rsa
then: 
  ssh-copy-id  user@remote_server(VM)
by doing this you can run ansible-playbook and manage your server remotely without password.




