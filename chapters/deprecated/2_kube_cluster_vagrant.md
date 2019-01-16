# Install VirtualBox and Vagrant

| TOOL  | VERSION  |  LINK |
|---|---|---|
| VirtualBox  |   5.1.26  |   https://www.virtualbox.org/wiki/Downloads |
| Vagrant  | 1.9.7   | https://www.vagrantup.com/downloads.html   |  




## Importing a VM Template

If you have already copied/downloaded the box file **ubuntu-xenial64.box**, go to the directory which contains that file. If you do not have a box file, skip to next section.

```
vagrant box list

vagrant box add ubuntu/xenial64 ubuntu-xenial64.box

vagrant box list

```

## Provisioning Vagrant Nodes

Clone repo if not already

```
git clone https://github.com/schoolofdevops/lab-setup.git


```

Launch environments with Vagrant

```
cd lab-setup/kubernetes/vagrant-kube-cluster

vagrant up

```

Login to nodes

Open three different terminals to login to 3 nodes created with above command

**Terminal 1**
```
vagrant ssh kube-01
sudo su

```
**Terminal 2**

```
vagrant ssh kube-02
sudo su
```

**Terminal 3**

```
vagrant ssh kube-03
sudo su
```


Once the environment is setup, follow **Initialization of Master** onwards from the following tutorial
https://github.com/schoolofdevops/kubernetes-fundamentals/blob/master/tutorials/1.%20install_kubernetes.md
