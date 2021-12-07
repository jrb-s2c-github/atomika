# Atomika

Ansible scripts to boot a local Kubernetes cluster for use as a shared development environment and/or Spinnaker CI/CD constellation:

1) Edit the inventory file to add master node, co-master nodes for high availability and worker nodes.
2) Run k8s-containerd_pkg.yml to install and prepare each of the node
3) Run k8s_master_init.yml to boot up the master node
4) (Optional) Run k8s_comasters.yml to join the two other master nodes to enable high availability
5) Run k8s_workers.yml to join the worker nodes

In the multipass folder is a cloud_init.yml file to prepare ubuntu nodes to receive intsruction from Ansible.
The master key is in the master key folder.




