# Steps to set up NFS based Persistent Volumes

## Set up NFS Common

On all kubernetes nodes,

```
sudo apt-get update
sudo apt-get install nfs-common
```

## Set up NFS Provisioner in kubernetes


Change into nfs provisioner installation dir

```
cd step5
cd storage/nfs
```


Deploy nfs-client provisioner.

```
kubectl apply -f clusterrole.yml

```

Create the Storage Class.

```
kubectl apply -f serviceaccount.yaml
kubectl apply -f clusterrole.yaml
kubectl apply -f clusterrolebinding.yaml

kubectl apply -f statefulset-sa.yaml

```


## Test

file: catalogue-db-pvc.yml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: catalogue-db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs

```

file: catalogue-db-pod.yml

```
apiVersion: v1
kind: Pod
metadata:
  name: catalogue-db
  labels:
    app: catalogue-db
    env: dev
    version: latest
    tier: "3"
spec:
  containers:
    - name: catalogue-db
      image: schoolofdevops/catalogue-db:latest
      imagePullPolicy: Always
      ports:
        - containerPort: 3306
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: dfgk8923Jsk89JJSsdlf
        - name: MYSQL_DATABASE
          value: socksdb
        - name: MYSQL_USER
          value: catalogue_user
        - name: MYSQL_PASSWORD
          value: default_password
      volumeMounts:
        - name: catalogue-db-vol
          mountPath: /var/lib/mysql

  volumes:
    - name: catalogue-db-vol
      persistentVolumeClaim:
        claimName: catalogue-db-pvc

```
