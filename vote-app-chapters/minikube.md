# Single node k8s cluster with Minikube

Minikube offers one of the easiest zero to dev experience
to setup a single node kubernetes cluster. Its also the ideal way
to create a local dev environment to test kubernetes code on.

This document explains how to setup and work with single node kubernetes cluster with minikube.

## Install Minikube

Instructions to install minikube may vary based on the operating system and choice of the hypervisor.
[This is the official document](https://kubernetes.io/docs/tasks/tools/install-minikube/) which explains how to install minikube.

## Start all in one single node cluster with minikube

```
minikube status
```

[output]

```
minikube:
cluster:
kubectl:
```


```
minikube start
```

[output]
```
Starting local Kubernetes v1.8.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

```
minikube status

```

[output]
```
minikube: Running
cluster: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
```

#### Launch a kubernetes dashboard

```
minikube dashboard
```


#### Setting up docker environment

```
minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/gouravshah/.minikube/certs"
export DOCKER_API_VERSION="1.23"
# Run this command to configure your shell:
# eval $(minikube docker-env)
```

Run the command given above,
e.g.

```
eval $(minikube docker-env)
```

Now your docker client should be able to connect with the minikube cluster

```
docker ps
```

### Additional Commands

```
minikube ip
minikube get-k8s-versions
minikube logs
```
