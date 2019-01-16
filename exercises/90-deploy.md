#### 1. Create an Nginx Deployment using kubectl with following properties.
```
  1. Name: nginx-frontend
  2. Labels: app=nginx, tier=frontend
  3. Image: nginx:latest
  3. Replicas: 3
  4. ImagePullPolicy: Always
```

#### 2. For the above mentioned Deployment, change the image version from `nginx:latest` to `nginx:1.12.0` using `kubectl set image` command

#### 3. For the above mentioned Deployment, undo the changes using `kubectl rollout` command and rollback to revision 1.

#### 4. Create a Deployment manifest with the following properties and apply it. If it fails, try to do rolling update the image version to `0.3.0`
```
  1. Name: web-app
  2. Labels: env=prod, tier=web
  3. Image: robotshop/rs-web:v1.0
  4. Port: 8080
  5. Restart Policy: Always
```

#### 5. For the following application, fix and fill in the missing details.
```
apiVersion: extensions/v1beta1
kind: deployment
metadata:
  labels:
  name: mysql
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: robotshop/rs-mysql-db:latest
        name: 
        ports:
        - Containerport: 3306
      restartPolicy: xxxx
```
