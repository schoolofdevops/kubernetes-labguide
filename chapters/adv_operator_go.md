# Writing Custom Operators with Go

Sample code used in this lab is available at: [Sample Git Repo](https://github.com/advk8s/static-website-operator)

Setup Sample Code 


```
cd ~
git clone https://github.com/advk8s/static-website-operator.git operator-sample
export SAMPLE_HOME="$(pwd)/operator-sample"

```

**Step 1: Install Go**

Refer to the [official documentation](https://go.dev/doc/install) to install Go on your operating system.


```
# For Linux 
wget -c https://go.dev/dl/go1.24.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.2.linux-amd64.tar.gz 
export PATH=$PATH:/usr/local/go/bin 
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

go version 
```


**Step 2: Install Operator SDK**

Refer to [official documentation to install Operator SDK](https://sdk.operatorframework.io/docs/installation/) on your operating system.

```
# For Linux 
# Download Operator SDK
export ARCH=$(case $(uname -m) in x86_64) echo -n amd64 ;; aarch64) echo -n arm64 ;; *) echo -n $(uname -m) ;; esac)
export OS=$(uname | awk '{print tolower($0)}')
export OPERATOR_SDK_DL_URL=https://github.com/operator-framework/operator-sdk/releases/download/v1.39.2
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
cp $SAMPLE_HOME/controllers/staticwebsite_controller.go $PROJECT_HOME/internal/controller/staticwebsite_controller.go
```


**Step 6: Update the Main.go File**

[Examine the Source Code](https://github.com/advk8s/static-website-operator/blob/main/main.go)

```
cp $SAMPLE_HOME/main.go $PROJECT_HOME/cmd/main.go
```


**Step 7: Generate the CRD Manifests and Code**

```
# Check the CRD manifests before generation
ls config/crd

# Run code generation
make generate

# Update the CRD manifests
make manifests
```

Examine the generated manifests:

```
ls config/crd/bases/ 
```

you shall see a new CRD manifest:

```
websites.example.com_staticwebsites.yaml
```

**Step 8: Build and Load the Operator Image to Kind**
For local testing with kind, we'll build the operator image and load it directly:

```
# Build the image locally
make docker-build IMG=static-website-operator:v0.1.0

# Validate the image
docker image ls 

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

Open a new terinal and start wathing for the resources which will be created by the operator:

```
kubectl get staticwebsites,deploy,svc,pv,pvc
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


## Adding New Features to the Operator


Lets now add two new features to the operator:

  1. Add two new shortnames to the StaticWebsite CRD ie, `sw` and `sws`
  2. Add a new field `NodePort` to the StaticWebsite CRD, defining which the service should be exposed on a NodePort.


**Step 1: Update the CRD**

Update the CRD to include the new shortnames: 

File: `api/v1alpha1/staticwebsite_types.go`

```
// +kubebuilder:object:root=true
// +kubebuilder:subresource:status
// +kubebuilder:printcolumn:name="Phase",type="string",JSONPath=".status.phase"
// +kubebuilder:printcolumn:name="Replicas",type="integer",JSONPath=".spec.replicas"
// +kubebuilder:printcolumn:name="Available",type="integer",JSONPath=".status.availableReplicas"
// +kubebuilder:printcolumn:name="URL",type="string",JSONPath=".status.url"
// +kubebuilder:printcolumn:name="Age",type="date",JSONPath=".metadata.creationTimestamp"
// +kubebuilder:resource:shortName=sw;sws

```
Its the last line that you see above that you have to add 

Also upadte the CRD to accept the new field:

```
// StaticWebsiteSpec defines the desired state of StaticWebsite
type StaticWebsiteSpec struct {
    // Image is the container image for the static website
    // +kubebuilder:validation:Required
    Image string `json:"image"`
    // Replicas is the number of replicas to deploy
    // +kubebuilder:validation:Minimum=1
    // +kubebuilder:default=1
    Replicas int32 `json:"replicas,omitempty"`
    // Resources defines the resource requirements for the container
    // +optional
    Resources ResourceRequirements `json:"resources,omitempty"`
    // Storage defines the storage configuration for the website content
    // +optional
    Storage *StorageSpec `json:"storage,omitempty"`
    // NodePort specifies the NodePort to expose the service on
    // +optional
    // +kubebuilder:validation:Minimum=30000
    // +kubebuilder:validation:Maximum=32767
    NodePort *int32 `json:"nodePort,omitempty"`
}
```


Update the controller to handle the new field:

File: `controllers/staticwebsite_controller.go`

```
// serviceForWebsite returns a website Service object
func (r *StaticWebsiteReconciler) serviceForWebsite(website *websitesv1alpha1.StaticWebsite) *corev1.Service {
        labels := map[string]string{
                "app":        "staticwebsite",
                "controller": website.Name,
        }

        serviceType := corev1.ServiceTypeClusterIP
        servicePort := corev1.ServicePort{
            Port:       80,
            TargetPort: intstr.IntOrString{Type: intstr.Int, IntVal: 80},
            Name:       "http",
        }

        logger := log.FromContext(context.Background())
        logger.Info("Creating service for website",
          "Name", website.Name,
          "HasNodePort", website.Spec.NodePort != nil)

        // If NodePort is specified, set the service type to NodePort and assign the NodePort
        if website.Spec.NodePort != nil {
            serviceType = corev1.ServiceTypeNodePort
            servicePort.NodePort = *website.Spec.NodePort
        }

        svc := &corev1.Service{
            ObjectMeta: metav1.ObjectMeta{
                Name:      website.Name,
                Namespace: website.Namespace,
            },
            Spec: corev1.ServiceSpec{
                Type:     serviceType,
                Selector: labels,
                Ports:    []corev1.ServicePort{servicePort},
            },
        }

        // Set the owner reference for garbage collection
        controllerutil.SetControllerReference(website, svc, r.Scheme)
        return svc
}
```

Also in the same controller, update the status URL in the Reconcile function to include NodePort information when applicable:

```
        // Update the status
        if found.Status.ReadyReplicas > 0 && website.Status.Phase != "Running" {
                website.Status.Phase = "Running"
                website.Status.AvailableReplicas = found.Status.ReadyReplicas

                // Update URL based on whether NodePort is used
                if website.Spec.NodePort != nil {
                    website.Status.URL = fmt.Sprintf("http://<node-ip>:%d", *website.Spec.NodePort)
                } else {
                    website.Status.URL = fmt.Sprintf("http://%s.%s.svc.cluster.local", website.Name, website.Namespace)
                }

                if err := r.Status().Update(ctx, website); err != nil {
                        logger.Error(err, "Failed to update StaticWebsite status")
                        return ctrl.Result{}, err
                }
        } else if found.Status.ReadyReplicas != website.Status.AvailableReplicas {
          ...
          ...
          rest of the code 
```

Rebuild the controller and deploy as

```
make generate
make manifests
make docker-build IMG=static-website-operator:v0.2.0
kind load docker-image static-website-operator:v0.2.0
make install
make deploy IMG=static-website-operator:v0.2.0
```

Validate the shortnames as 

```
kubectl api-resources

kubectl get sw
kubectl get sws
```


Update the sample resource to include the new `nodePort` field:

File: `config/samples/websites_v1alpha1_staticwebsite.yaml`

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
  nodePort: 30400
```

Delete the previous version and create a new instance of static website as, 


```
kubectl delete  -f config/samples/websites_v1alpha1_staticwebsite.yaml
kubectl apply -f config/samples/websites_v1alpha1_staticwebsite.yaml
```

validate 
```
kubectl get sw,deploy,svc,pvc
```

### Cleaning up 

When you are done with the example, you can delete the operator and its resources:

```
make undeploy
```

validate 

```
kubectl get sw,deploy,svc,pvc
```
