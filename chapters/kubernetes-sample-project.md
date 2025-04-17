# Mini Project: Deploying Multi Tier Application Stack

In this project , you would write definitions for deploying the vote application stack with all components/tiers which include,

  * vote
  * redis
  * worker
  * db
  * result

## Project Description

  * Apply the deployment and service code for the applications marked as ready
  * Complete the code for  deployments and services marked as TODO
  * Apply the definitions that you have completed
  * Validate the application workflow is operational by loading vote and result applications from the browser


Following table depicts the state of readiness of the above services.

| App     | Deployment     | Service |
| :------------- | :------------- | :------------- |
| vote       | ready       | ready       |
| redis       | ready       | ready       |
| worker       | TODO       | n/a       |
| db       | ready       | ready       |
| result       | TODO       | TODO       |


### Apply existing code

Switch to the project directory and create  the namespace for the project as,

```
cd k8s-code/projects/instavote/dev/

kubectl config set-context --current --namespace=instavote
```

validate 

```
kubectl get ns 
kubectl config get-contexts 
```

If you have not created the deployment for vote app yet, apply it as, 

```
kubectl apply -f https://gist.githubusercontent.com/initcron/53d53d3d5bf7584e4da69489fbf15209/raw/0245fabc4dcf80a238860f7b315d0005eca4e61a/vote-deploy.yaml
```

If you have created the deployments and services for vote, redis and deployment for db, just create the db service as, 

```
kubectl apply -f db-svc.yaml
```


If you have not created anything yet, create the following objects as,

```

kubectl apply -f vote-svc.yaml
kubectl apply -f redis-deploy.yaml -f redis-svc.yaml
kubectl apply -f db-deploy.yaml -f db-svc.yaml
```

validate

```
kubectl get all
```
Where you should see,

  * replicaset and service for vote app created
  * deployments and services for redis and db created

If you see the above objects, proceed with the next task.

### Completing Code for worker and result apps

You would find the files available in the same directory as above i.e. *k8s-code/projects/instavote/dev/* with either partial or blank code. Your job is to complete the deployment and service yaml specs and apply those. While writing the specs, you could refer to the following specification.  

  * worker
    * image: schoolofdevops/worker:latest
  * results
    * image: schoolofdevops/vote-result
    * application port: 80
    * service type: NodePort
    * nodePort : 30100



#### To Validate:

```
kubectl get all
```

The above command should show,
  * five deployments and four services created
  * services for vote and result app should have been exposed with NodePort

Find out the NodePort for vote and service apps and load those from your browser.


![Front-End.\label{fig:captioned_image}](images/vote-rc.png)

This is how the vote application should look like.


You could also load the Result page on the `30100` port of your Node IP/Address as, 

![Result Page.\label{fig:captioned_image}](images/Result.png)

Above is how the result app should look like.

Final validation is, if you submit the vote, you should see the results changing accordingly. You would see this behavior only if all deployments and services are created and configured properly.
