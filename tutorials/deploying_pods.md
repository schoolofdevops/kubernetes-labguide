## Deploying Pods

Life of a pod

  * Pending : in progress
  * Running
  * Succeeded : successfully exited
  * Failed
  * Unknown

### Probes
  * livenessProbe : Containers are Alive
  * readinessProbe : Ready to Serve Traffic



### Resource Configs

Each entity created with kubernetes is a resource including pod, service, deployments, replication controller etc. Resources can be defined as YAML or JSON.  Here is the syntax to create a YAML specification.

**AKMS** => Resource Configs Specs

```
apiVersion: v1
kind:
metadata:
spec:
```

Spec Schema: https://kubernetes.io/docs/user-guide/pods/multi-container/





Lets now create the  Pod config by adding the kind and specs to above schema.

Filename: ghost-pod.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: ghost
  labels:
    app: ghost
    tier: front
    version: latest
spec:
  containers:
    - name: app
      image: ghost:0-alpine
      ports:
        - containerPort: 2368
          protocol: TCP

```


### Launching and operating a Pod

We could perform the following operations on a pod

  * get
  * describe
  * edit
  * logs
  * exec
  * port-forward
  * attach
  * cp


Syntax:

```
 kubectl apply -f FILE
```

To Launch pod using configs above,

```
kubectl apply -f ghost-pod.yaml

```

To view pods


```
kubectl get po -o wide

kubectl get pod ghost

```

To get detailed info

```
kubectl describe pods ghost
```

[Output:]
```
Name:         ghost
Namespace:    mogambo
Node:         kube-03/128.199.139.254
Start Time:   Wed, 07 Mar 2018 20:32:32 +0530
Labels:       app=ghost
              tier=front
              version=latest
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"ghost","tier":"front","version":"latest"},"name":"ghost","namespace":"mog...
Status:       Running
IP:           10.36.0.13
Containers:
  app:
    Container ID:   docker://a4d4cf23c7eff824751ef7bc4a9c14696e7fb5cca6f7e383dfc1afd8a9b27144
    Image:          ghost:0-alpine
    Image ID:       docker-pullable://ghost@sha256:04c99c1f5ed25a873c490dcd8f7028246b2c91eb004f0c94349b737fdcefd253
    Port:           2368/TCP
    State:          Running
      Started:      Wed, 07 Mar 2018 20:32:34 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-grqbf (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-grqbf:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-grqbf
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              11s   default-scheduler  Successfully assigned ghost to kube-03
  Normal  SuccessfulMountVolume  11s   kubelet, kube-03   MountVolume.SetUp succeeded for volume "default-token-grqbf"
  Normal  Pulled                 10s   kubelet, kube-03   Container image "ghost:0-alpine" already present on machine
  Normal  Created                10s   kubelet, kube-03   Created container
  Normal  Started                9s    kubelet, kube-03   Started container
```

Commands to operate the pod

```
kubectl logs ghost

kubectl exec -it ghost ps sh

kubectl exec -it ghost bash


```

delete
```
kubectl delete pod ghost

kubectl get pods
```

## Creating a pod for db with environment and data persistence  


Lets start writing spec for a database pod with the following configs,

  * image: 'mysql:8'
  * port:  3306 / tcp
  * labels:
    * tier: back
    * app: db
    * version: 8




File: db-pod.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    tier: back
    app: db
    version: 8
spec:
  containers:
    - name: mysql
      image: mysql:8
      ports:
        - containerPort: 3306
          protocol: TCP

```

Lets add the environment variables to the database to setup following vars

  * MYSQL_ROOT_PASSWORD
  * MYSQL_DATABASE
  * MYSQL_USER
  * MYSQL_PASSWORD


```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    tier: back
    app: db
    version: 8
spec:
  containers:
    - name: mysql
      image: mysql:8
      ports:
        - containerPort: 3306
          protocol: TCP
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: dfgk8923Jsk89JJSsdlf
        - name: MYSQL_DATABASE
          value: ghost
        - name: MYSQL_USER
          value: ghostapp
        - name: MYSQL_PASSWORD
          value: skljlj230kHshk02Ksc
```

Lets add configurations for data persistence.

  * create a **volumes** definition
  * attach volume to container using **VolumeMounts** property

Local volumes are of two types:
  * emptyDir
  * hostPath


```
apiVersion: v1
kind: Pod
metadata:
  name: db
  labels:
    tier: back
    app: db
    version: 8
spec:
  containers:
    - name: mysql
      image: mysql:8
      ports:
        - containerPort: 3306
          protocol: TCP
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: dfgk8923Jsk89JJSsdlf
        - name: MYSQL_DATABASE
          value: ghost
        - name: MYSQL_USER
          value: ghostapp
        - name: MYSQL_PASSWORD
          value: skljlj230kHshk02Ksc
      volumeMounts:
        - name: db-data
          mountPath: /var/lib/mysql
  volumes:
    - name: db-data
      hostPath:
        path: /var/lib/mysql
        type: DirectoryOrCreate

```


To create this pod,

```
kubectl apply -f db-pod.yaml

kubectl describe pod db

kubectl get events
```



## Creating Multi Container Pods

file: multi_container_pod.yml

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
kubectl apply -f multi_container_pod.yml
```

Check Status

```
root@kube-01:~# kubectl get pods
NAME      READY     STATUS              RESTARTS   AGE
nginx     0/2       ContainerCreating   0          7s
ghost      1/1       Running             0          3m
```

Checking logs, logging in

```
kubectl logs  web  -c loop
kubectl logs  web  -c nginx

kubectl exec -it web  sh  -c nginx
kubectl exec -it web  sh  -c loop

```

#### Reading List :

Node Selectors, Affinity
https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
