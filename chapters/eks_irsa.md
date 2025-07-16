## RBAC and IRSA(IAM Roles for Service Accounts)

### Overview

In this lab, you'll learn how to deploy a Flask application on Amazon EKS using IAM Roles for Service Accounts (IRSA) to securely manage permissions for accessing AWS resources like S3. The Flask application (`schoolofdevops/s3checker:latest`) will check access to an S3 bucket and create a file in it to verify permissions.

### Why Use IRSA?

IAM Roles for Service Accounts (IRSA) provides a way to securely assign AWS IAM roles to Kubernetes pods running on Amazon EKS. This allows your application to assume IAM roles and access AWS resources without needing to manage AWS credentials directly. This improves security by leveraging temporary credentials managed by AWS and reduces the risk of exposing sensitive information.

```
+---------------------+
|     AWS IAM         |
|  +----------------+ |
|  | IAM Role       | |
|  | (S3AccessRole) | |
|  +-------+--------+ |
|          |          |
|          |          |
+----------|----------+
           |
           | Assumes role
           v
+---------------------+
| Amazon EKS Cluster  |
|  +----------------+ |
|  | Service Account| |
|  | (s3-access-sa)  | |
|  +-------+--------+ |
|          |          |
|          |          |
+----------|----------+
           |
           | Bound to
           v
+----------------------------+
| EKS Pod                    |
| +------------------------+ |
| | Flask App Container    | |
| | (schoolofdevops/       | |
| | s3checker:latest)      | |
| +---------+--------------+ |
|           |                |
|           |                |
+-----------|----------------+
            |
            | Uses IRSA
            v
+----------------------------+
| AWS S3 Bucket              |
| (your-s3-bucket-name)      |
|                            |
| - ListBucket               |
| - PutObject                |
| - GetObject                |
+----------------------------+

```

1. **AWS IAM Role Creation**:  
  * An IAM role (`S3AccessRole`) is created with a policy that allows access to the S3 bucket (`your-s3-bucket-name`).

2. **IAM Role Association with EKS Service Account**:  
  * The IAM role is associated with a Kubernetes service account (`s3-access-sa`) in the EKS cluster. This is done using `eksctl` which sets up the necessary trust relationship.

3. **EKS Pod Deployment**:  
  * A Kubernetes pod running the Flask application (`schoolofdevops/s3checker:latest`) is deployed in the EKS cluster. This pod uses the service account (`s3-access-sa`) which is annotated with the IAM role.

4. **IRSA Mechanism**:  
  * The pod assumes the IAM role (`S3AccessRole`) through the service account (`s3-access-sa`). This is facilitated by the IRSA mechanism, which uses Kubernetes service account tokens and AWS STS (Security Token Service) to issue temporary credentials to the pod.

5. **Access S3 Bucket**:  
  * The Flask application running inside the pod uses the temporary credentials obtained via IRSA to access the S3 bucket (`your-s3-bucket-name`). The application can list, put, and get objects in the bucket as per the IAM policy.

⠀
By using IRSA, the application running in the EKS pod can securely access AWS resources without embedding long-term AWS credentials in the application code or environment variables. Instead, it leverages temporary credentials that are managed and rotated by AWS.

### Prerequisites

* An AWS account
* An EKS cluster set up and running
* AWS CLI installed and configured
* `kubectl` installed and configured to access your EKS cluster
* `eksctl` installed for easy EKS management


## Deploy S3 Checker Service

  * Create a S3 bucket in the same region, to test this service with. Note down the name of the bucket.


  * Create a Kubernetes deployment + service YAML file

File: `s3checker-all.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: s3checker
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: s3checker
  template:
    metadata:
      labels:
        app: s3checker
    spec:
      containers:
      - name: s3checker
        image: schoolofdevops/s3checker:latest
        ports:
        - containerPort: 5000
        env:
        - name: BUCKET_NAME
          value: xxxxxxxx
---
apiVersion: v1
kind: Service
metadata:
  name: s3checker
  namespace: default
spec:
  type: NodePort
  selector:
    app: s3checker
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
      nodePort: 30500
```

where,
* replace `xxxxxxxx` with bucket name.

Apply the deployment using `kubectl`:

```
kubectl apply -f s3checker-all.yaml
```

validate

```
kubectl get pods,svc -n default
```


Access the application using the external IP of any Node and `30500` port.


### Configure IRSA


Create an IAM policy that grants the necessary permissions to access your S3 bucket.

File : `s3-access-policy.json`

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:PutObject",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::your-s3-bucket-name",
                "arn:aws:s3:::your-s3-bucket-name/*"
            ]
        }
    ]
}
```

where, replace `your-s3-bucket-name` with the name of the bucket you created earlier.


Save this policy as `s3-access-policy.json` and create the policy using the AWS CLI:

```
aws iam create-policy --policy-name S3AccessPolicy --policy-document file://s3-access-policy.json
```

Note the ARN of the created policy.

#### Create an IAM Role for the Service Account

Use `eksctl` to create an IAM role and associate it with a Kubernetes service account:

```
eksctl utils associate-iam-oidc-provider --region ap-southeast-1 --cluster eks-cluster-01

eksctl create iamserviceaccount \
  --name s3-access-sa \
  --namespace default \
  --cluster eks-cluster-01 \
  --attach-policy-arn arn:aws:iam::<account-id>:policy/S3AccessPolicy \
  --approve \
  --override-existing-serviceaccounts
```

where, replace `arn:aws:iam::<account-id>:policy/S3AccessPolicy` with actual ARN ofthe IAM Policy `S3AccessPolicy` created above. 

validate

```
eksctl get iamserviceaccount --cluster eks-cluster-01
kubectl get sa -n default
```

**Update Your Application Deployment to Use the Service Account**

File: `s3checker-all.yaml`

```
.....
  template:
    metadata:
      labels:
        app: s3checker
    spec:
      serviceAccountName: s3-access-sa
      containers:
      - name: s3checker
```

Apply the deployment using `kubectl`:

```
kubectl apply -f s3checker-all.yaml
```

Now check the application to see if its able to connect to s3 bucket and check the s3 bucket if the application has created a file.

### Clean up

* Empty the s3 bucket and delete it.
* Delete deployment as

```
kubectl delete -f s3checker-all.yaml
```

Remove IAM Service Account  

```
eksctl delete  iamserviceaccount \
  --name s3-access-sa \
  --namespace default \
  --cluster eks-cluster-01
```

### Summary

In this tutorial, you learned how to:

1. Create an IAM policy and role for accessing S3.
2. Associate an IAM role with a Kubernetes service account using IRSA.
3. Deploy a Flask application on EKS using the configured service account.

⠀
By following these steps, you have securely deployed a Flask application on EKS with the necessary permissions to access AWS resources using IRSA. This method enhances security by managing permissions through IAM roles instead of static credentials.
