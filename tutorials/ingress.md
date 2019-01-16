


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-dep
  labels:
    app: nginx-ingress
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      containers:
      - image: nginxdemos/nginx-ingress:0.9.0
        name: nginx-ingress
        ports:
        - containerPort: 80
          hostPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress-svc
  labels:
    app: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30000
  selector:
    app: nginx-ingress

```
