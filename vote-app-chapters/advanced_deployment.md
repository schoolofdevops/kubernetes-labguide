## Deployments

Deployments provide reliability and scalability to our application stack. Deployment makes sure that desired number of pods, which is specified declaratively in the deployment file (ex. catalogue-deploy.yml), are always up and running. If a pod fails to run, deployment will remove that pod and replace it with a new one.

### Deployments vs Replication Controllers

Replication Controllers are the older version of Replica Sets. In some ways RC are superior than RS and in some cases it is inferior. But, how does replication work when compared to deployments?

Replication Controllers are,
  * Older method of deploying application
  * Updates to the application is done with **kubectl rolling-update command**
  * Does not support roll back feature like in Deployments.
  * It has no backends, interacts with pods directly.

Deployments are,
  * The newer method of deploying application.
  * Provides both **rolling update and rollback**
  * Replica Sets are the backend of Deployments.

### Deployment API Specs

A typical deployment file looks like the following,

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: <name-of-deployment>
  label:
    <deployment-labels>
spec:
  replicas: <number-of-replicas>
  template:
    metadata:
      label:
        <pod-labels>
    spec:
      containers:
        <pod-spec>
```

### Defining Deployment Strategies

We have two types of deployment strategies in Kubernetes, which are,
  * Rolling Update
  * Recreate.

### Rolling Update

Let us look at our catalogue deployment file,

`File: catalogue-deploy.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: catalogue
  namespace: instavote
  labels:
    app: catalogue
    env: dev
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: catalogue
        env: dev
    spec:
      tolerations:
        - key: "dedicate"
          operator: "Equal"
          value: "catalogue"
          effect: "NoExecute"
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

We have not defined any deployment strategies here. Let us apply this deployment file.

```
kubectl apply -f catalogue-deploy.yml

[output]
deployment "catalogue" created
```

```
kubectl get deployment catalogue --export -o yaml

[...]
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```

As we can see, deployment strategy of **rolling update is applied by default**. When we use rolling update,
  * We can avoid down time since old version of application is still receiving traffic.
  * Only a few pod is updated at a time.

Along with the type RollingUpdate, we see two fields.
  * maxSurge
  * maxUnavailable

#### MaxSurge

The maximum batch size of the available pods will get updated at once. For example, if we have 4 pods running with max surge value of 25%, then one pod will be updated at once.

#### MaxUnavailable

As the title implies, how many pods can be unavailable during the update process.

Rolling updates are the preferred way to update an application. It is slower than **recreate** type, but it provides us with zero downtime deployments.
When you create a deployment, a **replica set** for that deployment is also created.

```
kubectl get rs --selector app=catalogue

[output]
NAME                   DESIRED   CURRENT   READY     AGE
catalogue-6bc67654d5   1         1         1         17m
```

When you do an update, a new replica set is created. But the old replica set is kept for roll back purposes.

Let us change the deployment to use a new version.

`File: catalogue-deploy.yml`

```
[...]
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:v1
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

Apply the changes

```
kubectl apply -f catalogue-deploy.yml

kubectl get pods --selector app=catalogue

[output]
NAME                         READY     STATUS              RESTARTS   AGE
catalogue-6bc67654d5-tbn9h   0/1       ContainerCreating   0          7s
catalogue-6c4f7b49d8-bdtgh   1/1       Running             0          8m

kubectl get rs --selector app=catalogue

[output]
NAME                   DESIRED   CURRENT   READY     AGE
catalogue-6bc67654d5   0         0         0         21m
catalogue-6c4f7b49d8   1         1         1         1m

kubectl rollout status deployment catalogue

[output]
deployment "catalogue" successfully rolled

kubectl rollout history deployment catalogue

[output]
deployments "catalogue"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

As we can see, we have two revisions of our deployment. This revisions are used for roll backs about which we will learn later in this chapter.

### Recreate

When the **Recreate** deployment strategy is used,
  * The old pods will be deleted
  * Then the new pods will be created.

This will create some downtime in our stack. Hence, this strategy is only recommended for development use cases.

Let us change the deployment strategy to **recreate** and image tag to *latest*.

`File: catalogue-deploy.yml`

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: catalogue
  namespace: instavote
  labels:
    app: catalogue
    env: dev
spec:
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: catalogue
        env: dev
    spec:
      tolerations:
        - key: "dedicate"
          operator: "Equal"
          value: "catalogue"
          effect: "NoExecute"
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 80

```

### Rollbacks

As we discussed earlier, the major advantage of using deployments over replication controller is its roll back feature. Let us see how it is done.

First check the image being used by the deployment.

```
kubectl describe deployment catalogue

[...]
  Containers:
   catalogue:
    Image:        schoolofdevops/catalogue:v1
    Port:         80/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
```

The current deployment has the image with the tag **v1**.

```
kubectl rollout history deployment catalogue

[output]
deployments "catalogue"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

We have two revisions. Revision 1 is the first ever deployment we have done while revision 2 is the one in which we changed the image tag to v1. Let us rollback to revision 1.

Let us rollback to revision 1 which has the image tag of **none**.

```
kubectl rollout undo deployment catalogue --to-revision=1

[output]
deployment "catalogue"
```

```
kubectl describe deployment catalogue

[...]
  Containers:
   catalogue:
    Image:        schoolofdevops/catalogue
    Port:         80/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
```

### Pause/Unpause

When you are in the middle of a new update for your application and you found out that the application is behaving as intended. In those situations,
  1. we can pause the update,
  2. fix the issue,
  3. resume the update.

Let us change the image tag to V2 in pod spec.

`File: catalogue-deploy.yml`

```
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:V2
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

Apply the changes.

```
kubectl apply -f catalogue-deploy.yml

kubectl get pods

[Output]
NAME                         READY     STATUS         RESTARTS   AGE
catalogue-6c4f7b49d8-g5dgc   1/1       Running        0          16m
catalogue-765554cc7-xsbhs    0/1       ErrImagePull   0          9s
```

Our deployment is failing. From some debugging, we can conclude that we are using a wrong image tag.

Now pause the update

```
kubectl rollout pause
```

Set the deployment to use **v2** version of the image.

```
kubectl set image deployment catalogue catalogue=schoolofdevops/catalogue:v2

[output]
deployment "catalogue" image updated
```

Now resume the update

```
kubectl rollout resume deployment catalogue
kubectl rollout status deployment catalogue

[Ouput]
deployment "catalogue" successfully rolled out
```

```
kubectl get pods

[Output]
NAME                         READY     STATUS    RESTARTS   AGE
catalogue-6875c8df8f-k4hls   1/1       Running   0          1m
```

When we do this, we skip the need of creating a new rolling update altogether.

### Additional Release Strategies

Along with rolling update and recreate strategies, we can also do,

  * Canary releases
  * Blue/Green deployments.

### Canary Releases

### Blue/Green Deployments


---
1. Create two more catalogue images with different products.
2. Recreate type gives an error. Need to be fixed.
3. Add Use-cases for recreate strategy type.
4. Add additional details of why we skip creating a replica set / rolling update in pause/unpause section.
