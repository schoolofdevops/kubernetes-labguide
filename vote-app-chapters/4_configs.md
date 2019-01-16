
In this lesson we are going to cover the following topics

  * Setting up monitors
  * Configs
  * Context
  * Namespaces


## Setup monitoring console for Kubernetes
Unix **screen** is a great utility for a devops professional. You could setup a simple monitoring for kubernetes cluster using a **screenrc** script as follows on kube-01 node (node where *kubectl* is configured)

file: k8s-code/monitoring/deploy.screenrc

```
screen watch -n 1 kubectl get pods
split
focus down
screen watch -n 1 kubectl get deploy
split
focus down
screen watch -n 1 kubectl get services
split
focus down
screen watch -n 1 kubectl get replicasets
focus bottom
```

Open a dedicated terminal to run this utility.  Launch it using

```
screen -c monitoring/deploy.screenrc

```


### Working with Unix Screen

To detach

```
^a d  
```

To list existing screen sessions
```
screen -ls
```

To re attach
```
screen  -x <session_id>
```

To delete a session

```
screen -X -S <session_id> quit
```

e.g.
```
screen -ls
There are screens on:
	26484.pts-2.kube-01	(01/12/2018 06:47:41 AM)	(Detached)
	18472.pts-2.kube-01	(01/12/2018 06:43:21 AM)	(Detached)
2 Sockets in /var/run/screen/S-root.

screen -X -S 26 quit
```


## Listing Configurations

Check current config
```
kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

## Creating a namespace for instavote project

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **instavote**  

file: k8s-code/projects/instavote/instavote-ns.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: instavote
```

To create namespace

```
cd k8s-code/projects/instavote
kubectl get ns
kubectl apply -f dev_instavote.yaml
kubectl get ns
```


And switch to it
```
kubectl config set-context $(kubectl config current-context) --namespace=instavote

```
