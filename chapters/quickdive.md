# LAB K102: Kubernetes Quick Dive

In this lab you are going to deploy the **instavote** application stack [as described here](https://github.com/schoolofdevops/example-voting-app) in a kubernetes environment using **kubectl** commands. Later, you would learn how to do the same by writing declarive *yaml* syntax.  

Purpose of this lab is to quickly get your app up and running and demonstrate kubernetes key features such as scheduling, high availability, scalability, load balancing, service discovery etc.



### Deploying app with kubernetes

Launch vote application with kubernetes. (simiar to docker run command)

```
kubectl  run vote --image=schoolofdevops/vote:v1
```

Since the above command is now deprecated, you could also use a alternate command such as

`Following is an alternate. run it only if you have not used kubectl run above`
```
kubectl create deployment vote --image=schoolofdevops/vote:v1
```

You could now validate that the instance of vote app is running by using the following commands,

```
kubectl get pods

kubectl get deployments
```


### Scalability


Scale the vote app to run 4 instances.

```
kubectl scale deployment vote --replicas=4
kubectl get deployments,pods
```


### High Availability

```
kubectl get pods
```

The above command will list pods. Try to delete a few pods and observe how it affects the availability of your application.

```
kubectl delete pods vote-xxxx vote-yyyy
kubectl get deploy,rs,pods

```


### Load Balancing

Publish the application (similar to using -P for port mapping)

```


kubectl expose deployment vote --type=NodePort --port 80

kubectl get svc

```


Connect to the app,  refresh the page to see it load balancing.  Also try to vote and observe what happens.  


### Deploying a new version


```
kubectl scale deployment vote --replicas=12

kubectl set image deployment vote vote=schoolofdevops/vote:v2

```


watch the rolling update  in action

```
watch kubectl get deploy,rs,pods
```

### Service Discovery

**Pre Test**: Try to submit a vote from the frontend vote app. Does that work ?

Now lets launch rest of the apps.


```
kubectl  create deployment  redis  --image=redis:alpine

kubectl expose deployment redis --port 6379

```

optionally, you could launch rest of the services.

```
kubectl  create deployment  worker --image=schoolofdevops/worker

kubectl  create deployment  db --image=postgres:9.4

kubectl expose deployment db --port 5432

kubectl create deployment  result --image=schoolofdevops/vote-result

kubectl expose deployment result --type=NodePort --port 80

```

**Post Tests**:

  * Try to submit a vote from the frontend vote app. Does that work ?  If yes, try to deduce how it could connect to the backend applications such as redis.
  * Access results ui application. When you submit vote, do the results change ?


#### Cleaing up

Once you are done observing, you could delete it with the following commands,

```

kubectl delete deploy db redis vote worker result

kubectl delete svc db redis result vote
```

### Summary

When you deploy an application in kubernetes, you submit it to the api server/ cluster manager. Kubernetes automatically schedules it on a cluster, networks the pods, provides service discovery. In addition as you observed, your application is scalable, high available and is already running behind a  load balancer.
