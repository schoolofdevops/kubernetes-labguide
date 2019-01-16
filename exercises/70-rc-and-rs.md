#### 1. The following replication controller manifest has some bugs in it. Fix them.
```
apiVersion: v1/beta1
kind: ReplicationController
metadata:
  name: loop
spec:
  replicas: 3
  selecter:
    app: loop
  templates:
    metadata:
      name: loop
      labels:
      app: loop
    specs:
      container:
      - name: loop
          image: schoolofdevops/loop
```

`Reference`: [Replication Controller](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/)

#### 2. Create Pods using following manifests. Each Pod has different values for the Label "platform". You should create a ReplicaSet with 4 replicas, which also selects both Pods based on "platform" Label. **Tip: You may have to use matchExpressions attribute in your ReplicaSet.**
```
file: sync-aws-po.yml
apiVersion: v1
kind: Pod
metadata:
  name: sync-aws
  labels:
    version: 2
    platform: aws
spec:
  containers:
    - name: sync
      image: schoolofdevops/sync:v2

file: sync-gcp-po.yml
apiVersion: v1
kind: Pod
metadata:
  name: sync-gcp
  labels:
    version: 2
    platforms: gcp
spec:
  containers:
    - name: sync
      image: schoolofdevops/sync:v2
```

#### 3. Create a Namespace with the name test and prod. 
        - 1. For test create it using kubectl cli
        - 2. For prod, create from a namespace manifest(yaml) file.

#### 4. The following manifest is working as intended. Try to debug.
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: web
  labels:
    role: role
spec:
  replicas: 3
  selector:
    app: robotshop
  template:
    metadata:
      name: robotshop
      labels:
        app: robotshop
  containers:
    - name: web
      image: robotshop/rs-web:latest
      ports:
        - containerPort: 8080
          protocol: TCP
```

#### 5. How do you get the logs from all pods of a Replication Controller?
`Reference`: [logs from all pods in a RC](https://stackoverflow.com/questions/33069736/how-do-i-get-logs-from-all-pods-of-a-kubernetes-replication-controller)

#### 6. The Pods from a Replication Controller has stuck in `Terminating` status and doesn't actually get deleted. How do you delete these Pods.?
`Reference`: [Delete Pods forcefully](https://stackoverflow.com/questions/35453792/pods-stuck-at-terminating-status)

#### 7. How do you force repulling an image with the same tag?
```
apiVersion: v1
kind: Pod
metadata:
  name: kuard
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:1
      name: kuard
      ports:
        - containerPort: 8080
          name: http
          protocol: TCP
```
`Reference`: [Force image pull](https://stackoverflow.com/questions/33112789/how-do-i-force-kubernetes-to-re-pull-an-image)

#### 8. When I try to apply the following Pod manifest, I am getting `image <user/image>:latest not found`. How would you fix it? 
     `Tip`: This image is in a private registry.
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: web
  labels:
    role: role
spec:
  replicas: 3
  selector:
    app: robotshop
  template:
    metadata:
      name: robotshop
      labels:
        app: robotshop
  containers:
    - name: web
      image: my-private-registry/robotshop/rs-web:latest
      ports:
        - containerPort: 8080
          protocol: TCP
```
`Reference`: [Pulling images from private registry](https://stackoverflow.com/questions/32726923/pulling-images-from-private-registry-in-kubernetes/32972366#32972366)

#### 9. Launch the following Replication Controller in Prod namespace and use kubectl to scale the replicas from 3 to 6.
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: web
  labels:
    role: role
spec:
  replicas: 3
  selector:
    app: robotshop
  template:
    metadata:
      name: robotshop
      labels:
        app: robotshop
    spec:
      containers:
        - name: web
          image: robotshop/rs-web:latest
          ports:
            - containerPort: 8080
              protocol: TCP
```

#### 10. Pod that I've scheduled is in waiting state. How would I debug this issue?
`Reference`: [My pod stays waiting](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/#my-pod-stays-waiting)

