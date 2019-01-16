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


Create a deployment with following specs,

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
