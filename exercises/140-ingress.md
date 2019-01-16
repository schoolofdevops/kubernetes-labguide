#### 1. Create a nginx ingress controller by following the documentation.
`Reference`: [Nginx ingress](https://kubernetes.github.io/ingress-nginx/deploy/#generic-deployment)

#### 2. Use the nginx ingress to with the following properties for the deployment give below. Create the missing components to expose this deployment.
```
[ingress-details]
ingress controller: nginx
domain name: mogambo.com
path: /
service: frontend
service port: 80
```
```
[frontend-deployment]
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: front-end
    spec:
      containers:
      - image: schoolofdevops/frontend:latest
        imagePullPolicy: Always
        name: front-end
        ports:
        - containerPort: 8079
          protocol: TCP
```

#### 3. Create a basic authentication for the frontend deployment with the following parameters with nginx ingress controller.
```
username: serviceadmin
password: sup3rs3cr3t
```
