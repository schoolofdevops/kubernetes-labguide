# XER001 - Pods



## 1

Create a Pod manifest which uses "ghost" image and open port 2368.

```
apiVersion: v1
kind: Pod
metadata:
  name: ghost
spec:
  containers:
  - image: xxx
    name: ghost
    ports:
    - containerPort: xxx
      hostPort: xxx
```

Get the name of the Node in which the Pod is scheduled by running,
```
kubectl describe pod <POD_NAME>

[output]
Name:           <POD_NAME>
Namespace:      default
Node:           <NODE_NAME>/<NODE_IP>
Start Time:     Wed, xx May 201x 15:59:29 +0530
```

Try to access the application on the host's port 2368.

```
curl <NODE_IP>:2368
```

`Reference`: [Ghost Docker image](https://hub.docker.com/_/ghost/)

## 2

Create a Pod with ubuntu:trusty image and a command to echo “YOUR_NAME” which overrides the default CMD/ENTRYPOINT of the image.
`Reference`: [Define command argument in a Pod](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)

## 3

Apply the following Pod manifest and read the error. Fix it by editing it.
```
apiVersion: v1apps/beta1
kind: Pod
metadata:
  name: mogambo-frontend
  label:
    role: frontend
spec:
  containers:
    - name: frontend
      image: schoolofdevops/frontend:orange
      ports:
        - containerName: web
          Port: 8079
          protocol: TCP
```
`Reference`: [Debugging a unscheduled Pod](https://stackoverflow.com/questions/37302776/kubectl-get-pods-kubectl-get-pods-status-imagepullbackoff)

## 4

A Pod with the following pod always crashes with CrashLoopBackOff error. How would you fix it?

```
      image: schoolofdevops/nginx:break
      ports: 80
```
`Reference`: [Debugging a crashed Pod](https://sysdig.com/blog/debug-kubernetes-crashloopbackoff/)

## 5

You are running a Pod with hostPort option. Your Pod status stays “pending”. What could be the issue for the Pod not being scheduled?
`Reference`: [Debugging a unscheduled Pod](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/#my-pod-stays-waiting)

## 6

The given manifest for multi-container pod is not working as intended. It does not sync the content between containers like expected. What could be the issue? Find the issue just by reading the manifest.

```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    tier: front
    app: nginx
    role: ui
spec:
  containers:
    - name: nginx
      image: nginx:stable-alpine
      ports:
        - containerPort: 80
          protocol: TCP
      volumeMounts:
        - name: data
          mountPath: /var/www/html-sample-app

    - name: sync
      image: schoolofdevops/sync:v2
      volumeMounts:
        - name: datanew
          mountPath: /var/www/app

  volumes:
    - name: data
      emptyDir: {}
```

## 7

For the above given manifest, the following command is not working. What could be the issue?
```
kubeclt exec -it web -sh -c synch
```

## 8

Try to apply the following manifest. If fails, try to debug.
```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app:
    role: role
spec:
  containers:
    - name: web
      image: robotshop/rs-web:latest
      ports:
        - containerPort: 8080
          protocol: TCP
```

## 9

Fix the following manifest. Don't apply it. Just fix it by reading.
```
apiVersion: v1
kind: pod
metadata:
  name: web
labels:
  role: role
spec:
  containers:
    - name: web
      image: robotshop/rs-web:latest
      ports:
        - containerport: 8080
          protocol: TCP
```

## 10

Mount `/var/www/html` from Pod using the follwing manifest. Fill the missing fields.
```
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    role: role
spec:
  containers:
    - name: web
      image: robotshop/rs-web:latest
      ports:
        - containerPort: 8080
          protocol: TCP
  volumes:
    - name: roboshop-storage
      emptyDir: {}
```

## 11

Write a Pod manifest with the image nginx which has a volume that mounts /etc/nginx/ directory. Use "hostPath" volume type.
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts:
      xxx
```
`Reference`: [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
