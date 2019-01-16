### Canary  Releases

```
cd step4
cd projets/mogambo/dev
mkdir canary
```


File: canary/frontend-canary-deploy.yml


```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: frontend-canary
  namespace: mogambo
spec:
  replicas: 3
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
        release: canary
    spec:
      containers:
        - name: frontend
          image: schoolofdevops/frontend:v2.0
          ports:
            - containerPort: 8079
              protocol: TCP

```

deploy the canary version

```

kubectl apply -f canary/frontend-canary-deploy.yml

```

### Blue/Green  Releases


```
cd step4
cd projets/mogambo/dev
mkdir blue-green

```

file: frontend-blue-deploy.yml

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: frontend-blue
  namespace: mogambo
spec:
  replicas: 5
  strategy:
    type: Recreate
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
        release: bluegreen
        code: blue
    spec:
      containers:
        - name: frontend
          image: schoolofdevops/frontend:v3.0
          ports:
            - containerPort: 8079
              protocol: TCP

```

file: step5/projects/mogambo/dev/blue-green/frontend-green-deploy.yml
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: frontend-green
  namespace: mogambo
spec:
  replicas: 5
  strategy:
    type: Recreate
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
        release: bluegreen
        code: green
    spec:
      containers:
        - name: frontend
          image: schoolofdevops/frontend:v1.0
          ports:
            - containerPort: 8079
              protocol: TCP

```

file: blue-green/frontend-test-svc.yml

```
apiVersion: v1
kind: Service
metadata:
  name: frontend-test
  namespace: mogambo
spec:
  selector:
    app: frontend
    env: dev
    release: bluegreen
    code: blue
  ports:
    - port: 8079
      protocol: TCP
      targetPort: 8079
  type: NodePort

```


file: frontend-svc.yml
```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: mogambo
spec:
  selector:
    app: frontend
    env: dev
    release: bluegreen
    code: blue
  ports:
    - port: 8079
      protocol: TCP
      targetPort: 8079
  type: NodePort

```

### Recreate

When the **Recreate** deployment strategy is used,
  * The old pods will be deleted
  * Then the new pods will be created.

This will create some downtime in our stack. Hence, this strategy is only recommended for development use cases.

Let us change the deployment strategy to **recreate** and image tag to *latest*.

file: catalogue-deploy.yml

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: catalogue
  namespace: mogambo
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
