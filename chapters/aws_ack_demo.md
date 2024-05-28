# AWS Controller for Kubernetes (ACK ) Demo


From AWS Console,

  * create user with FullS3Access.  
  * Note down the Access/Secret keys.    
  * Add these credentials as kubernetes secret in a new namespace as  

```
kubectl create namespace ack-system
kubectl create secret generic aws-creds --from-literal=key=XXXX --from-literal=secret=YYYY --namespace ack-system
```


## Setup S3 Controller for ACK

You could find the ACK Conrtrollers from [ECR Public Gallery](https://gallery.ecr.aws/aws-controllers-k8s?page=1)
Example is S# Cotroller :  https://gallery.ecr.aws/aws-controllers-k8s/s3-controller

Code for this is available at https://github.com/aws-controllers-k8s/s3-controlle

Clone the git repo which contains the helm chart for the controller as

```
git clone https://github.com/aws-controllers-k8s/s3-controller

```

Add the configuration

```
cd s3-controller/helm/
```

edit `values.yaml` and approx at line 62 add the following config

```
  extraEnvVars:
    - name: AWS_ACCESS_KEY_ID
      valueFrom:
        secretKeyRef:
          name: aws-creds
          key: key
    - name: AWS_SECRET_ACCESS_KEY
      valueFrom:
        secretKeyRef:
          name: aws-creds
          key: secret
```


also at line 98 add the region as

```
aws:
  # If specified, use the AWS region for AWS API calls
  region: "us-east-1"
```


install helm chart as

```
helm install -n ack-system ack-s3 .
```

and validate

```
kubectl get all -n ack-system
```


## Create S3 Bucket from Kubernetes

List the CRDs installed by the controller

```
kubectl get crds | grep s3

kubectl get buckets

kubectl explain buckets
```

Write  a Custom Resource to create a bucket with as

```
apiVersion: s3.services.k8s.aws/v1alpha1
kind: Bucket
metadata:
  name: my-ack-s3-bucket
spec:
  name: my-ack-s3-bucket-xxxxxx  # S3 bucket names need to be globally unique
```

where replace `xxxxxx` with some unique number

and apply

```
kubectl apply -f my-bucket.yaml
```

and like a magic, you shall see a s3 bucket created on AWS.

you could also explore

```
kubectl get buckets
kubectl describe bucket my-ack-s3-bucket
```

and finally

```
kubectl delete bucket my-ack-s3-bucket
```

to see it gone from AWS as well …. poof !
