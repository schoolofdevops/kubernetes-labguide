# Server Side Apply Option with ArgoCD


The "Server-Side Apply" option in ArgoCD changes how resource updates are applied to the Kubernetes cluster. Let's break down what this means in simple terms.


## Client-Side Apply (SERVER-SIDE Apply = OFF)

* **Default Behavior**: This is the standard method used by `kubectl apply`.
* **How It Works**:
  * The `kubectl` command sends the entire resource configuration from your local machine (client) to the Kubernetes API server.
  * The server replaces the current resource configuration with the new one provided by the client.
* **Pros**:
  * Simple and straightforward.
  * Works well for many standard use cases.
* **Cons**:
  * Can lead to conflicts if multiple clients are updating the same resource because it replaces the entire configuration.

⠀
## SERVER-SIDE Apply = ON

* **Advanced Behavior**: This is a more sophisticated approach that Kubernetes provides.
* **How It Works**:
  * The client sends a partial or full resource configuration to the server.
  * The server merges this configuration with the existing resource configuration.
  * This approach uses a concept called "field ownership," where different clients can own and update different parts of the same resource.
* **Pros**:
  * Better handling of concurrent updates from multiple clients.
  * More granular control over resource updates, reducing conflicts.
  * Useful for declarative resource management, where multiple tools or teams manage different parts of the same resource.
* **Cons**:
  * Slightly more complex to understand and implement.
  * Requires Kubernetes 1.18 or later.



## Analogy: Updating a Document

Consider a shared document managed by multiple editors:

* **Client-Side Apply (OFF)**:

  * Each editor sends their entire copy of the document.
  * The latest version completely replaces the previous one.
  * Risk of overwriting each other's changes.
* **Server-Side Apply (ON)**:

  * Each editor sends their changes or edits.
  * The server merges these changes into the existing document.
  * Changes from different editors are combined smoothly.



## Detailed Example

Let's use a deployment example to illustrate how Team A (responsible for container images and replicas) and Team B (responsible for resource limits and environment variables) would work with both approaches.

### Example Deployment Resource

First, let's look at the complete deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: company/web-app:v1.2.3
        ports:
        - containerPort: 8080
        env:
        - name: LOG_LEVEL
          value: "info"
        - name: API_ENDPOINT
          value: "https://api.example.com"
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "200m"
            memory: "256Mi"
```

### Client-Side Apply Approach

With client-side apply (the traditional approach), each team would need to maintain and apply the entire YAML file.

#### Team A Updates the Image Version

Team A updates the image from v1.2.3 to v1.3.0 and changes replicas from 3 to 5:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
spec:
  replicas: 5  # Changed from 3 to 5
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: company/web-app:v1.3.0  # Changed from v1.2.3 to v1.3.0
        ports:
        - containerPort: 8080
        env:
        - name: LOG_LEVEL
          value: "info"
        - name: API_ENDPOINT
          value: "https://api.example.com"
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "200m"
            memory: "256Mi"
```

They apply this with: `kubectl apply -f deployment-team-a.yaml`

#### Team B Updates Resources and Environment Variables

Meanwhile, Team B wants to update resource requests and add a new environment variable:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
spec:
  replicas: 3  # Team B might still have the old value
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: company/web-app:v1.2.3  # Team B might still have the old version
        ports:
        - containerPort: 8080
        env:
        - name: LOG_LEVEL
          value: "debug"  # Changed from info to debug
        - name: API_ENDPOINT
          value: "https://api.example.com"
        - name: FEATURE_FLAG_ENABLED  # New environment variable
          value: "true"
        resources:
          limits:
            cpu: "800m"  # Increased from 500m
            memory: "1Gi"  # Increased from 512Mi
          requests:
            cpu: "300m"  # Increased from 200m
            memory: "512Mi"  # Increased from 256Mi
```

They apply this with: `kubectl apply -f deployment-team-b.yaml`

#### The Problem with Client-Side Apply

If Team B applies their changes after Team A, they will **overwrite** Team A's changes:
- The image would revert to v1.2.3
- The replicas would revert to 3

This is because client-side apply replaces the entire resource with the version provided by the last team to apply changes.

### Server-Side Apply Approach

With server-side apply, each team only needs to specify the fields they care about.

#### Team A Updates Using Server-Side Apply

Team A's patch file (`team-a-patch.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: web
        image: company/web-app:v1.3.0
```

They apply this with: `kubectl apply -f team-a-patch.yaml --server-side`

#### Team B Updates Using Server-Side Apply

Team B's patch file (`team-b-patch.yaml`):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
spec:
  template:
    spec:
      containers:
      - name: web
        env:
        - name: LOG_LEVEL
          value: "debug"
        - name: API_ENDPOINT
          value: "https://api.example.com"
        - name: FEATURE_FLAG_ENABLED
          value: "true"
        resources:
          limits:
            cpu: "800m"
            memory: "1Gi"
          requests:
            cpu: "300m"
            memory: "512Mi"
```

They apply this with: `kubectl apply -f team-b-patch.yaml --server-side`

#### The Result with Server-Side Apply

The resulting deployment will include **both teams' changes**:

- Image: company/web-app:v1.3.0 (from Team A)  
- Replicas: 5 (from Team A)  
- Resource limits/requests: increased values (from Team B)  
- Environment variables: updated and new variables (from Team B)  

Server-side apply maintains "field ownership" - each client (team) owns the fields they've applied, and those fields won't be overwritten by other clients unless they explicitly claim ownership of those fields.

### How This Works with ArgoCD

When you enable "Server-Side Apply" in ArgoCD:

1. ArgoCD acts as a client that applies changes with the server-side approach
2. It preserves field ownership for different teams or components
3. It submits only the fields defined in your Git repo for the application
4. Other controllers or teams can own other fields without conflicts

This is particularly useful in environments where:
- Multiple controllers manage different aspects of resources
- You have operators that manage custom resources
- Different teams use different GitOps repositories to manage parts of the same applications

The resulting deployment combines all changes correctly without unexpected overwrites, making it more suitable for complex enterprise environments with multiple teams and automation tools.


⠀
## Conclusion

"Server-Side Apply" is a powerful feature for managing Kubernetes resources, especially in environments where multiple clients or teams update the same resources. It helps reduce conflicts and provides better control over resource updates by merging changes on the server. For most basic use cases, client-side apply is sufficient, but for more complex scenarios involving multiple contributors, server-side apply is highly beneficial.
