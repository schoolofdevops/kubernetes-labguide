## Creating a Deployment


A Deployment is a higher level abstraction which sits on top of replica sets and allows you to manage the way applications are deployed, rolled back at a controlled rate.

Deployment has mainly two responsibilities,

  * Provide Fault Tolerance: Maintain the number of replicas for a type of service/app. Schedule/delete pods to meet the desired count.
  * Update Strategy: Define a release strategy and update the pods accordingly.

Deployment spec (deployment.spec) contains the following,

    * replicaset specs
      * selectors  
      * replicas  
    * deployment spec
      * strategy
      * rollingUpdate
      * minReadySeconds
    * pod template
      * metadata, labels
      * container specs

Create project directory with dev env

```
cd step2
mkdir -p projects/mogambo/dev
cd projects/mogambo/
```


Create a namespace

file: mogambo-ns.yml

```
kind: Namespace
apiVersion: v1
metadata:
  name: mogambo

```

```
kubectl apply -f mogambo-ns.yml
kubectl get ns

kubectl config set-context $(kubectl config current-context) --namespace=mogambo

```


Create a deployment with following specs,

  * replicas: 2
  * pod template : frontend
    * labels:
      * tier: "1"
      * app: frontend
      * env: dev
      * version: v1.0
    * container: frontend
      * image: schoolofdevops/frontend:v1.0
      * port:  8079/TCP


File: frontend-deploy.yml

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: frontend
  namespace: mogambo
spec:
  replicas: 2
  template:
    metadata:
      name: frontend
      labels:
        tier: "1"
        app: frontend
        env: dev
        version: v1.0
    spec:
      containers:
        - name: frontend
          image: schoolofdevops/frontend:v1.0
          ports:
            - containerPort: 8079
              protocol: TCP
```




Lets  create the Deployment
```
kubectl apply -f frontend-deploy.yml
```

Now that the deployment is created. To validate,

```
kubectl get deployment
kubectl get rs
kubectl get pods --show-labels
```

Sample Output
```
kubectl get deployments
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend   2         2         2            1           3m
```


## Scaling a deployment  

To scale a deployment in Kubernetes:

```
kubectl scale deployment/frontend --replicas=5
```

Sample output:
```
kubectl scale deployment/frontend --replicas=5
deployment "frontend" scaled
```

## Adding a Deployment Strategy


deployment.spec

  * replicas: 8
  * strategy:
      type: RollingUpdate
  * rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  * minReadySeconds: 40
  * revisionHistoryLimit: 4
  * paused: false


File: frontend-deploy.yml


```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: frontend
  namespace: mogambo
spec:
  replicas: 8
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  minReadySeconds: 40
  revisionHistoryLimit: 4
  paused: false
  template:
    metadata:
      name: frontend
      labels:
        tier: "1"
        app: frontend
        env: dev
        version: v1.0
    spec:
      containers:
        - name: frontend
          image: schoolofdevops/frontend:v1.0
          ports:
            - containerPort: 8079
              protocol: TCP

```

Apply Changes and monitor the rollout

```
kubectl apply -f frontend-deploy.yml
kubectl rollout status deployment/frontend
```

## Rolling Back a Failed Update

Lets update the image to a tag which is non existent. We intentionally introduce this intentional error to fail fail the deployment.


```
...
containers:
  - name: frontend
    image: schoolofdevops/frontend:v2.0

```

Do a new rollout and monitor

```
kubectl apply -f frontend-deploy.yml
kubectl rollout status deployment/frontend
```


## Rolling Back a Failed Update

Lets update the image to a tag which is non existent. We intentionally introduce this intentional error to fail fail the deployment.


```
...
containers:
  - name: frontend
    image: schoolofdevops/frontend:xyz

```

Do a new rollout and monitor

```
kubectl apply -f frontend-deploy.yml
kubectl rollout status deployment/frontend
```

Also watch the pod status which might look like

```
frontend-645bb6fcd5-xsndp   1/1       Running            0          3m
frontend-b9bc5495f-8nbz5    0/1       ImagePullBackOff   0          54s
frontend-b9bc5495f-9x5gr    0/1       ImagePullBackOff   0          54s
frontend-b9bc5495f-pdx2l    0/1       ImagePullBackOff   0          55s
```

To get the revision history and details  
```
kubectl rollout history deployment/frontend
kubectl rollout history deployment/frontend --revision=x
[replace x with the latest revision]
```

[Sample Output]

```
kubectl rollout history deployment/frontend
deployments "frontend"
REVISION	CHANGE-CAUSE
1		kubectl scale deployment/frontend --replicas=5
3		<none>
6		<none>
7		<none>

root@kube-01:~# kubectl rollout history deployment/frontend --revision=7
deployments "frontend" with revision #7
Pod Template:
  Labels:	app=frontend
	pod-template-hash=656710519
	role=ui
	tier=front
  Containers:
   frontend:
    Image:	schoolofdevops/frontend:movi
    Port:	8079/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

To undo rollout,

```
kubectl rollout undo deployment/frontend
```

or

```
kubectl rollout undo deployment/frontend --to-revision=1
kubectl get rs
kubectl describe deployment frontend
```
