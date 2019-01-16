
file: storageclass.yml

```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io1
  zones: us-east-1b
  iopsPerGB: "10"
```


file: pvc.yml

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-data
  annotations:
    volume.beta.kubernetes.io/storage-class: standard
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

file: pvc-dep.yml

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  template:
    metadata:
      labels:
        application: redis
        version: 3.2.5
    spec:
      containers:
      - name: redis
        image: redis:3.2.5
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
        - name: redis-data
          persistentVolumeClaim:
            claimName: redis-data
```
