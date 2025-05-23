# Create Kubernetes Cluster with KIND

This lab describes the process of how you could quickly create a multi node Kubernetes Envonment using [KIND](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx), which is a simple and quick way to set up a learning environment. Advantage it offers over minikube or docker desktop based kubernetes setup is its a multi node environment closer to the real world setup.  


## Clean up Containers

Before proceeding, clean up any containers running on the host.

`be warned that the following command would DELETE ALL CONTAINRES on the host`

```
docker rm -f $(docker ps -aq)
```

## Install Kubectl and KIND

To install `kubectl` client, refer to the official documentation here [Install Tools | Kubernetes](https://kubernetes.io/docs/tasks/tools/)

Validate by running
```
kubectl version --client=true

kubectl version --client=true -o yaml

```


Install KinD (Kubernetes inside Docker)  using operating specific instructions at  [kind – Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) .

Validate by running

```
kind
```


## Setup Kubernetes Cluster with KIND

Download  Cluster Configurations and Create  a 3 Node Kubernetes Cluster as

```
git clone https://github.com/initcron/k8s-code.git
cd k8s-code/helper/kind/
kind create cluster --config kind-three-node-cluster.yaml
```

Validate

```
kind get clusters
kubectl cluster-info --context kind-kind
kubectl get nodes
kubectl get pods -A 
```

[sample output]

```
root@demo:~# kubectl get nodes
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   78s   v1.19.1
kind-worker          Ready    <none>   47s   v1.19.1
kind-worker2         Ready    <none>   47s   v1.19.1
```

Wait till you see all nodes in Ready state and you have a cluster operational.

Wait for a couple of minutes and then validate if the nodes are up and running.

Setup Visualiser

```
cd ~
git clone  https://github.com/schoolofdevops/kube-ops-view
kubectl apply -f kube-ops-view/deploy/
```

To check whether visualiser has come up, use the following commands,

```
kubectl get pods,services
```

[Expected output ]

```
[root@bbb-01 ~]# kubectl get pods,services
NAME                                 READY   STATUS    RESTARTS   AGE
pod/kube-ops-view-65466fb5c9-7gwnm   1/1     Running   0          61s

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kube-ops-view   NodePort    10.96.54.166   <none>        80:32000/TCP   61s
service/kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        4m28s
```

To access the visualiser, visit  http://IPADDRESS:32000 ( where replace IPADDRESS with the actual hostname or IP of the docker host).

You shall see a visualiser similar to the following loaded on the browser.
![](Screenshot%202022-02-25%20at%208.14.45%20PM.png)

If you see this page , Congratulations !! You have the cluster setup.

## Restarting and Resetting the Cluster (Skip)

`Note: This is a Optional Topic. Skil this during your initial setup lab.`

To stop and start the cluster, you could stop and containers created with docker and then start them back

```
docker ps
docker stop kind-control-plane kind-worker kind-worker2
```

to bring it back again,

```
docker start kind-control-plane kind-worker kind-worker2
```

Even if you restart your system and bring it up using the above command, it should work.

To reset the cluster (note you will be deleting the existing environment and create fresh one)

asusming your cluster name is `k8slab` reset it as :

```
kind get clusters
kind delete cluster --name k8slab
rm -rf  ~/.kube
kind create cluster --name k8slab --config kind-three-node-cluster.yaml
```
