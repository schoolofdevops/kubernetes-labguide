#### 1. Create two differenct ConfigMap manifest with the follwing properties. If there are any bugs, fix them.
```
Name: ConfigMap-A
data 1:
  SALT: astdXy3Cmn5DMbcPqeYK
  STATE: present

Name: ConfigMap-B
data 2:
  API: https://107.178.217.202
  USER: admin  
```

#### 2. Complete the following Pod definition to use the above mentioned configMaps.
```
apiVersion: v1
kind: Pod
metadata:
  name: config-test-pod
spec:
  containers:
    - name: config-test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "echo $(SALT) $(STATE) $(API)" ]
      xxx:
      [...]
  restartPolicy: Never
```

#### 3. Convert the following data into secret.
```
name: mongo-secret
data:
  username: mongoadmin
  password: Cr0wnJ3w3!
  secret_key: AsMPgc3xTroPcv6vi$NO
```

#### 4. Mount the above created secret as a volume with the /etc/mongoCreds path in the following Pod manifest. 
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: secret-test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ping" "localhost" ]
```

#### 5. Create a Secret from literal values using kubectl.
```
  username: casterAdmin
  password: s3cr3tsauc3
```
