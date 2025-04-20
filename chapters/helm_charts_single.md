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
mkdir instavote
cd instavote/

helm create vote
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


## Adding Redis Service  

Lets add a second service by creating a new chart structure, where we will have our main chart for application and a subchart for vote and redis.


```
cd ~
mv instavote vote 
mkdir instavote 
mv vote instavote/

cd instavote 
helm create redis 
cd redis
```

Edit values.yaml to update the image and tag as

```
image:
  repository: redis
  pullPolicy: IfNotPresent
  tag: "alpine"
```

also update the service port as 

```
service:
  type: ClusterIP
  port: 6379
```

Now lets update the main chart to include the redis and vote subcharts

```
cd ..
mkdir main
cd main
```

Add `Chart.yaml` and `values.yaml` files as 

File :  `Chart.yaml`

```
apiVersion: v2
name: instavote
description: A Helm chart for Kubernetes

type: application

version: 1.0.0

appVersion: "1.0.0"

dependencies:
  - alias: vote
    name: vote
    version: "0.1.0"
    repository: file://../vote
  - alias: redis
    name: redis
    version: "0.1.0"
    repository: file://../redis
```

File : `values.dev.yaml`

```
vote:
  repicaCount: 3
  image:
    repository: "schoolofdevops/vote"
    tag: "v7"
  service:
    type: NodePort
    port: 80
    nodePort: 30300
  options:
    A: "AAA"
    B: "BBB"

redis:
  repicaCount: 1
  image:
    repository: "redis"
    tag: "alpine"
  service:
    type: ClusterIP
    port: 6379
```

Also rename the chart name for vote as 

File : `instavote/vote/Chart.yaml`
```
apiVersion: v2
name: vote
description: A Helm chart for Kubernetes
```

Now install the main chart with helm as 

```
helm dependency update
helm install instavote -n dev --values=values.dev.yaml . --dry-run
helm install instavote -n dev --values=values.dev.yaml .
```

validate with

```
helm list -A
kubectl get all -n dev
```



###  Exercise
Now that you have deployed vote and redis, go ahead and add the code to deploy  worker, db and result as well.
