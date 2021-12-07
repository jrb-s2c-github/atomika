# Atomika
Ansible scripts to boot a local Kubernetes cluster for use as a shared development environment and/or host for a Spinnaker CI/CD constellation. Tested on Ubuntu 20.04.

## Essentials
1) Edit the inventory file to  
   1) add master node, co-master nodes for high availability and worker nodes;
   2) point to location of private key to be found in master key folder (has to be done for each node).
2) Run k8s-containerd_pkg.yml to install and prepare each of the nodes
3) Run k8s_master_init.yml to boot up the master node
4) (Optional) Run k8s_comasters.yml to join the two other master nodes to enable high availability
5) Run k8s_workers.yml to join the worker nodes

## Preparing a Linux box to take commands from Ansible
In the multipass folder is a cloud_init.yml file to prepare ubuntu nodes to receive intsruction from Ansible.
The master key is in the master key folder.

**Alternatively**, it can be done manually as follows:
### New Linux box:
1. Sudo adduser vmadmin
2. Sudo usermod -aG sudo vmadmin
3. sudo update-alternatives --config editor  select vim 
4. sudo visudo
5. Add: 'vmadmin ALL=(ALL) NOPASSWD:ALL'
6. test that user can ‘sudo ls /root’ without having to enter password

### Copy key from server where ansible commands are ran from:
1.	sudo ssh-copy-id -i ./vmadmin_key.pub vmadmin@192.168.68.109
2.	test access: sudo ssh vmadmin@192.168.68.109 -i ./vmadmin_key

## Adding a multipass image running on Windows
1) Install multipass on windows
2) Create an external virtual switch in Hyper-V manager. The underlying network adapter can be shared with windows should there not be a secondary network adapter available. This, however, is not optimal as this sharing between two operating system slows down networking. A secondary network adapter dedicated to the multipass virtual machine is recommended. I use an external wifi adapter for this.
3) Run: 'multipass launch --cloud-init cloud_init.yml --cpus 4 --mem 4048M -n zoops1 --network “WiFi 2”'
4) Note the ip address assigned to the external switch and add to vmworkers section in the Atomika inventory file.  

## References 
Read the first two to gain understanding what the two prompts starting the master boot-up are about. 
1) https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
2) https://metallb.universe.tf/configuration/
3) https://phoenixnap.com/kb/how-to-create-sudo-user-on-ubuntu#:~:text=Most%20Linux%20systems%2C%20including%20Ubuntu%2C%20have%20a%20user,terminal%2C%20enter%20the%20command%3A%20usermod%20-aG%20sudo%20newuser
4) https://multipass.run/docs/launch-command





