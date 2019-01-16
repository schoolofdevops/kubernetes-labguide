# Kubernetes - Cluster Setup using Kubespray

Kubespray is an *Ansible* based kubernetes provisioner. It helps us to setup a production grade, highly available and highly scalable Kubernetes cluster.

## Prerequisites

Hardware Pre requisites
  * 4 Nodes: Virtual/Physical Machines
  * Memory: 2GB each
  * CPU: 2 cores recommended
  * Hard disk: 20GB available

Software Pre Requisites
  * Ubuntu 16.04 Operating System
  * Python

## Architecture of a high available kubernetes cluster


## Preparing the kubernetes nodes

`On control node`

Ansible is the base provisioner in our cluster. But installing Ansible is out of the scope of this training. You can learn about installing and configuring Ansible from [here.](http://docs.ansible.com/ansible/latest/intro_installation.html)

### Setup passwordless SSH between control and kubernetes nodes

`On control node`

Ansible uses passwordless ssh<sup>1</sup> to create the cluster. Let us see how to set it up from your *control node*.
```
ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:yC4Tl6RYc+saTPcLKFdGlTLOWOIuDgO1my/NrMBnRxA ubuntu@node1
The key's randomart image is:
+---[RSA 2048]----+
|   E    ..       |
|  . o +..        |
| . +o*+o         |
|. .o+Bo+         |
|. .++.X S        |
|+ +ooX .         |
|.=.OB.+ .        |
| .=o*= . .       |
|  .o.   .        |
+----[SHA256]-----+
```

Just leave the fields to defaults. This command will generate a public key and private key for you.

```
cat ~/.ssh/id_rsa.pub | ssh ubuntu@10.40.1.26 'cat >> ~/.ssh/authorized_keys'
```

This will copy our newly generated public key to the remote machine. After running this command you will be able to SSH into the machine directly without using a password. Replace *10.40.1.26* with your respective machine's IP.

### Enable IPv4 Forwarding

`On all nodes`

```
ssh ubuntu@10.40.1.26
sudo su
vim /etc/sysctl.conf
```
Enalbe IPv4 forwarding by uncommenting the following line

```
net.ipv4.ip_forward=1
```

### Stop and Disable UFW Service

`On all nodes`

Execute the following commands to stop and disable ufw service.

```
systemctl stop ufw.service
systemctl disable ufw.service
```

### Install Python

`On all nodes`

Ansible needs python to be installed on all the machines.

```
sudo apt update
sudo apt install python
```

## Configuring Ansible Control node and Kubespray


`On control node`

Kubespray is hosted on GitHub. Let us the clone the [official repository](https://github.com/kubernetes-incubator/kubespray.git).

```
git clone https://github.com/kubernetes-incubator/kubespray.git
cd kubespray
```

### Set Remote User for Ansible

`On control node`

Add the following section in ansible.cfg file

```
remote_user=ubuntu
```

Your *ansible.cfg* file should look like this.

```
[ssh_connection]
pipelining=True
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ConnectionAttempts=100 -o UserKnownHostsFile=/dev/null
#control_path = ~/.ssh/ansible-%%r@%%h:%%p
[defaults]
host_key_checking=False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp
stdout_callback = skippy
library = ./library
callback_whitelist = profile_tasks
roles_path = roles:$VIRTUAL_ENV/usr/local/share/kubespray/roles:$VIRTUAL_ENV/usr/local/share/ansible/roles
deprecation_warnings=False
remote_user=ubuntu
```

### Download Inventory Builder

`On control node`

Inventory builder (a python script) helps us to create inventory file. **Inventory** file is something with which we specify the groups of masters and nodes of our cluster.

```
cd inventory
wget https://raw.githubusercontent.com/kubernetes-incubator/kubespray/master/contrib/inventory_builder/inventory.py
```

To build the inventory file, execute the inventory script along with the IP addresses of our cluster as arguments

```
python inventory.py 10.40.1.26 10.40.1.25 10.40.1.20 10.40.1.11
```

These IPs are different for you. Please replace them with your corresponding IPs.
This step will result in the creation of a new file **inventory.cfg**. This is our inventory file.

`inventory.cfg`
```
[all]
node1    ansible_host=10.40.1.26 ip=10.40.1.26
node2    ansible_host=10.40.1.25 ip=10.40.1.25
node3    ansible_host=10.40.1.20 ip=10.40.1.20
node4    ansible_host=10.40.1.11 ip=10.40.1.11

[kube-master]
node1
node2

[kube-node]
node1
node2
node3
node4

[etcd]
node1
node2
node3

[k8s-cluster:children]
kube-node
kube-master

[calico-rr]

[vault]
node1
node2
node3
```

## Provisioning  kubernetes cluster with kubespray

`On control node`

We are set to provision the cluster. Run the following ansible-playbook command to provision our Kubernetes cluster.

```
ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v
```

Option -i = Inventory file path
Option -b = Become as root user
Option -v = Give verbose output

This Ansible run will take around 30 mins to complete.

## Install Kubectl

`On control node`

Before we proceed further, we will need to install **kubectl** binary in our control node. Read installation procedure from this [link](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

## Getting the Kubernetes Configuration File

`On control node`

Once the cluster setup is done, we have to copy over the cluster config file from the master machine. We will discuss about this file extensively in the next chapter.

```
ssh ubuntu@10.40.1.26
sudo su
cp /etc/kubernetes/admin.conf /home/ubuntu
chown ubuntu:ubuntu /home/ubuntu/admin.conf
exit
exit
scp ubuntu@10.40.1.26:~/admin.conf .
cd
mkdir .kube
mv admin.conf .kube/config
```

## Check the State of the Cluster

`On control node`

Let us check the state of the cluster by running,

```
kubectl cluster-info

Kubernetes master is running at https://10.40.1.26:6443
KubeDNS is running at https://10.40.1.26:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
kubectl get nodes

NAME      STATUS    ROLES         AGE       VERSION
node1     Ready     master,node   21h       v1.9.0+coreos.0
node2     Ready     master,node   21h       v1.9.0+coreos.0
node3     Ready     node          21h       v1.9.0+coreos.0
node4     Ready     node          21h       v1.9.0+coreos.0
```

If you are able to see this, your cluster has been set up successfully.

---
<sup>1</sup> You can use private key / password instead of passwordless ssh. But it requires additional knowledge in using Ansible.
