# Writing Custom Operators with Go

Sample code used in this lab is available at: [Sample Git Repo](https://github.com/advk8s/static-website-operator)

Setup Sample Code 


```
cd ~
git clone https://github.com/advk8s/static-website-operator.git operator-sample
export SAMPLE_HOME="$(pwd)/operator-sample"

```

**Step 1: Install Go**

```
# For Linux wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz export PATH=$PATH:/usr/local/go/bin echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

go version 
```


**Step 2: Install Operator SDK**


```
# Download Operator SDK
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.28.0
curl -LO ${OPERATOR_SDK_DL_URL}/operator-sdk_${OS}_${ARCH}

# Install the binary
chmod +x operator-sdk_${OS}_${ARCH}
sudo mv operator-sdk_${OS}_${ARCH} /usr/local/bin/operator-sdk

# Verify installation
operator-sdk version


```

**Step 3: Create a New Operator Project**

Now let's create the operator project structure with a local module path:

```
# Create a directory for your operator
mkdir -p $HOME/static-website-operator
export PROJECT_HOME="$(pwd)/static-website-operator"
cd $HOME/static-website-operator

# Initialize a new operator project with local module path
operator-sdk init --domain example.com --repo static-website-operator

# Create an API and controller
operator-sdk create api --group websites --version v1alpha1 --kind StaticWebsite --resource --controller
```


**Step 4: Define the Custom Resource Definition**

[Examine the Source Code](https://github.com/advk8s/static-website-operator/blob/main/api/v1alpha1/staticwebsite_types.go)

```
cp $SAMPLE_HOME/api/v1alpha1/staticwebsite_types.go $PROJECT_HOME/api/v1alpha1/staticwebsite_types.go
```


**Step 5: Update the Controller**


[Examine the Source Code](https://github.com/advk8s/static-website-operator/blob/main/controllers/staticwebsite_controller.go)

```
cp $SAMPLE_HOME/controllers/staticwebsite_controller.go $PROJECT_HOME/controllers/staticwebsite_controller.go
```


**Step 6: Update the Main.go File**

[Examine the Source Code](https://github.com/advk8s/static-website-operator/blob/main/main.go)

```
cp $SAMPLE_HOME/main.go $PROJECT_HOME/main.go
```


**Step 7: Generate the CRD Manifests and Code**

```
# Run code generation
make generate

# Update the CRD manifests
make manifests
```

**Step 8: Build and Load the Operator Image to Kind**
For local testing with kind, we'll build the operator image and load it directly:

```
# Build the image locally
make docker-build IMG=static-website-operator:v0.1.0

# Load the image into kind
kind load docker-image static-website-operator:v0.1.0
```

**Step 9: Deploy the Operator to Your Kind Cluster**
Deploy the operator to your kind cluster:

```
# Install the CRDs into the cluster
make install

# Deploy the operator with the local image
make deploy IMG=static-website-operator:v0.1.0
```

You can verify that the operator is running:

```
# Check the operator namespace
kubectl get pods -n static-website-operator-system
```

**Step 10: Create a Sample StaticWebsite Resource**

Create a Custom Resource Sample at `config/samples/websites_v1alpha1_staticwebsite.yaml
`
```
apiVersion: websites.example.com/v1alpha1
kind: StaticWebsite
metadata:
  name: sample-website
spec:
  image: nginx:alpine
  replicas: 2
  resources:
    limits:
      cpu: "200m"
      memory: "256Mi"
    requests:
      cpu: "100m"
      memory: "128Mi"
  storage:
    size: "1Gi"
    storageClassName: "standard"  # Use the default storage class in your cluster
    mountPath: "/usr/share/nginx/html"
```


Create a sample StaticWebsite custom resource using the YAML file we created above:

```
# Apply the sample resource
kubectl apply -f config/samples/websites_v1alpha1_staticwebsite.yaml
```

**Step 11: Verify the Deployment**
Check if the resources were created properly:

```
# Check the StaticWebsite resource
kubectl get staticwebsites

# Check the created resources
kubectl get deployments
kubectl get services
kubectl get pvc

# Check the status of the pods
kubectl get pods

# Check the logs of the operator
kubectl logs -n static-website-operator-system deployment/static-website-operator-controller-manager -c manager
```





