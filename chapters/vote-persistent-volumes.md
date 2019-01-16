# Steps to set up NFS based Persistent Volumes

## Set up NFS Common

On all kubernetes nodes, if you have not already installed nfs, use the following command to do so

```
sudo apt-get update
sudo apt-get install nfs-common
```

[Skip this step if you are using a vagrant setup recommended as part of this course. ]

## Set up NFS Provisioner in kubernetes


Change into nfs provisioner installation dir

```
cd k8s-code/storage
```


Deploy nfs-client provisioner.

```
kubectl apply -f nfs

```

This will create all the objects required to setup a nfs provisioner.

### Creating a Persistent Volume Claim

switch to project directory

```
cd k8s-code/projects/instavote/dev/
```


file: db-pvc.yaml

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 2Gi
  storageClassName: nfs

```


And lets create the Persistent Volume Claim

```
kubectl get pvc,storageclass

kubectl logs -f nfs-provisioner-0

kubectl apply -f db-pvc.yaml

kubectl get pvc,storageclass

kubectl describe pvc db-pvc
```


file: db-deploy.yaml

```
...
spec:
   containers:
   - image: postgres:9.4
     imagePullPolicy: Always
     name: db
     ports:
     - containerPort: 5432
       protocol: TCP
     #mount db-vol to postgres data path
     volumeMounts:
     - name: db-vol
       mountPath: /var/lib/postgresql/data
   #create a volume with pvc
   volumes:
   - name: db-vol
     persistentVolumeClaim:
       claimName: db-pvc
```

Observe which host **db** pod is running on
```
kubectl get pod -o wide --selector='role=db'

```
And apply this code as

```
kubectl apply -f db-deploy.yaml

kubectl get pod -o wide --selector='role=db'

```

  * Observe the volume and its content created on the nfs server
  * Observe which host the pod for *db* was created this time. Analyse the behavior of a deployment controller. 
