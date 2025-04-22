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
cd vote/
```


Edit `values.yaml`  to  update replicaCount, image and tag as

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
helm install -n dev vote  . --dry-run
helm install -n dev vote .
```

Validate with

```
helm list -A
kubectl get all
```

## Setting up Multi Environemnt Deployment with HELM 

Copy over the values.yaml file to values.staging.yaml

```
cp values.yaml values.staging.yaml
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
helm install -n staging vote --values=values.staging.yaml . --dry-run
helm install -n staging vote --values=values.staging.yaml . 
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



File:  `values.yaml`
```
replicaCount: 2

options:
  A: MacOS
  B: Windows

```

now upgrade the release as

```
helm upgrade -n dev vote .
```

validate 

```
helm list -A
kubectl get all -n dev
```

Check the application to see the default values being visible.

You could override the values from the command line as

```
helm upgrade -n dev vote --set options.A=Green --set options.B=Yellow .
```

check the web app now

try one more time, this time with a description

```
helm upgrade -n dev vote --set options.A=Orange --set options.B=Blue . --description "Overriding values to Orage and Blue form CLI"
```

you should see the values change every time.

Check the release now

```
helm list -A

```

[sample output]
```
NAME  	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
vote  	dev        	4       	2025-04-20 03:42:44.279388435 +0000 UTC	deployed	vote-0.1.0   	1.16.0
vote  	staging    	1       	2025-04-20 03:34:03.00413779 +0000 UTC 	deployed	vote-0.1.0   	1.16.0
```

You could also examine the history of the release as

```
helm history -n dev vote
```

try rolling back a version

e.g.

```
helm rollback vote
```
you could also go back to a specific version using the revision number as  

```
helm rollback vote xx
```

where replace `xx` with the revision number you wish to roll back to.


## Adding Redis Service  

Lets add a second service by creating a new chart structure, where we will have our main chart for application and a subchart for vote and redis.


```
# switch back to instavote directory
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

Now lets create the main chart for instavote app to include the redis and vote as subcharts

```
# switch back to instavote directory
cd instavote 
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

At this time the code should look like this 

``` 
# instavote chart directory
.
├── main
│   ├── Chart.yaml
│   └── values.dev.yaml
├── redis
│   ├── Chart.yaml
│   ├── charts
│   ├── templates
│   │   ├── NOTES.txt
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   ├── hpa.yaml
│   │   ├── ingress.yaml
│   │   ├── service.yaml
│   │   ├── serviceaccount.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
└── vote
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   ├── serviceaccount.yaml
    │   └── tests
    │       └── test-connection.yaml
    ├── values.staging.yaml
    └── values.yaml
```


Install/Update depeendencies with 

```
helm dependency update
```

```
ls
ls -l charts/
```

Clean up earlier helm installations if any 

```
helm list -A 
helm uninstall vote -n dev
helm uninstall vote -n staging
helm list -A
```

Now install the main chart with helm as 

```
helm install instavote -n dev --values=values.dev.yaml . --dry-run
helm install instavote -n dev --values=values.dev.yaml .
```

validate with

```
helm list -A
kubectl get all -n dev
```



###  Exercises

  * You will notice that the redis pods are being restarted every few seconds. Can you figure out why and fix it ? Apply the changes with helm.
  * Now that you have deployed vote and redis, go ahead and add the code to deploy  worker, db and result as well.

### Cleaning up 

To clean up all the resources, run the following command

```
helm uninstall instavote -n dev
```