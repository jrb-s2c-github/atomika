# Atomika
Ansible scripts amdended from https://subok-tech.com/installing-kubernetes-using-ansible-on-ubuntu-20-04/ to boot a local Kubernetes cluster (both windows and linux worker nodes) for use as a shared development environment and/or host for a Spinnaker CI/CD constellation (see https://github.com/jrb-s2c-github/spinnaker_tryout) Tested on Ubuntu 20.04.

## Another description on how to use these Ansible playbooks to boot you own out-of-cloud cluster can be read at https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster and https://dzone.com/articles/ansible-boots-kubernetes.

## Contributing
Should you wish to contribute or improve, feel free to fork and create a pull request back for me to approve. Alternatvely drop me a message on linkedin at https://www.linkedin.com/in/janrb/ 
to be added as a contributor.

## Essentials
1) Edit the inventory file to  
   1) add master node, co-master nodes for high availability and worker nodes;
   2) point to location of private key to be found in master key folder (has to be done for each node).
2) Run k8s_containerd_pkg.yml to install and prepare each of the nodes
3) Run k8s_master_init.yml to boot up the master node
4) (Optional) Run k8s_comasters.yml to join the two other master nodes to enable high availability
5) Run k8s_workers.yml to join the worker nodes

## Preparing a Linux box to take commands from Ansible
In the multipass folder is a cloud_init.yml file to prepare ubuntu nodes to receive instruction from Ansible.
The master key is in the master key folder.

**Alternatively**, it can be done manually as follows:
### New Linux box:
1. Sudo adduser vmadmin
2. Sudo usermod -aG sudo vmadmin
3. sudo update-alternatives --config editor. Select vim should you be unfamiliar with Nano.
4. sudo visudo
5. Add: 'vmadmin ALL=(ALL) NOPASSWD:ALL'
6. test that user can ‘sudo ls /root’ without having to enter password

### Copy key from server where ansible commands are ran from:
1.	Create public and private keys for user vmadmin using the ssh-keygen command on Ansible server should the keys not yet exist. This private key is the one that should be referenced in the Ansible inventory file.
2.	sudo ssh-copy-id -i ./vmadmin_key.pub vmadmin@192.168.68.109
3.	test access: sudo ssh vmadmin@192.168.68.109 -i ./vmadmin_key

## Adding a multipass VM running on Windows
1) Install multipass on windows
2) Create an external virtual switch in Hyper-V manager. The underlying network adapter can be shared with windows should there not be a secondary network adapter available. This, however, is not optimal as this sharing between two operating system slows down networking. A secondary network adapter dedicated to the multipass virtual machine is recommended. I use an external wifi adapter for this.
3) Run: 'multipass launch --cloud-init cloud_init.yml --cpus 4 --mem 4048M -n zoops1 --network “WiFi 2”'
4) Note the ip address assigned to the external switch and add to vmworkers section in the Atomika inventory file.  

## Testing Ingress and MetalLB loadbalancer ##
1) Run 'kubectl get service ingress-nginx-controller --namespace=ingress-nginx' and check that an IP address has been assigned to field "EXTERNAL-IP". This means MetalLB is listening on this IP address.
2) Run 'kubectl create deployment demo --image=httpd --port=80' to install web server
3) Run 'kubectl expose deployment demo' to expose web server as service
4) Run 'kubectl create ingress demo --class=nginx --rule www.demo.io/=demo:80' to create Ingress resource
5) Open www.demo.io inside a web browser on any node in the cluster and check that "It works!" is displayed

See https://kubernetes.github.io/ingress-nginx/deploy/#quick-start for more

## References 
Read the first two to gain understanding what the two prompts starting the master boot-up are about. 
1) https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
2) https://metallb.universe.tf/configuration/
3) https://phoenixnap.com/kb/how-to-create-sudo-user-on-ubuntu#:~:text=Most%20Linux%20systems%2C%20including%20Ubuntu%2C%20have%20a%20user,terminal%2C%20enter%20the%20command%3A%20usermod%20-aG%20sudo%20newuser
4) https://multipass.run/docs/launch-command





