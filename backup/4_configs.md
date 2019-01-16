
In this lesson we are going to cover the following topics

  * Setting up monitors
  * Configs
  * Context
  * Namespaces


## Setup monitoring console for Kubernetes

Unix **watch** is a great utility for a devops professional. You could setup a simple monitoring for kubernetes cluster using a **watch**  along with **kubectl** command.  Run the following on the managment node. It could be your local system configured with kubectl, or a remote shell on a node where *kubectl* is configured.


```
watch -n 1 kubectl get pods,deploy,svc

```
## Listing Configurations

Check current config
```

kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

## Creating a namespace 

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **mogambo**  

file: k8s-code/projects/mogambo/mogambo-ns.yml
```
kind: Namespace
apiVersion: v1
metadata:
  name: mogambo
```

To create namespace

```
cd k8s-code/projects/mogamgo
kubectl get ns
kubectl apply -f mogamgo-ns.yml
kubectl get ns
```


And switch to it
```
kubectl config set-context $(kubectl config current-context) --namespace=mogambo

```
