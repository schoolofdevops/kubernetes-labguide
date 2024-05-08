# Mini Project: Deploying Multi Tier Application Stack

In this project , you would write definitions for deploying the [instavote application](https://github.com/schoolofdevops/example-voting-app) stack with all components/tiers which include,

  * vote
  * redis
  * worker
  * db
  * result

## Project Specs

  * Apply the deployment and service code for the applications marked as ready
  * Complete the code for  deployments and services marked as TODO
  * Apply the definitions that you have completed
  * Validate the application workflow is operational by loading vote and result applications from the browser
  * Set up ingress controller and add ingress rules so that you are able to access the apps using domain names e.g. vote.example.com and result.example.com


Following table depicts the state of readiness of the above services.

| App     | Deployment     | Service |
| :------------- | :------------- | :------------- |
| vote       | TODO       | ready       |
| redis       | ready       | ready       |
| worker       | TODO       | n/a       |
| db       | ready       | ready       |
| result       | TODO       | TODO       |


### Phase I - Apply existing code

Create a namespace and switch to it

```
kubectl create namespace instavote
kubectl config set-context --current --namespace=instavote
kubectl config get-contexts
```


Apply the existing manifests
```

cd k8s-code/projects/instavote/dev/

kubectl apply  -f vote-svc.yaml
kubectl apply -f redis-deploy.yaml -f redis-svc.yaml
kubectl apply -f db-deploy.yaml -f db-svc.yaml
```

validate

```
kubectl get all
```
Where you should see,

  * deplyoment and services for redis and db created
  * nodeport service for vote app

If you see the above objects, proceed with the next task.

### Phase II - Create Deployments and Services for Remaining Services  

You may find the files available in the same directory as above i.e. *k8s-code/projects/instavote/dev/* with either partial or blank code. Your job is to complete the deployment and service yaml specs and apply those. While writing the specs, you could refer to the following specification.  

  * vote
    * image: schoolofdevops/vote:v1
    * application port: 80
    * service type: NodePort
    * nodePort : 30000
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


### Phase III - Set up host based traffic routing with Ingress  

Use the documentation here [Nginx Ingress Controller with KIND](https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx) to set up ingress controller.


Launch Nginx Ingress controller as :


```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Check the pod for Nginx Ingress, if its running

```
kubectl get pods -n ingress-nginx
```

You may see the pod in pending state. Check why its pending and fix it. Thats part of your project work.

Once you have the ingress controller working, port [this ingress rule](https://kubernetes-tutorial.schoolofdevops.com/ingress/#set-up-named-based-routing-for-vote-app) for **vote** app and also add one for **result** app to make it work with nginx ingress controller that you have.

Also add the `host` file configuration as per the same document and validate you are able to use http://vote.example.com/ and http://result.example.com/ to access your services respectively.
