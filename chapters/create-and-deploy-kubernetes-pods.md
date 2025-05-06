# Launching Pods with Kubernetes

In this lab, you would learn how to launch applications using  the basic deployment unit of kubernetes i.e. **pods**. This time, you are going to do it by writing declarative configs with *yaml* syntax.

## Launching Pods Manually  

You could use ``` kubectl run ``` to launch a pod by specifying just the image.  

For example, if you would like to launch a pod for redis, with image redis:alpine, the following command would work,

```
kubectl run redis  --image=redis
```


## Kubernetes Resources and writing YAML specs

Each entity created with kubernetes is a resource including pod, service, deployments, replication controller etc. Resources can be defined as YAML or JSON.  Here is the syntax to create a YAML specification.

**AKMS** => Resource Configs Specs

```
apiVersion: v1
kind:
metadata:
spec:
```

Use this [Kubernetes API Reference Document](https://kubernetes.io/docs/reference/) while writing the API specs. This is the most important reference  while working with kubernetes, so its highly advisible that you bookmark it for the version of kubernetes that you use.

To find the version of kubernetes use the following command,

```
kubectl version -o yaml
```

To list the running pods,

```
kubectl get pods

```

To list API objects, use the following commands,

```
kubectl api-resources
```

## LAB PART I 

### Writing Pod Spec

Lets now create the  Pod config by adding the kind and specs to schma given in the file vote-pod.yaml as follows.

Filename: k8s-code/pods/vote-pod.yaml
```
apiVersion:
kind: Pod
metadata:
spec:
```


**Problem Statement:** Create a YAML spec to launch a  pod with one container to run vote application, which matches the following specs.

  * pod:
    * metadata:
      * name: vote
      * labels:
        * app: python
        * role: vote
        * version: v1
  * containers:
    * name: app
    * image: schoolofdevops/vote:v1



Refer to the [Pod Spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#pod-v1-core) find out the relevant properties and add it to the **vote-pod.yaml** provided in the supporting code repo.

`Filename: k8s-code/pods/vote-pod.yaml`

```
apiVersion: v1
kind: Pod
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
```

[Use this example link to refer to pod spec](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.13/#pod-v1-core)


### Launching and operating Pods


To launch a monitoring screen to see whats being launched, use the following command in a new terminal window where kubectl is configured.


```
watch kubectl get all

```

kubectl Syntax:

```
kubectl
kubectl apply --help
kubectl apply -f FILE
```


to do a dry run before applying use the following command,

```
kubectl apply -f vote-pod.yaml --dry-run=client
```

to finally  **launch** pod using configs above, remove dry run options and run as

```
kubectl apply -f vote-pod.yaml

```

To **view** pods

```
kubectl get pods

kubectl get po

kubectl get pods -o wide

kubectl get pods --show-labels

kubectl get pods -l "role=vote,version=v1"

kubectl get pods vote
```

To get detailed info

```
kubectl describe pods vote
```


Commands to  **operate** the pod

```

kubectl logs vote

kubectl exec -it vote  sh


```

Run the following commands inside the container in a pod after running exec command as above

```
ifconfig
cat /etc/issue
hostname
cat /proc/cpuinfo
ps aux
```

`use ^d or exit to log out`


## LAB PART II

## Adding a Volume for data persistence

Lets create a pod for database and attach a volume to it. To achieve this we will need to

  * create a **volumes** definition
  * attach volume to container using **VolumeMounts** property

Local host volumes are of two types:  

  * emptyDir  
  * hostPath  

We will pick hostPath. [Refer to this doc to read more about hostPath.](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)


File: db-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    app: postgres
    role: database
    tier: back
spec:
  containers:
    - name: db
      image: postgres:9.4
      ports:
        - containerPort: 5432
      env:
        - name: "POSTGRES_PASSWORD"
          value: "somepassword"
      volumeMounts:
      - name: db-data
        mountPath: /var/lib/postgresql/data
  volumes:
  - name: db-data
    hostPath:
      path: /var/lib/pgdata
      type: DirectoryOrCreate
```

To create this pod,

```
kubectl apply -f db-pod.yaml

kubectl get pods
```

At this time, you may see the pod in ```CrashLoopBackOff``` state. Try debugging the issue by checking the logs and resolve it before proceeding.

Once resolved, check the pod is in running state
```
kubectl get pods

kubectl describe pod db

kubectl get events
```


**Exercise** : Examine **/var/lib/pgdata** on the node on which its scheduled (use ```kubectl get pods -o wide``` to find the node) to check if the directory is been created and if the data is present.



## Creating Multi Container Pods - Sidecar Example

file: multi_container_pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    tier: front
    app: nginx
    role: ui
spec:
  containers:
    - name: nginx
      image: nginx:stable-alpine
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - name: data
          mountPath: /var/www/html-sample-app

    - name: sync
      image: schoolofdevops/sync:v2
      volumeMounts:
        - name: data
          mountPath: /var/www/app

  volumes:
    - name: data
      emptyDir: {}
```

To create this pod

```
kubectl apply -f multi_container_pod.yaml
```

Check Status

```
root@kube-01:~# kubectl get pods
NAME      READY     STATUS              RESTARTS   AGE
web     0/2       ContainerCreating   0          7s
vote      1/1       Running             0          3m
```

Checking logs, logging in
```
kubectl logs  web  -c sync
kubectl logs  web  -c nginx

kubectl exec -it web  sh  -c nginx
kubectl exec -it web  sh  -c sync

```

Observe whats common and whats isolated in two containers running inside  the same pod using the following commands,

shared
```
hostname
ifconfig
```

isolated
```
cat /etc/issue
ps aux
df -h

```

## Adding Resource requests and limits

We can control the amount of resource requested and also put a limit on the maximum a container in a pod could take up.  This can be done by adding to the existing pod spec as below. Refer to [official document on resource management](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) here.


`Filename: vote-pod.yaml`

```
apiVersion: v1
kind: Pod
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

Lets apply the changes now

```
kubectl apply -f vote-pod.yaml
```

If you already have **vote** pod running, you may see an output similar to below,

[sample output]

```
The Pod "vote" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations` (only additions to existing tolerations)
{"Volumes":[{"Name":"default-token-snbj4","HostPath":null,"EmptyDir":null,"GCEPersistentDisk":null,"AWSElasticBlockStore":null,"GitRepo":null,"Secret":{"SecretName":"default-token-snbj4","Items":null,"DefaultMode":420,"Optional":null},"NFS":null,"ISCSI":null,"Glusterfs":null,"PersistentVolumeClaim":null,"RBD":null,"Quobyte":null,"FlexVolume":null,"Cinder":null,"CephFS":null,"Flocker":null,"DownwardAPI":null,"FC":null,"AzureFile":null,"ConfigMap":null,"VsphereVolume":null,"AzureDisk":null,"PhotonPersistentDisk":null,"Projected":null,"PortworxVolume":null,"ScaleIO":null,"StorageOS":null}],"InitContainers":null,"Containers":[{"Name":"app","Image":"schoolofdevops/vote:v1","Command":null,"Args":null,"WorkingDir":"","Ports":null,"EnvFrom":null,"Env":null,"Resources":{"Limits":
....
...
```

From the above output, its clear that not all the fields are mutable(except for a few e.g labels). Container based deployments primarily follow concept of **immutable deployments**. So to bring your change into effect, you need to re create the pod as,

```
kubectl delete pod vote

kubectl apply -f vote-pod.yaml

kubectl describe pod vote
```

From the output of the describe command above, you could confirm the resource constraints you added are in place.

#### Exercise

    * Define the value of **cpu.request** > **cpu.limit** Try to apply and observe.
    * Define the values for **memory.request** and **memory.limit** higher than the total system memory. Apply and observe the deployment and pods.  


### Deleting Pods

Now that you  are done experimenting with pod, **delete** it with the following command,

```
kubectl delete pod vote web db

kubectl get pods
```

#### Reading List

  * [PodSpec:](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#pod-v1-core)
  * [Managing Volumes with Kubernetes](https://kubernetes.io/docs/concepts/storage/volumes)
  * [Node Selectors, Affinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node)
