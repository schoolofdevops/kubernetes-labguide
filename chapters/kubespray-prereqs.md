# Provisioning HA Lab Cluster  with Vagrant

## Vagrant Setup:

This tutorial assumes you have Vagrant+VirtualBox setup. While Vagrant is used for basic infrastructure requirements, the lessons learned in this tutorial can be applied to other platforms. Start from [Set up Kubernetes Using Kubespray](#Settin-Up-Kubernetes-Using-Kubespray)(or) Refer to this [Document](https://github.com/schoolofdevops/ultimate-kubernetes-bootcamp/blob/master/chapters/cluster_setup_kubespray.md), if you have VMs running elsewhere

### Software Requirements on Host Machine:
   * Virtual Box (latest)
   * Vagrant (latest)
   * Git Bash (Only for Windows)
   * Conemu (Only for Windows)

### Set up Learning Environment:

Setup the repo

```
git clone https://github.com/schoolofdevops/kubespray-1

```

Bring up the VMs

```
cd kubespray-1

vagrant up

vagrant status
```

Login to nodes

Open four different terminals to login to 4 nodes created with above command

**Terminal 1**
```
vagrant ssh k8s-01
sudo su

```
**Terminal 2**

```
vagrant ssh k8s-02
sudo su
```

**Terminal 3**

```
vagrant ssh k8s-03
sudo su
```


**Terminal 4**

```
vagrant ssh k8s-04
sudo su
```


Once the environment is setup, follow [**Production Grade Setup with Kubespray**](https://schoolofdevops.github.io/ultimate-kubernetes-bootcamp/cluster_setup_kubespray/)
