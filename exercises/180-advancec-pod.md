#### 1. In your Kubernetes cluster, Label one node with the label `frontend` and schedule the frontend deployment in the frontend node.
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: frontend
        role: ui
    spec:
      containers:
      - image: schoolofdevops/frontend:latest
        imagePullPolicy: Always
        name: front-end
        ports:
        - containerPort: 8079
          protocol: TCP
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
    [...]
```

#### 2. Schedule the catalogue pods in the same node as frontend pod.
```
kind: Deployment
metadata:
  name: catalogue
  labels:
    app: catalogue
    env: dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: catalogue
  template:
    metadata:
      labels:
        app: catalogue
        env: dev
    spec:
      containers:
        - name: catalogue
          image: schoolofdevops/catalogue:v2
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

#### 3. Make one of the node(ex. node 2) as unschedulable. Only carts pods should be schedulable on that node.
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: carts
  labels:
    app: carts
    env: dev
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: carts
        env: dev
    spec:
      containers:
        - name: carts
          image: schoolofdevops/carts
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```
