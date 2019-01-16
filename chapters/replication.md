
# Making application high available with Replication Controllers

If you are not running a monitoring screen, start it in a new terminal with the following command.

```
watch -n 1 kubectl get  pod,deploy,rs,svc
```


```
kubectl delete pod vote
```


### Setting up a Namespace



Check current config
```

kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

## Creating a namespace

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **instavote**  



```
cd projects/instavote
cat instavote-ns.yaml
```

[output]
```
kind: Namespace
apiVersion: v1
metadata:
  name: instavote
```

Lets create a namespace

```
kubectl get ns
kubectl apply -f instavote-ns.yaml

kubectl get ns
```



And switch to it
```
kubectl config --help

kubectl config get-contexts

kubectl config current-context

kubectl config set-context $(kubectl config current-context) --namespace=instavote

kubectl config view

kubectl config get-contexts

```

**Exercise**: Go back to the monitoring screen and observe what happens after switching the namespace.


To understand how ReplicaSets works with the selectors  lets launch a pod in the new namespace with existing specs.

```
cd k8s-code/pods
kubectl apply -f vote-pod.yaml

kubectl get pods
cd ../projects/instavote/dev/
```

Lets now write the spec for the Rplica Set. This is going to mainly contain,

  * replicas
  * selector
  * template (pod spec )
  * minReadySeconds



*file: vote-rs.yaml*

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
spec:
  replicas: 5
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
  template:
```


Lets now add the metadata and spec from pod spec defined in vote-pod.yaml. And with that, the Replica Set Spec changes to

*file: vote-rs.yaml*

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
spec:
  replicas: 5
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
  template:
    metadata:
      name: vote
      labels:
        app: python
        role: vote
        version: v1
    spec:
      containers:
        - name: app
          image: schoolofdevops/vote:v1
          ports:
            - containerPort: 80
              protocol: TCP

```

### Replica Sets in Action


```
kubectl apply -f vote-rs.yaml --dry-run

kubectl apply -f vote-rs.yaml

kubectl get rs

kubectl describe rs vote

kubectl get pods


```

**Exercise** :  

  * Switch to monitoring screen, observe how many replicas were created  and why
  * Compare selectors and labels of the pods created with and without replica sets

```
kubectl get pods

kubectl get pods --show-labels
```

### Exercise: Deploying new version of the application


```
kubectl edit rs/vote
```

Update the version of the image from **schoolofdevops/vote:v1** to **schoolofdevops/vote:v2**

Save the file. Observe if application got updated. Note what do you observe. Do you see the new version deployed ??


### Exercise: Self Healing Replica Sets

List the pods and kill some of those, see what replica set does.

```
kubectl get pods
kubectl delete pods  vote-xxxx  vote-yyyy
```

where replace xxxx and yyyy with actual pod ids.

Questions:

  * Did replica set replaced the pods ?
  * Which version of the application is running now ?


Lets now delete the pod created independent of replica set.
```
kubectl get pods
kubectl delete pods  vote
```

Observe what happens.
  * Does replica set take any action after deleting the pod created outside of its spec ? Why?
