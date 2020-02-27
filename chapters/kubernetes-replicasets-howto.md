
# Lab K104 - Adding HA and Scalability with ReplicaSets

If you are not running a monitoring screen, start it in a new terminal with the following command.

```
watch -n 1 kubectl get  pod,deploy,rs,svc
```


### Creating a Namespace and switching to it



Check current config
```

kubectl config view
```

You could also examine the current configs in file **cat ~/.kube/config**

## Creating a namespace

Namespaces offers separation of resources running on the same physical infrastructure into virtual clusters. It is typically useful in mid to large scale environments with multiple projects, teams and need separate scopes. It could also be useful to map to your workflow stages e.g. dev, stage, prod.   

Lets create a namespace called **instavote**  

```
kubectl get ns

kubectl create namespace instavote

kubectl get ns

```
And switch to it

```

kubectl config --help

kubectl config get-contexts

kubectl config current-context

kubectl config set-context --help

kubectl config set-context --current --namespace=instavote

kubectl config get-contexts

kubectl config view


```


**Exercise**: Go back to the monitoring screen and observe what happens after switching the namespace.


To understand how ReplicaSets works with the selectors  lets launch a pod in the new namespace with existing specs.

```
cd k8s-code/pods
kubectl apply -f vote-pod.yaml

kubectl get pods
```

## Adding ReplicaSet Configurations

Lets now write the spec for the Rplica Set. This is going to mainly contain,

  * replicas
  * selector
  * template (pod spec )
  * minReadySeconds


From here on, we would switch to the project and environment specific path and work from there.


```
cd projects/instavote/dev

```


`edit file: vote-rs.yaml`

```
apiVersion: xxx
kind: xxx
metadata:
  xxx
spec:
  xxx
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
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```

Above file already containts the spec that you had written for the pod. You would observe its already been added as part of *spec.template* for replicaset.

Lets now add the details specific to replicaset.

*file: vote-rs.yaml*

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: vote
spec:
  replicas: 4
  minReadySeconds: 20
  selector:
    matchLabels:
      role: vote
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3, v4, v5]}
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
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "250m"
```

The complete file will look similar to above. Lets now go ahead and apply it.


```
kubectl apply -f vote-rs.yaml --dry-run

kubectl apply -f vote-rs.yaml

kubectl get rs

kubectl describe rs vote

kubectl get pods

kubectl get pods --show-labels
```

### High Availability

Try deleting pods created by the replicaset,

`replace pod-xxxx and pod-yyyy with actuals`
```
kubectl get pods

kubectl delete pods vote-xxxx vote-yyyy
```
Observe as the pods are automatically created again.


Lets now delete the pod created independent of replica set.

```
kubectl get pods
kubectl delete pods  vote
```

Observe what happens.
  * Does replica set take any action after deleting the pod created outside of its spec ? Why?

### Exercise: Deploying new version of the application


```
kubectl edit rs/vote
```

Update the version of the image from **schoolofdevops/vote:v1** to **schoolofdevops/vote:v2**

Save the file.

Observe what happens ?

  * Did application get  updated.
  * Did updating replicaset launched new pods to deploy new version ?


### Scalability

Scaling up application is as easy as running,  

```
kubectl scale --replicas=8 rs/vote

kubectl get pods --show-labels
```  

Observe what happens

  * Did the number of  replicas increase to 8 ?
  * Which version of the app are the new pods running with ?


#### Summary

With **ReplicaSets** your application is now high available as well as scalable. However ReplicaSet by itself does not have the intelligence to trigger a rollout if you update the version. For that, you are going to need a **deployment** which is something you would learn in an upcoming  lesson.
