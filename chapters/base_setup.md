# Base Setup
`Skip this step if using a pre configured lab environment`

**Skip this step and scroll to Initializing Master if you have setup nodes with vagrant**


On all nodes which would be part of this cluster, you need to do the base setup as described in the following steps. To simplify this, you could also   [download and run this script](https://gist.github.com/initcron/40b71211cb693f541ce35fe3fb1adb11)

### Create Kubernetes Repository
`Skip this step if using a pre configured lab environment`


We need to create a repository to download Kubernetes.

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
```
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
```


### Installation of the packages
`Skip this step if using a pre configured lab environment`

We should update the machines before installing so that we can update the repository.
```
apt-get update -y
```
Installing all the packages with dependencies:
```
apt-get install -y docker.io kubelet kubeadm kubectl kubernetes-cni
```
```
rm -rf /var/lib/kubelet/*
```

### Setup sysctl configs
`Skip this step if using a pre configured lab environment`

In order for many container networks to work, the following needs to be enabled on each node.

```
sysctl net.bridge.bridge-nf-call-iptables=1
```
The above steps has to be followed in all the nodes.

`Begin from the following step if using a pre configured lab environment`

