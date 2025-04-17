# Building your own Helm Charts

If you have not installed helm yet, do so by using

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 |  bash 

helm

```
 
## Part I - Generating a Chart with One Service

Create a namespace and switch to it

```
kubectl create namespace dev
kubectl config set-context --current --namespace=dev
kubectl config get-contexts
```

Generate a helm chart scaffold and change into the directory created to create a copy of values.yaml

```
cd ~
helm create instavote
cd instavote/
cp values.yaml values.dev.yaml 
```


Edit `values.dev.yaml`  to  update replicaCount, image and tag as

```

replicaCount: 2
image: 
  repository: schoolofdevops/vote 
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion. 
  tag: "v5" 
```

Update service type to Node Port
```
service:
  type: NodePort
  port: 80
  nodePort: 30300
```

Also update the `service.yaml` template with the additional property for `nodePort` defined as ,

File : `service.yaml`
```
apiVersion: v1
kind: Service
metadata:
  name: vote
  labels:
    {{- include "instavote.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
      nodePort: {{ .Values.service.nodePort }}
  selector:
    {{- include "instavote.selectorLabels" . | nindent 4 }}
```

Install this chart with helm to deploy the `vote` service as:

```
helm install instavote -n dev --values=values.dev.yaml  . --dry-run
helm install instavote -n dev --values=values.dev.yaml  .
```

Validate with

```
helm list -A
kubectl get all
```

## Setting up Multi Environemnt Deployment with HELM 

Copy over the values.dev.yaml file to values.staging.yaml

```
cp values.dev.yaml values.staging.yaml
```

Update the properties in values.staging.yaml as, 

```

replicaCount: 4 
image: 
  repository: schoolofdevops/vote 
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion. 
  tag: "v4" 

```


Update service type to Node Port
```
service:
  type: NodePort
  port: 80
  nodePort: 30400
```


Create a namespace for staging as 

```
kubectl create namespace staging
```

Install this chart with helm to deploy the `vote` service as:

```
helm install instavote -n staging --values=values.staging.yaml  . --dry-run
helm install instavote -n staging --values=values.staging.yaml  .
```

Validate with

```
helm list -A
kubectl get all -n staging 
```


## Part III : Overriding Values, Rollbacks

Lets learn how to override values from the command line. To do so, lets add one property

File : `templates/deployment.yaml`

```
containers:
  - name: vote
    env:
      - name: OPTION_A
        value: {{ .Values.options.A }}
      - name: OPTION_B
        value: {{ .Values.options.B }}
```

This will read from values file and set those as environmnt variables. Lets set the default values.



File:  `values.dev.yaml`
```
replicaCount: 2

options:
  A: MacOS
  B: Windows

```

now upgrade the release as

```
helm upgrade  instavote -n dev --values values.dev.yaml .
```

Check the application to see the default values being visible.

You could override the values from the command line as

```
helm upgrade instavote --values values.dev.yaml --set options.A=Green --set options.B=Yellow .```

check the web app now

try one more time

```
helm upgrade instavote --values values.dev.yaml --set options.A=Orange --set options.B=Blue .
```

you should see the values change every time.

Check the release now

```
helm list -A

```

[sample output]
```
NAME     	NAMESPACE 	REVISION	UPDATED                                	STATUS  	CHART          	APP VERSION
instavote 	dev      	   7       	2024-05-14 03:31:34.717912894 +0000 UTC	deployed	instavote-0.1.0	1.16.0
```

try rolling back a version

e.g.

```
helm rollback instavote
```
you could also go back to a specific version using the revision number as  

```
helm rollback instavote xx
```

where replace `xx` with the revision number you wish to roll back to.



###  Exercise
Now that you have deployed vote and redis, go ahead and add the code to deploy  worker, db and result as well.
