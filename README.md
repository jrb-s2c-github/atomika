>>> V5_1 has been released. Cluster formation is now much faster and easier to execute. Only five steps
are required to have it up and running. 

# Atomika
```
 ATOMIKA: Local deployments to Kubernetes (on Windows)
          
                    .   .xXXXX+.   .
               .   ..   xXXXX+.-   ..   .   
         .   ..  ... ..xXXXX+. --.. ...  ..   .
     .   ..  ... .....xXXXX+.  -.-..... ...  ..   .
   .   ..  ... ......xXXXX+.  . .--...... ...  ..   . 
  .   ..  ... ......xXXXX+.    -.- -...... ...  ..   .
 .   ..  ... ......xXXXX+.   .-+-.-.-...... ...  ..   .
 .   ..  ... .....xXXXX+. . --xx+.-.--..... ...  ..   .
.   ..  ... .....xXXXX+. - .-xxxx+- .-- .... ...  ..   .
 .   ..  ... ...xXXXX+.  -.-xxxxxx+ .---... ...  ..   .
 .   ..  ... ..xXXXX+. .---..xxxxxx+-..--.. ...  ..   .
  .   ..  ... xXXXX+. . --....xxxxxx+  -.- ...  ..   .
   .   ..  ..xXXXX+. . .-......xxxxxx+-. --..  ..   .
     .   .. xXXXXXXXXXXXXXXXXXXXxxxxxx+. .-- ..   .
         . xXXXXXXXXXXXXXXXXXXXXXxxxxxx+.  -- .
           xxxxxxxxxxxxxxxxxxxxxxxxxxxxx+.--
            xxxxxxxxxxxxxxxxxxxxxxxxxxxxx+-   Ojosh!ro
            
  MIT License, Copyright (c) 2021 S2C Consulting (PtyLtd ZA)
```

Atomika is a collection of Ansible playbooks to boot a bare-metal Kubernetes cluster. It provides support for high 
availability and opens external access using a [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/). 

A second set of playbooks reads deployment declarations, integrates container images and deploys to Atomika. Maven JIB 
is used to integrate Docker containers and kubectl used to set up the required Service and Deployment orchestration agents. 

Atomika is intended as a play or development environment. It is not recommended for production use without further hardening. 

Atomika allows for single-node, basic and high availability topologies. Guidance is given lower down how to declare each 
topology. Do not try to boot high availability out of the box. Rather, get a basic topology consisting of one control
plane and one worker going the first time around.  

Atomika has been tested on a cluster of Unbuntu 22 machines running inside Windows. However, it can run on any collection 
of physical and virtual machines.

Ansible controllers pre-loaded with the correct version of Atomika is maintained by the [jrb-s2c-github/atomika_wormhole](https://github.com/jrb-s2c-github/atomika_wormhole)
project and can be downloaded from [here](https://drive.google.com/drive/folders/1OY1rDy6MwYi0iXD159igjnJ7IOrBbJ1U).

## Dzone.com articles on Atomika
- [Running Ansible From Windows Using Virtualization](https://dzone.com/articles/running-ansible-from-windows-using-virtualization) 
- [Ansible by Example](https://dzone.com/articles/ansible-boots-kubernetes)
- [Anatomy of a High Availability Kubernetes Cluster](https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster)
- [Fast Deployments of Microservices Using Ansible and Kubernetes](https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services)
- [Safe Clones With Ansible](https://dzone.com/articles/safe-clones-with-ansible)

## Using Wormhole as the Ansible controller
Wormhole comes with both Ansible and Atomika cloned from GitHub. Atomika's 
root ($atomika-home) is located at /home/ansible/atomika/ meaning:
* its playbooks is at /home/ansible/atomika/atomika;
* those of jetpack at /home/ansible/atomika/jetpack and
* the inventory templates at /home/ansible/atomika/inventory.

The ansible user should be accessed using the ansible/ansible.pub keypair located at the
root of atomika_wormhole with sudo password of 'atmin'.

## Guide for the impatient 

> It only takes five steps to start the Atomika Kubernetes cluster!

#### Step 1
*Download* the matching version of an Atomika Wormhole image from [here](https://drive.google.com/drive/folders/1OY1rDy6MwYi0iXD159igjnJ7IOrBbJ1U).

#### Step 2
*Create two nodes*. On Windows this can be done by running the liftoff Powershell script from the [jrb-s2c-github/atomika_wormhole](https://github.com/jrb-s2c-github/atomika_wormhole) 
project in a PowerShell admin console. This script can be found at startup_scripts/liftoff.ps1. 

This will require 8GB of ram. Should ram be limited boot the control-plane/master with 2GB
(change -MemoryStartupByte to 2GB in liftoff.ps1) or opt to use the single node inventory 
in the inventory folder.

Pick one of the two machines and use it as the Ansible controller. 

#### Step 3
*Register the nodes's private key* with your ssh-agent. The ansible/ansible.pub keypair can be found in the root of 
atomika-wormhole.

#### Step 4
*Configure a co-plane/worker topology* in $atomika-home/atomika/inventory/basic_inventory.yml. Change
the ip addresses to that of the two nodes started in step 2. Ignore the builder group 
until your cluster formed, and you are ready for fast deployments using jetpack.

#### Step 5
*Boot Atomika from /home/ansible/atomika/* from an Ansible controller (anyone of the two machines):
> ansible@wormhole:/home/ansible/atomika/$ ansible-playbook -i atomika/inventory/basic_inventory.yml atomika/k8s_boot.yml -K -e metal_lb_range=172.26.64.3-172.26.64.200
                                                                
The sudo password is 'atmin' and ip range should be on the gateway's subnet. It will be used to select an ip address for 
the Ingress from. Run 'ip route' to find the gateway's ip address. 

The first time things can time out. Be patient, running '*kubectl get nodes*' as the ansible user from time to time. You 
can also rerun k8s_boot.yml to destroy the cluster and try everything again.

#### Test for yourself
This test is also automatically performed at the very end of the boot to test readiness .

*Test the Kubernetes Ingress:*
> ansible@wormhole:/home/ansible/atomika/$ kubectl -n ingress-nginx get svc ingress-nginx-controller

Note the external IP and map it to www.demo.io in the hosts file (/etc/hosts or 
C:\Windows\System32\drivers\etc\hosts). Open www.demo.io in a browser and see Atomika 
in action.

## Test Jetpack and Ingress routing

Get Atomika up and running as per the "guide for the impatient" above.

Add ip address of the worker node to the builder at the very bottom of the basic inventory. 
In case of only running a single node, the ip address should be that of the master

Run jetpack to checkout, compile, integrate and deploy the sample deployment 
declarations from jetpack/vars.yml:
>ansible-playbook jetpack/deploy.yml -i atomika/inventory/basic_inventory.yml -K

Enter 'atmin' as sudo password and hit enter to clone without passing a security credential.

Open http://www.demo.io/env1/hello and http://www.demo.io/env2/hello from a browser (or 
curl) to see the [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) 
routing to two different internal Kubernetes services.

Here is how this routing is configured:
```
ingress:
  host: www.demo.io
  rules:
    - service: hello1
      namespace: env1
      ingress_path: /env1/hello
      service_path: /
    - service: hello2
      namespace: env2
      ingress_path: /env2/hello
      service_path: /
```

## Contributing
Should you wish to contribute or improve code or documentation, feel free to fork and 
create a pull request back for me to approve. Alternatively, drop me a message on 
LinkedIn at https://www.linkedin.com/in/janrb/ to be added as a contributor.

## Booting an Atomika K8S cluster the long way
The first step is to read this [Dzone.com](https://dzone.com/articles/ansible-boots-kubernetes) article very carefully 
to gain background knowledge and to get Ansible up and running.

The next step is to clone the project, followed by amending template files and feeding them to the relevant Ansible plays 
as outlined lower down.

### Bootstrapping
Bootstrapping adds the same user account to all nodes. The Ansible control node uses this account to connect to the 
target nodes over SSH. It assigns sudo rights and associates a private\public key pair for authentication.

Bootstrapping can be skipped by downloading an image prepared by the [jrb-s2c-image/atomika-wormhole](https://github.com/jrb-s2c-github/atomika_wormhole)
project. The images are available [here](https://drive.google.com/drive/folders/1OY1rDy6MwYi0iXD159igjnJ7IOrBbJ1U). Download 
one with 'atomika' (and not 'bare') in the name and with a version that corresponds to the version of Atomika being used. *User
access is via the ansible/ansible.pub keypair in the root of atomika-wormhole. These keys 
authenticates a sudo user with the name of ansible.*

However, one can also bootstrap from your own Linux image as follows:
1) Configure the node(*s*) in the relevant inventory file under the atomika/inventory/ folder that matches the topology 
of your choice. Also, change the ansible_user field to that of root or a sudo account on the target machine. *This should 
be changed back to ansible once bootstrapping has been concluded.*
2) Create a private/public SSH key for the orchestration user. There are many guides that explain how to 
create the keypair. The command should be somewhat as follows:
>**ssh-keygen -f ansible -t ecdsa -b 521**
3) The private and public keys for the ansible user account will be ansible and ansible.pub, respectively. Store the 
private key safely and replace the file "ansible.pub" in the bootstrap folder of your local Atomika repo.
4) Run the playbook:
>**ansible-playbook --ask-pass   bootstrap/bootstrap.yml -i atomika/inventory/*****.yml -K**

Should each machine be accessible by different accounts, you should bootstrap machine-by-machine using the -l switch: 
>**ansible-playbook --ask-pass   bootstrap/bootstrap.yml -i atomika/inventory/basic_inventory.yml -K -l machineX**

5) Install the nuts and bolts required by Kubernetes:
>**ansible-playbook -i atomika/k8s_init.yml atomika/inventory/*****.yml -K**

Note that above plays will ask for both the account and the sudo password. Should it 
be a root account the "-K" switch can be removed and the sudo password will not be 
prompted for.

This [Dzone.com](https://dzone.com/articles/ansible-boots-kubernetes) article also provides a more detailed explanation 
on bootstrapping.

### Configure single node topology
It is possible to boot a single node K8S cluster. It may not be best practice, but it can be useful for local testing using 
the Jetpack playbooks for fast local deployments. In my experience the first cluster 
formation takes a bit longer though.

The details of the node should be configured in the Ansible repository located at atomika/inventory/single_node_inventory.yml. 

Since there is only one node, the master (control-plane) will have to double up as a worker as well. Consequently,
the taint that prevents pods from being scheduled on control planes are removed during boot up. Similarly, a label is
removed to allow MetalLb to attract the traffic destined for the Ingress towards its sole speaker pod on the master node.

### Configure basic topology
The basic topology consists of one control plane and as many clients/worker nodes as required. Configure your topology in 
the Ansible inventory located at atomika/inventory/basic_inventory.yml:
* Configure the master 
* Add the same amount of client sections as there are to be workers in the cluster 
* Configure the details of each worker

### Configure high Availability topology
Please study the [DZone.article](https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster) 
for an explanation of Atomika and high availability. Subsequently, configure your HA topology in atomika/inventory/ha_atomika_inventory.yml
by adding all the required nodes and specifying the location of the ansible user's private key for each node.

In short the steps are:
* Configure your topology in the relevant Ansible inventory located at atomika/inventory/ha_atomika_inventory.yml
* Configure the detail of the first control plane
* Add the correct amount of worker/client nodes to the inventory
* Add the correct amount of co-master control-planes to the inventory
* Configure the workers
* Configure the co-masters 
* Configure the machine that will host HAProxy

### TODO Raspberry Pi and Arch Linux

### Booting Atomika
1) Configure the correct topology under atomika/inventory as explained higher up. Ensure that the ip address and location of 
the private key of the ansible user created during bootstrapping are correct for every target node referenced in the inventory. 
2) Boot the Atomika cluster:
>ansible-playbook -i atomika/inventory/ha_atomika_inventory.yml atomika/k8s_boot.yml -K -e metal_lb_range=172.26.64.3-172.26.64.200

When prompted, provide a range of IP addresses that are available to be assigned to the Ingress. Press enter to accept 
the default range of 192.168.68.200-192.168.68.210. Other formats that are acceptable are given on the [MetalLB](https://metallb.io/) website.
The IP range should be on the same subnet as your gateway. Should you be running virtual machines on Windows using Hyper-V's 
default switch, this gateway is located on the subnet with the lowest numeric value in 
use by the vm's. More can be read [here](https://learn.microsoft.com/en-us/windows-server/networking/sdn/technologies/hyper-v-network-virtualization/hyperv-network-virtualization-technical-details-windows-server),
but Windows routes traffic between its vm's internally using a star network. The 
gateway can be determined by running and seeing that the gateway is 172.26.64.1:
```aidl
janrb@dquick:~/atomika$ ip route
default via 172.26.64.1 dev eth0
172.18.0.0/16 dev docker0 proto kernel scope link src 172.18.0.1 linkdown
172.19.0.0/16 dev br-627ac2a318b9 proto kernel scope link src 172.19.0.1 linkdown
172.26.64.0/20 dev eth0 proto kernel scope link src 172.26.68.178
```
In this case the MetalLb can be instructed to use an ip address between 172.26.64.2-172.26.64.200

Since images are pulled from container registries, the first boot might fail. Keep on waiting and/or running the boot
command. Once a cluster formed once, subsequent boots will be much faster. 

#### By default, the user named 'ansible' will be given rights to run kubectl commands during the booting process. 
In the tips and tricks section lower down, a way is given to extend this rights to a 
second user. However, nothing prevents you to copy /etc/kubernetes/admin.conf to any 
user's home directory that has to control the kube.

## Admin commands
Administrative playbooks can be found in the atomika/admin folder.

### Resetting kubeadm on all nodes 
It is possible to reset each node in the cluster with a single command:
>ansible-playbook atomika/admin/kubeadm_reset.yml -i atomika/inventory/*******.yml

However, this reset is enforced every time the k8s_boot.yml runs.

## Jetpack: Declarative deployments of Spring microservices using Maven JIB
A fuller description on how to declare Continuous Integration and Deployment (CI/CD) for a Spring microservices project 
is available at [Dzone.com ](https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services).

The details can be viewed in jetpack/deploy.yml, but in short the steps performed by the build server are:
1) Requests a GitHub personal action token - see above [DZone.com] (https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services) 
article
2) Creates temporary keys to clone the specified GitHub repositories
3) Clones the repositories
4) Performs a full "maven install" to cater for multi-moduled Maven projects
5) Creates a Docker image using the Maven JIB plugin
6) Bypasses a Docker repo by pushing the image directly into ContainerD
7) Enforces the various K8S tasks using kubectl, such as creating the namespace, creating the service&deployment and runs 
the pre- and post-commands specified in jetpack/vars.yml

A sample declaration is available at jetpack/vars.yml. It specifies the deployment of the GitHub project located at 
https://github.com/jrb-s2c-github/spinnaker_tryout.

### Build server
A new entry is required in the inventory to designate the server that will build and deploy the container to its ContainerD
daemon.

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

One of the worker nodes should function as the build server to avail the image to its ContainerD.  Pods are after all not 
scheduled on control-planes unless the taint is removed. 

The IP address and location of the ansible user's private key has to be configured as per usual. Note that only one build
server is required.

### Declaration
The various elements of a deployment declaration will be discussed next, each in its own section.

#### Namespaces

```
namespaces:
  - name: hello_ns
```
Each element of this list will be created as a K8S namespace.

#### Maven parents

```
mvn_parents:
- name: hello1/hello_svc
```

Each element of this list represents a maven folder in which maven will be run to install all external dependencies and 
compiled child modules into the build server's local maven repo. The JIB command can then integrate using these local 
artefacts.

Note that the path is given relative to the ansible user's home directory where the project has been cloned. For the 
example above it means that the project has been cloned into ~ansible/hello1/, with hello1 taken from the name given to 
the K8s service/deployment in the jetpack declaration:
```
apps:
  - name: hello1
```
It follows that hello_svc is the subdirectory under ~ansible/hello1 that contains the main pom.

#### Pre- and post-commands

```
pre_k8s_cmds:
- kubectl create -n cc deployment hazelcast --image=hazelcast/hazelcast:latest-snapshot-jdk21 --port=5701
- kubectl expose -n cc deployment hazelcast

post_k8s_cmds:
- kubectl -n cc scale deployment hazelcast --replicas 2
```

These are shell commands that wil be run before and after the normal kubectl commands that create the orchestration 
artefacts.

#### Ingress declaration

A Kubernetes Ingress routes endpoints to microservice endpoints exposed inside the cluster: 

```
ingress:
   host: www.demo.io                   # DNS name to access Ingress on
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

The DNS name of the Ingress (www.demo.io in above snippet) should be mapped by a DNS server or an entry in the hosts file
(/etc/hosts) to the IP address of the Ingress Controller. The following command will provide this external IP:
> kubectl -n ingress-nginx get svc ingress-nginx-controller

This is the same IP address that the [MetalLB](https://metallb.io/) LoadBalancer announces and attracts traffic on. In other words, MetalLb
supports the Ingress of type ingress-nginx in its endeavours.     

#### Declaring Spring Microservice deployments

```
apps:
- name: hello1                   # Name of the K8S Service
  github_account: jrb-s2c-github # GitHub account that contains the repository to clone
  git_repo: spinnaker_tryout     # GitHub repository to clone
  jib_dir: hello_svc             # Use "." for a single module or name of directory containing JIB connfiguration for multi-module maven project   
  image: s2c/hello_svc           # Name of container image  that JIB will create
  namespace: env1                # K8S namespace that the micro-services should be added to
  git_branch: kustomize          # Git branch to checkout
  replicas: 3                    # Amount of micro-services instances to start
  application_properties:        # The application.properties of te Spring micro-service
  application.properties: |
  my_name: LocalKubeletEnv1
- name: hello2                   # Declaration of a second microservice to deploy
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
with the base image. JIB adds the code to this base before publishing the new image.

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

More on the maven JIB plugin can be read at https://github.com/GoogleContainerTools/jib

## Execution of deployment
The command to integrate and deploy is:
>ansible-playbook jetpack/deploy.yml  -i atomika/inventory/****.yml

As always, take care to specify the correct topology inventory to use after the -i switch.

The play will request a GitHub personal access token. Hit enter to bypass all this for public repos or
enter a classic access token for private repos.

Read this [DZone.com](https://dzone.com/articles/safe-clones-with-ansible) 
article for the background, but this will initiate a safe GIT clone. This classic access token should be given the following scopes/permissions: 
*repo, admin:public_key, user, and admin:gpg_key*. 

## Testing Ingress and MetalLB LoadBalancer 
This is performed at the end of the boot, but are documented here for the sake of completion.

1) Run 'kubectl get service ingress-nginx-controller --namespace=ingress-nginx' and check that an IP address has been assigned to field "EXTERNAL-IP". This means MetalLB is listening on this IP address.
2) Run 'kubectl create deployment demo --image=httpd --port=80' to install web server
3) Run 'kubectl expose deployment demo' to expose web server as service
4) Run 'kubectl create ingress demo --class=nginx --rule www.demo.io/=demo:80' to create Ingress resource
5) Determine external IP of Ingress (kubectl -n ingress-nginx get svc ingress-nginx-controller) and add a DNS mapping to 
it in the hosts (/etc/hosts or C:\Windows\System32\drivers\etc\hosts) file
5) Open www.demo.io inside a web browser or on any node in the cluster and check that "It works!" is displayed

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

### V2 @ 2023/10/23
Version two sees:
1) refactored code.
2) easier preparation of target machines to accept instruction from the Ansible controller using bootstrapping. 
explained higher up 

More detail on V2:
1) V2 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V2/README.md)
2) [DZone.com article](https://dzone.com/articles/ansible-boots-kubernetes) that explains how to use Ansible to boot a 
Kubernetes cluster 

### V3 @ 2023/11/23
Version three provided support for high availability topologies.

It, furthermore, improved ease of use with the:
1) Addition of the k8s_boot.yml playbook that ensures all nodes are added to the cluster in the topology, instead of 
having to run playbooks for each stage/type separately
2) Addition of the kubeadm_reset.yml playbook to call "kubeadm reset" on each node in the inventory topology

More detail on V3:
1) V3 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V3/README.md)
2) A [Dzone.com](https://dzone.com/articles/anatomy-of-a-high-availability-kubernetes-cluster) article that details a way 
to establish high availability with a K8S bare-metal cluster such as Atomika.

### V4 @ 2024/01/02
Version 4 added Jetpack that allows one to declare deployments of maven projects to Atomika. 

More detail on V4:
1) V3 [README.md](https://github.com/jrb-s2c-github/atomika/blob/V3/README.md)
2) A [Dzone.com](https://dzone.com/articles/fast-feature-branch-deployments-of-micro-services) explaining how to configure
Jetpack to deploy using Maven, JIB, YAML and Kubectl.

### V5 @ 2024/06/23
1) Upped version of Ubuntu that is supported from 20 to 22
2) Upped version of K8S to 1.30.2
3) Upped version of containerd to 1.6.33-1
4) Upped version of MetalLB to 0.14.5
5) Upped version of ingress-nginx to 1.10.1 
6) Improved documentation in README.md
6) Removed support for cloudinit, since it has been replaced by the more generic bootstrapping playbook (bootstrap.yml)
7) Moved location of inventory into its own sub-directory at atomika/inventory 
8) Removed taints and labels to have Ingress work on single-node clusters as well 
9) Added sample inventories for single-node, single/basic control-plane and high availability clusters 
10) Added sample input yaml for Jetpack (jetpack/vars.yml)
11) Moved kubectl commands from ansible commands to the ansible kubernetes galaxy collection
12) Refactored ansible code

### V5_1 2024/07/15
1) Removed machine reboots steps and removed most of the waits. Cluster formation rarely
required this after all images have been pulled during the first boot.    
2) A separate project was created at [jrb-s2c-github/atomika_wormhole](https://github.com/jrb-s2c-github/atomika_wormhole)
that a) will prepare machines for use as Ansible control Kubernetes nodes and b) allow such machines 
to boot without requiring human interaction.   
3) Added default ssh keys that will be baked into the atomika_wormhole boot image. The onus will be on the user to replace
this keypair after first use should the security requirements warrant it.
4) Improved documentation in general
5) Refactored and simplified code to improve flow of execution. This speeds up cluster formation.
6) Keyscanning of controlled nodes by Ansible controller was implemented by the key_scan.yml playbook.
7) Removed the amount of 'prompts/-e switches' to provide by making cluster formation more opinionated. For instance,
providing a second user to be given a kubeconfig for kubectl commands is not mandatory anymore.
8) Jetpack can clone from public repos without requiring security tokens.

## Outstanding
1) Is it possible to upgrade the cluster K8s version from Ansible? 
2) Graphical user interface to configure bootstrapping, Atomika topology and Jetpack CI/CD
3) Should it be possible to skip "mvn install" step? The JIB command is sufficient for single module projects. 
4) Split atomika_base role out as it should only run once to prepare a target node for orchestration 
5) Add group_vars to hold version info of metallb, ingress-nginx from k8s_ingress_controller.yml, k8s and containerd. Can 
a BOM be generated from this? 
6) Add support for other Linux distro's using some sort of templating, starting with the undocumented ARCH linux/ Raspberry PI's 
7) Jetpack should not delete namespaces everytime, it should only deploy what has changed 
8) Once Ubuntu nodes can be configured from scripts, work on a way to boot a Windows cluster from scratch with one click
from a GUI. 
9) Testing harness 
10) Run docker registry on one node so all images can be pulled from there by the other nodes in the cluster 
11) DNS server to register name of Ingress to remove need to mess with hosts files. This
is only a problem when not using Wormhole. 
12) Integration with ansible lint on some level 
13) Defaulting metallb range to something on the gateway's local subnet 
14) Checking whether things can be sped up by not gathering facts every time?

# Common problems

## Playbook refuse to start due to connection issues
The Ansible controller uses SSH to connect to the target nodes. Common solutions to fix connection failure are:
1) Sign on from the Ansible controller to the target node using the user configured in 
the inventory or perform key scanning to establish trust between the servers. This 
should have been resolved with version 5.1.
2) Check that the ansible user set in the inventory is correct. This should not happen when sticking to wormhole images.
3) Make sure that you created the public/private keys for the user configured for each node in the inventory. Ansible user 
is used in the sample inventories and is therefore recommended way. This should not happen when sticking to wormhole images.
4) Should there be an empty workers group the taints/labels preventing scheduling on masters and MetalLB speakers to announce
will not be removed from masters and the cluster will never stabilize.

## Tips and tricks
* https://zwischenzugs.com/2021/08/27/five-ansible-techniques-i-wish-id-known-earlier/
* Use --start-at-task switch to continue from last successful task after fixing the cause of a failed task:
*ansible-playbook atomika/k8s_boot.yml  -i atomika/inventory/single_node_inventory.yml --start-at-task="Initializing Kubernetes Cluster"*
* Combining --start-at-task with --step is for Ansible superusers
* Add the "-vvv" switch to the ansible-playbook command for verbose feedback.
* The kubectl commands can be run on any control-plane
* Error: "dr: \\\"cni0\\\" already has an IP address different from" ==> run *"ip link delete cni0 && ip link delete flannel.1"*
* **On the master node the cluster can also be interrogated by root: "sudo kubectl 
--kubeconfig /etc/kubernetes/admin.conf get nodes"**. This is especially useful between 
cluster initialization and the copying of the kubeconfig into the home directory of the 
ansible user.
* Command completion of kubectl will be enabled by k8s_boot.yml, however signing out and
in might be required. *On Wormhole command completion works out of the box*. Should you
not use Wormhole, bash completion should be installed (*sudo apt install bash-completion*) 
followed by signing out and back in again. 
* **It is possible to instruct Atomika to assign rights to a second user**, over and above
the ansible user, **to use kubectl**. Append *'-e second_kubectl_user=${second_user}*' to k8s_boot.yml.

## Other things to keep in mind
1) Check that ip addresses are correct in the inventory
2) Check that user account is correct during bootstrapping and that it has been changed
to the ansible user after bootstrapping 
3) Run HAProxy on its own node 
5) Builder server should not be a control-plane unless it is a single node cluster. Should apps image fail to deploy 
(ErrImgPull), this is most likely the reason.
