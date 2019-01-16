#### 1. Complete the following Service manifest for Pod with the label `app=webapp` and which is running on port `8888`.
```
kind: Service
apiVersion: v1
metadata:
  name: service
```

#### 2. From the following manifest, change the service type to Load Balancer. Try to access the service by accessing the load balancer URL. (This excercise assumes you already have vote application deployed in your environment.)
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30100
  type: NodePort
```

#### 3. How would you access your service on Port 80 from internet for the following?
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
```

#### 4. The following service is created in `instavote` namespace. Use kubectl command to change its namespace to `default`.
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  namespace: instavote
  labels:
    role: vote
spec:
  selector:
    role: vote
  ports:
    - port: 80
      targetPort: 80
```

#### 5. Write a Service manifest which listens to a RS with ports 80, 8080 and 8443. The Service should be created in the `dev` namespace. 
`Reference`: [Multi-port Service](https://kubernetes.io/docs/concepts/services-networking/service/#multi-port-services) 
