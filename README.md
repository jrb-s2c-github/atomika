# Atomika
This projects contains Ansible playbooks that boot a local Kubernetes cluster, even providing opinionated support for high 
availability and opening it up using and Ingress. Additional playbooks allow the declaration of fast deployments of 
qualifying Spring microservices from GitHub. Such a project should use Maven JIB for integration.

# TODO It has been tested on Ubuntu server v

It is not recommended to use Atomika in production environments without further hardening. It is intended as a play or
development environment.

## Another description on how to use these Ansible playbooks to boot your own out-of-cloud cluster can be read at https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services and https://dzone.com/articles/safe-clones-with-ansible.

## Contributing
Should you wish to contribute or improve, feel free to fork and create a pull request back for me to approve. Alternatvely drop me a message on linkedin at https://www.linkedin.com/in/janrb/ 
to be added as a contributor.

## Booting an Atomika K8S cluster
The first step is to read this [Dzone.com](https://dzone.com/articles/ansible-boots-kubernetes) article very carefully to gain understanding in how to get Ansible up and running
and to build the necessary background knowledge for what is to follow.

The second step is to clone the Atomika project.

### Bootstrapping
Bootstrapping adds the user account to all nodes that the Ansible control node will use to orchestrate the target nodes 
or cluster memberss using SSH. As such it creates a sudo account for the Ansible user and set up the private\public key 
combination that will be used for authentication as the ansible user.

Do not try to boot high availability out of the box. Rather firs get the single node topology going, followed by a basic
control plane and one worker. Only then attempt to start up Atomika in high availibity mode.

Bootstrapping requires the following steps:
1) Configure the server*S* under orchestration in the inventory file under atomika/inventory/ that matches the topology of 
your choice as described lower down. Also, change the ansible_user field to that of root or a sudo account. This should
be changed back to ansible once bootstrapping has been finished.
2) Create a private/public SSH key for the orchestration user. There are many Howtos that explain how to do this, but the 
command should be somewhat as follows:
>**ssh-keygen -f ansible -t ecdsa -b 521**
3) The private and public keys for a user called ansible will be ansible and ansible.pub, respectively. Store the private 
key somewhere safe and replace the file "ansible.pub" in the root of your local Atomika repo.
4) Run the playbook to create a user called "ansible" with associated public key as discussed higher up:
>**ansible-playbook --ask-pass   bootstrap/bootstrap.yml -i atomika/inventory/*****.yml -K**

Note that this play will ask for both the account and the sudo password. Should it be a root account the "-K" switch should 
be removed so the sudo password is not prompted for.

This [Dzone.com](https://dzone.com/articles/ansible-boots-kubernetes) article also provides an explanation into bootstrapping.

### Single node topology
It is possible to boot a single node K8S cluster. It may not be best practice, but it can be useful for local testing using 
the jetpack playbooks that declares fast local deployments of Java maven projects.

The steps to configure are:
1) configure your topology in the Ansible inventory in atomika/inventory/single_node_inventory.yml, specifically by 
amending the ip address and location of the private key of the ansible user created during bootstrapping.
2) Run the boot command given lower down in its own section and enter the user that will issue kubectl commands
3) Type enter when prompted for IP ranges. The creation of a MetalLb loadbalancer & Ingress combo has to be configured as per the
example of Atomika in high availability mode.
4) Ignore the error given by "Load address pool" task that arises from the lack of MetalLb & Ingress combo.
5) Since there is only one node, the master (control-plane) will have to double up as a worker as well. Consequently,
the taint that prevents pods from being scheduled on the control-plane should be removed after booting:
>kubectl taint node --all  node-role.kubernetes.io/control-plane:NoSchedule-


### Basic topology
The basic topology consists of one control plane and as many clients/worker nodes as is required:

1) Add the correct amount of worker/client nodes to the template inventory located at atomika/inventory/basic_inventory.yml
2) Amend the IP addresses of the control-plane and all worker nodes
3) Specify the location of the private key of the ansible user
4) Run the boot command given lower down in its own section and enter the user that will issue kubectl commands
5) Type enter when prompted for IP ranges. The creation of a MetalLb loadbalancer & Ingress combo has to be configured as per the
   example of Atomika in high availability mode.
6) Ignore the error given by "Load address pool" task that arises from the lack of MetalLb & Ingress combo.

### High Availability topology
Please study the [DZone.article](https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster) explaining 
for an explanation of Atomika and high availability. Subsequently, configure your HA topology in atomika/inventory/ha_atomika_inventory.yml
by adding all the required nodes and specifying the location of the ansible user's private key for each node.

during bootstrapping leave builder out

# TODO delete main/old inventory file

### Raspberry Pi 

### Booting Atomika
With configuration of the topology ready, all that remains is to boot the Atomik cluster:
>ansible-playbook atomika/k8s_boot.yml  -i atomika/inventory/******.yml

When prompted for, enter the user that will be issuing the kubectl commands. 

## Admin commands
Under the atomika/admin folder admin playbooks are stored.

### Resetting kubeadm on all nodes 
It is possible to reset kubeadmin on each node in the cluster:
>ansible-playbook atomika/admin/kubeadm_reset.yml -i atomika/inventory/*******.yml


## Declarative deployments of Spring microservices using Maven JIB
An earlier description on how to declare Continuous Integration and Deployment (CI/CD) from a Java maven project is available
at [Dzone.com ](https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services).

The details can be viewed in jetpack/deploy.yml, but in short the steps performed on the build server are:
1) Requesting a GitHub action token - see DZone.com article referenced directly above for more
2) Creating temporary keys to clone the relevant repositories
3) Cloning the relevant repositories
4) Running a full "maven install" to cater for multi-moduled Maven projects
5) Creating a Docker image using the Maven JIB plugin
6) Bypassing a Docker repo by pushing the image directly into ContainerD
7) Running the various K8S commands using kubectl, such as creating the namespace, creating the service and running the 
pre- and post-commands specified in jetpack/vars.yaml

A sample declaration is available at jetpack/vars.yaml.

# TODO sample java ms project

# TODO test sample jetpack/vars.yaml

### Build server
A new entry in the inventory is required to designate the server that will build and deploy the container to the K8S clients:

```
builder:
  hosts:
    builder1:
      ansible_connection: ssh
      ansible_host: "192.168.68.115"
      ansible_user: ansible
      ansible_ssh_common_args: "-o ControlMaster=no -o ControlPath=none"
      ansible_ssh_private_key_file: ./bootstrap/ansible
```

The IP address and location of the ansible user's private key has to be configured as per usual. Note that only one build
server is required.

### Declaration
The various elements of a deployment declaration will be discussed next, each in its own section.

#### Namespaces

Each element of this list will be created as a K8S namespace in the Atomika cluster:

```
namespaces:
  - name: hello_ns
```

#### GitHub Repositories

Each element of this list represents a GitGub repository that will be cloned:

```
git_repos:
- name: hello1
```

#### Pre- and post-commands

Here one specifies shell commands to run before and after the integration and deployment steps of the process:

```
pre_k8s_cmds:
- kubectl create -n cc deployment hazelcast --image=hazelcast/hazelcast:latest-snapshot-jdk21 --port=5701
- kubectl expose -n cc deployment hazelcast

post_k8s_cmds:
- kubectl -n cc scale deployment hazelcast --replicas 2
```

#### Ingress declaration

An K8S Ingress can be opened to route Ingress endpoints to that of Spring Controller endpoints under orchestration 
from K8S services: 

```
ingress:
   host: www.demo.io   # Leave on www.demo.io unless you own the domain and it is not referenced in any DNS registry anywhere
   rules:
      - service: hello1                # K8S service that Ingresss should route to
        namespace: env1                # K8S namespace of K8S service
        ingress_path: /env1/hello      # Endpoint that clients will call    
        service_path: /                # Endpoint of Spring microservice to map to
      - service: hello2
        namespace: env2
        ingress_path: /env2/hello
        service_path: /
```

#### Declaring Spring Microservice deployments

```
apps:
- name: hello1                 # Name of the K8S Service
  git_repo: spinnaker_tryout   # GitHub repository to clone
  jib_dir: hello_svc           # Use "." for a single module 
                               # or name of directory containing JIB connfiguration for multi-module maven project TODO   
  image: s2c/hello_svc         # Name of container image  that JIB will create
  namespace: env1              # K8S namespace that the micro-services should be added to
  git_branch: kustomize        # Git branch to checkout
  replicas: 3                  # Amount of micro-servicesinstances to start
  application_properties:      # The application.properties of te Spring micro-service
  application.properties: |
  my_name: LocalKubeletEnv1
- name: hello2                 # Declaration of a second microservice to deploy
  git_repo: spinnaker_tryout
  jib_dir: hello_svc
  image: s2c/hello_svc
  namespace: env2
  config_map_path:
  git_branch: kustomize
  application_properties:
  application.properties: |
  my_name: LocalKubeletEnv2
```

The pom.xml of a micro-service should have a JIB build plugin configured. The "to" tag is not important, since Atomika 
pushes the image directly into the ContainerD daemon of the build server. However, the "from" tag should be populated 
with your choses base container image.

More on the maven JIB plugin can be read here: https://github.com/GoogleContainerTools/jib

```
    <build>
        <plugins>        
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>3.1.4</version>
                <configuration>
                    <from>
                        <image>openjdk:17-jdk-slim</image>
                    </from>
                    <to>
                        <image>docker.io/rb/cc-hellor:31</image>
                    </to>
                </configuration>
            </plugin>
```

## Guide for the impatient
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

TODO replaced with description of bootstrapping

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

## Release Notes

### V1
First iteration of Atomika that:
* allowed for local bootup of bare-metal/self-hosted K8S cluster that was used to host [Spinnaker](https://spinnaker.io/) CI/CD 
constellation
* was tested with Ubuntu multipass to allow nodes hosted on Windows machines
* provided support for [MetalLB](https://metallb.io/) load balancer
* can be opened up using a K8S Ingress

See the [README.md](https://github.com/jrb-s2c-github/atomika/tree/V1) at the time for more.

### V2
Version two sees:
1) refactored code.
2) easier preparation of target machines to accept SSH connections from the Ansible controller. A new Ansible 
playbook that bootstraps the SSH account to the same public key on all nodes was namely added.  

More detail on V2:
1) V2 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V2/README.md)
2) [DZone.com article](https://dzone.com/articles/ansible-boots-kubernetes) that explains how to use Ansible to boot a 
Kubernetes cluster 

### V3
Version three provided support for high availability topologies.

It, furthermore, improved ease of use with the:
1) Addition of the k8s_boot.yml playbook that ensures all nodes are added to the cluster in the topology
as declared in the inventory, instead of having to run playbooks for each stage/type separately
2) Addition of the kubeadm_reset.yml playbook to call "kubeadm reset" on each node in the inventory topology

More detail on V3:
1) V3 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V3/README.md)
2) A [Dzone.com](https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster) article that details a way 
to establish high availability with a K8S bare-metal cluster such as Atomika.

### V4
Version 4 added playbooks that interpret YAML to clone Java projects from GitHub and steer deployment 
using Maven JIB and ContainerD running on the Atomika cluster. 

More detail on V4:
1) V3 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V3/README.md)
2) A [Dzone.com](https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services) explaining how to configure
the deployments using Maven, JIB and YAML.

### V5
1) Improved documentation in README.md
2) Upped version of K8S
3) Removed support for cloudinit, since it has been replaced by the more generic bootstrap.yml playbook
4) Moved location of inventory into its own sub-directory at atomika/inventory
5) Added sample inventories for single-node, single control-plane and high availability clusters

## Outstanding
1) Move to more recent version of Ubuntu
2) Improve flow of cluster bootup. Currently, common task are firstly done on the control planes then on the workers. It would
be better to perform all the common task simultaneously.
3) Remove cloudinit since it has been replaced by bootstrapping 
4) Tasks related to MetalLb and Ingress establishment should not be attempted when this has not been configured. Currently, 
it is attempted and give rise to errors in the boot play.
5) Documentation on how to open up the Ingress for topologies other than the high availability one. However, nothing prevents
one from attempting this yourself. Theoretically it should be possible to add an Ingress to any type of Atomika cluster.
6) Is it possible to do cluster upgrades from Ansible?
7) Graphical user interface to configure bootstrapping, Atomika topology and Jetpack CI/CD

## Publications in which Atomika features
1) As host for the Spinnaker CI/CD platform: https://github.com/jrb-s2c-github/spinnaker_tryout
2) TODO add all the others

## References 
Read the first two to gain understanding what the two prompts starting the master boot-up are about. 
1) https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
2) https://metallb.universe.tf/configuration/
3) https://subok-tech.com/installing-kubernetes-using-ansible-on-ubuntu-20-04/
3) https://phoenixnap.com/kb/how-to-create-sudo-user-on-ubuntu#:~:text=Most%20Linux%20systems%2C%20including%20Ubuntu%2C%20have%20a%20user,terminal%2C%20enter%20the%20command%3A%20usermod%20-aG%20sudo%20newuser
4) https://multipass.run/docs/launch-command

# Common problems

## Playbook refuse to start due to connection issues
Ansible uses ssh so try to connect from the Ansible controller to the target service directly using SSH, specifically:
1) before bootstrapping sign up using the user configured in the inventory to run the bootstrap play from:
>ansible_user: root
2) Check that the ansible user set in the inventory is correct
3) Make sure that you created the public/private keys for a user called ansible

## Ansible tips and tricks
1) https://zwischenzugs.com/2021/08/27/five-ansible-techniques-i-wish-id-known-earlier/
2) Use --start-at-task switch to continue from last successfull task after fixing the cause of a failed task, e.g.
>ansible-playbook atomika/k8s_boot.yml  -i atomika/inventory/single_node_inventory.yml --start-at-task="Initializing Kubernetes Cluster"
3) Add the "-vvv" switch to the ansible-playbook command for verbose feedback.





