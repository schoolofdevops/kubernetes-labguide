# Lab K208 - Kubernetes Cluster Administration


## Defining Quotas

Create and switch to a new **staging** namespace.

```
config get-contexts

kubectl create namespace staging

kubectl config set-context --current --namespace=staging

config get-contexts
```

Define quota

`file: staging-quota.yaml`
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: staging
  namespace: staging
spec:
  hard:
    requests.cpu: "0.5"
    requests.memory: 500Mi
    limits.cpu: "2"
    limits.memory: 2Gi
    count/deployments.apps: 1
```


```
kubectl get quota -n staging
kubectl apply -f staging-quota.yaml
kubectl get quota -n staging
kubectl describe  quota
```


`file: nginx-deploy.yaml`


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: staging
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: nginx
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          limits:
            memory: "500Mi"
            cpu: "500m"
          requests:
            memory: "200Mi"
            cpu: "200m"
```


```
kubectl apply -f nginx-deploy.yaml
kubectl describe  quota -n staging


```

Lets now try to scale up the deployment and observe.

```
kubectl scale deploy nginx --replicas=4

kubectl get deploy

NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   2/4     2            2           2m55s
```

What happened ?

  * Even though deployment updated the number of desired replicas, only  2 are available
  * Deployment calls replicaset to launch new replicas. If you describe the replicaset it throws an error related to quota being exceeded.

e.g.

```
# kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-56c479cd4f   4         2         2       5m4s

# kubectl describe rs nginx-56c479cd4f

  Warning  FailedCreate      34s (x5 over 73s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-56c479cd4f-kwf9h" is forbidden: exceeded quota: staging, requested: requests.cpu=200m,requests.memory=200Mi, used: requests.cpu=400m,requests.memory=400Mi, limited: requests.cpu=500m,requests.memory=500Mi
```

You just configured  resource quota based on a  namespace. Now, switch back your namespace to **instavote** or the the one you were using before the beginning of this lab.  

```
kubectl config set-context --current --namespace=instavote
kubectl config get-contexts
```



## Nodes  Maintenance

You could isolate a problematic node for further troubleshooting by **cordonning** it off. You could also **drain** it while preparing for maintenance.


### Cordon a Node

```
kubectl get pods -o wide

NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          5h        10.233.74.74   node4
redis-5bf748dbcf-ckn65    1/1       Running   0          42m       10.233.71.26   node3
redis-5bf748dbcf-vxppx    1/1       Running   0          1h        10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          5h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          1h        10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          50m       10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          50m       10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          50m       10.233.74.81   node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   0          39m       10.233.75.15   node2
```

Lets cordon one of the nodes and observe.

```
kubectl cordon node4
node/node4 cordoned
```

Observe the changes

```

$ kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP             NODE
db-66496667c9-qggzd       1/1       Running   0          5h        10.233.74.74   node4
redis-5bf748dbcf-ckn65    1/1       Running   0          43m       10.233.71.26   node3
redis-5bf748dbcf-vxppx    1/1       Running   0          1h        10.233.74.79   node4
result-5c7569bcb7-4fptr   1/1       Running   0          5h        10.233.71.18   node3
result-5c7569bcb7-s4rdx   1/1       Running   0          5h        10.233.74.75   node4
vote-56bf599b9c-22lpw     1/1       Running   0          1h        10.233.74.80   node4
vote-56bf599b9c-4l6bc     1/1       Running   0          51m       10.233.74.83   node4
vote-56bf599b9c-bqsrq     1/1       Running   0          51m       10.233.74.82   node4
vote-56bf599b9c-xw7zc     1/1       Running   0          51m       10.233.74.81   node4
worker-6cc8dbd4f8-6bkfg   1/1       Running   0          40m       10.233.75.15   node2



$ kubectl get nodes -o wide


NAME      STATUS                     ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready                      master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready                      node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready,SchedulingDisabled   node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2


```

Now launch a new deployment and scale it.

```
kubectl create deployment cordontest --image=busybox --replicas=5
kubectl scale deploy cordontest --replicas=5

kubectl get pods -o wide
```

what happened ?

  * New pods scheduled due the deployment above, do not get launched on the node which is been cordoned off.

```
$ kubectl uncordon node4
node/node4 uncordoned


$ kubectl get nodes -o wide
NAME      STATUS    ROLES         AGE       VERSION   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
node1     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node2     Ready     master,node   1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
node3     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-130-generic   docker://17.3.2
node4     Ready     node          1d        v1.10.4   <none>        Ubuntu 16.04.4 LTS   4.4.0-124-generic   docker://17.3.2
```


delete the test  deployment

```
kubectl delete deploy cordontest
```


### Drain a Node

Draining a node will not only mark it unschedulable but also will evict existing pods running on it. Use it with care.

```
$ kubectl drain node3
node/node3 cordoned
error: unable to drain node "node3", aborting command...

There are pending nodes to be drained:
 node3
error: pods with local storage (use --delete-local-data to override): kubernetes-dashboard-55fdfd74b4-jdgch; DaemonSet-managed pods (use --ignore-daemonsets to ignore): calico-node-4f8xc

```

Drain with options

```
kubectl drain node3 --delete-local-data --ignore-daemonsets
```

Observe the effect,

```
kubectl get pods -o wide
kubectl get nodes -o wide

```

To add the node back to the available schedulable node pool,

```
kubectl uncordon node4
node/node4 uncordoned

```


#### Summary

In this lab, we learnt about limiting resource by defining per namespace quota, as well as learnt how to prepare nodes for maintenance by cordoning and draining it.
