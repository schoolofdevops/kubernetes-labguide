# Steps to set up NFS based Persistent Volumes


### Creating a Persistent Volume Claim

switch to project directory

```
cd k8s-code/projects/instavote/dev/
```

Create the following file with the specs below

`file: db-pvc.yaml`

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


create the Persistent Volume Claim and validate

```
kubectl get pvc


kubectl apply -f db-pvc.yaml

kubectl get pvc,pv

```

Now, to use this PVC, db deployment needs to be updated with *volume* and *volumeMounts* configs as given in example below.


`file: db-deploy-pvc.yaml`

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

Apply *db-deploy-pcv.yaml*  as

```
kubectl apply -f db-deploy-pvc.yaml

kubectl get pod -o wide --selector='role=db'

kubectl get pvc,pv
```

  * Observe and note which host the pod for *db* is launched.
  * What state is it in ? why?
  * Has the persistentVolumeClaim been bound to a persistentVolume ? Why?




## Set up NFS Provisioner in kubernetes

Change into nfs provisioner installation dir

```
cd k8s-code/storage
```


Deploy nfs-client provisioner.

```
kubectl apply -f nfs

```

This will create all the objects required to setup a nfs provisioner. It would be launched with  Statefulsets. [Read the official documentation on Statefulsets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) to understand how its differnt than deployments.


```
kubectl get storageclass
kubectl get pods
kubectl logs -f nfs-provisioner-0

```

Now, observe the output of  the following commands,

```
kubectl get pvc,pv
kubectl get pods
```

  * Do you see pvc bound to pv ?
  * Do you see the pod for db running ?

Observe the dynamic provisioning, go to the host which is running nfs provisioner and look inside */srv* path to find the provisioned volume.

#### Summary

In this lab, you not only setup dynamic provisioning using NFS, but also learnt about statefulsets as well as rbac policies applied to the nfs provisioner.
