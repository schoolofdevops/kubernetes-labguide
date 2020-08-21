# LAB K102: Kubernetes Quick Dive

In this lab you are going to deploy the **instavote** application stack [as described here](https://github.com/schoolofdevops/example-voting-app) in a kubernetes environment using **kubectl** commands. Later, you would learn how to do the same by writing declarive *yaml* syntax.  

Purpose of this lab is to quickly get your app up and running and demonstrate kubernetes key features such as scheduling, high availability, scalability, load balancing, service discovery etc.



### Deploying app with kubernetes

Launch vote application with kubernetes. 

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


kubectl create service nodeport vote --tcp=80:80 --node-port=30300

kubectl get service

```


Connect to the app,  refresh the page to see it load balancing.  Also try to vote and observe what happens.  


### Roll Out a New Version


```
kubectl scale deployment vote --replicas=12

kubectl set image deployment vote vote=schoolofdevops/vote:v2

```


watch the rolling update  in action

```
kubectl rollout status deploy/vote
```



#### Cleaning up

Once you are done observing, you could delete it with the following commands,

```

kubectl delete deploy vote

kubectl delete service vote
```

### Summary

When you deploy an application in kubernetes, you submit it to the api server/ cluster manager. Kubernetes automatically schedules it on a cluster, networks the pods, provides service discovery. In addition as you observed, your application is scalable, high available and is already running behind a  load balancer.
