
In this lesson we are going to cover the following topics

  * Setting up monitors
  * Configs
  * Context
  * Namespaces


## Setup monitoring console for Kubernetes
Unix **screen** is a great utility for a devops professional. You could setup a simple monitoring for kubernetes cluster using a **screenrc** script as follows on kube-01 node (node where *kubectl* is configured)

file: k8s-code/monitoring/rc.screenrc

```
screen watch -n 1 kubectl get pods
split
focus down
screen watch -n 1 kubectl get rc
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

## Creating a dev namespace

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **dev**  

file: dev_ns.yaml
```
kind: Namespace
apiVersion: v1
metadata:
  name: dev
  labels:
    name: dev
```

To create namespace

```
kubectl apply -f dev_ns.yaml
```


And switch to it
```
kubectl config set-context $(kubectl config current-context) --namespace=dev

```
