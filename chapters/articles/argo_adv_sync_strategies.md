# Agro Advanced Module 3 Notes - Advanced Sync Strategies

## Wave Orchestration

Wave orchestration in ArgoCD provides a sophisticated mechanism for controlling the order of resource deployments within an application. This feature is crucial for managing complex dependencies and ensuring proper deployment sequences.

### Key Concepts

1. **Sync Waves**
   - Waves are defined using the annotation: `argocd.argoproj.io/sync-wave`
   - Lower numbers are processed first (e.g., -1 before 0, 0 before 1)
   - Resources within the same wave are processed in parallel
   - Next wave begins only after all resources in the current wave are healthy

### Implementation Patterns

1. **Infrastructure-First Pattern**
```yaml
# Namespace (Wave -2)
apiVersion: v1
kind: Namespace
metadata:
  name: application
  annotations:
    argocd.argoproj.io/sync-wave: "-2"

# ConfigMaps and Secrets (Wave -1)
apiVersion: v1
kind: ConfigMap
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"

# Core Services (Wave 0)
apiVersion: v1
kind: Service
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"

# Deployments (Wave 1)
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

2. **Database-First Pattern**
   - Wave -1: Database StatefulSet
   - Wave 0: Database initialization jobs
   - Wave 1: Application backend services
   - Wave 2: Frontend services

## Sync Phases and Hooks

Sync phases provide fine-grained control over the deployment lifecycle, while hooks enable custom actions at specific points in the sync process.

### Sync Phases

1. **PreSync Phase**
   - Executes before the main sync operation
   - Common uses:
     - Database schema migrations
     - Resource validation
     - Backup creation
     - Service announcements

2. **Sync Phase**
   - Main resource deployment phase
   - Follows wave ordering
   - Respects resource dependencies

3. **PostSync Phase**
   - Executes after successful sync
   - Use cases:
     - Smoke tests
     - Cache warming
     - Notification sending
     - Cleanup operations

### Hook Implementation

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-migration
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: migration
        image: migration-tool:v1
```

### Hook Policies

1. **Delete Policies**
   - `HookSucceeded`: Delete after successful execution
   - `HookFailed`: Delete after failed execution
   - `BeforeHookCreation`: Delete before creating new hook

2. **Timing Policies**
   - `argocd.argoproj.io/sync-wave`: Control execution order
   - `argocd.argoproj.io/hook-delete-policy`: Manage cleanup
   - `argocd.argoproj.io/sync-options`: Fine-tune sync behavior

## Resource Lifecycle Management

Resource lifecycle management in ArgoCD encompasses a comprehensive set of options that control how resources are created, updated, and deleted during the sync process.

### Application Finalizers

Finalizers are Kubernetes mechanisms that prevent resources from being immediately deleted, allowing for cleanup operations to complete first.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  # application specs...
```

- The `resources-finalizer.argocd.argoproj.io` finalizer ensures:
  - When the Application CR is deleted, ArgoCD first deletes all its managed resources
  - The Application CR remains in "Terminating" state until cleanup completes
  - Only after all resources are removed is the finalizer itself removed
  - Then the Application CR is finally deleted
  - This prevents orphaned resources in the cluster

### Core Sync Options

#### 1. **Pruning Control**
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     syncPolicy:
       automated:
         prune: true  # Enable automated pruning during sync
```

#### 2. **Prune Propagation Policy**
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     syncPolicy:
       syncOptions:
       - PrunePropagationPolicy=foreground
```
   
   Options:
   - `foreground`: Parent resources wait for child resources to be deleted first
   - `background`: Parent resources begin deletion, and children are deleted asynchronously
   - `orphan`: Child resources remain when parent resources are deleted

#### 3. **Prune Last**
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     syncPolicy:
       syncOptions:
       - PruneLast=true
```
   - Deletes resources only after all other sync operations complete
   - Provides safer handling for dependent resources

#### 4. **Resource Update Strategies**
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     syncPolicy:
       syncOptions:
       - Replace=true
       - ApplyOutOfSyncOnly=true
```
   - `Replace=true`: Forces resource replacement instead of patching
   - `ApplyOutOfSyncOnly=true`: Only sync resources that are out of sync

#### 5. **Validation Options**
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     syncPolicy:
       syncOptions:
       - SkipSchemaValidation=true
       - RespectIgnoreDifferences=true
```
   - `SkipSchemaValidation=true`: Bypasses Kubernetes schema validation
   - `RespectIgnoreDifferences=true`: Honors ignore difference configurations

### Resource Pruning Exclusions

To protect specific resources from being pruned during sync operations (but not from Application deletion):

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-data
  annotations:
    argocd.argoproj.io/sync-options: Prune=false
```

- `Prune=false`: Prevents this resource from being deleted during normal sync operations
- **Note**: This does NOT protect the resource when the entire Application is deleted

### Sync Hooks for Lifecycle Events

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-backup
  annotations:
    argocd.argoproj.io/hook: PreDelete
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:v1
        command: ['sh', '-c', './backup.sh']
      restartPolicy: Never
```

- Hooks provide actions at specific lifecycle points:
  - `PreSync`: Before sync starts
  - `Sync`: During sync
  - `PostSync`: After sync completes
  - `PreDelete`: Before resources are deleted

### Implementation Patterns

#### 1. **Selective Pruning Strategy**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    automated:
      prune: true  # Enable automated pruning
      selfHeal: true  # Enable drift correction
    syncOptions:
    - PruneLast=true  # Prune after all syncs complete
    - PrunePropagationPolicy=foreground  # Cascading deletion
```

#### 2. **Safe Update Pattern**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    syncOptions:
    - ApplyOutOfSyncOnly=true  # Only update changed resources
    - Validate=false  # Skip validation for specific cases
    - RespectIgnoreDifferences=true  # Honor ignore rules
```

### Advanced Use Cases

#### 1. **Stateful Application Management**
   - Use `Replace=false` to preserve PVC data
   - Implement `PruneLast=true` for safe database updates
   - Configure application finalizers for proper cleanup on deletion
   
   **Example: Complete Stateful Application Protection Strategy**
```yaml
   # Application definition
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: postgres-database
     finalizers:
       - resources-finalizer.argocd.argoproj.io
   spec:
     source:
       repoURL: https://github.com/myorg/databases.git
       path: postgres
       targetRevision: HEAD
     destination:
       server: https://kubernetes.default.svc
       namespace: database
     syncPolicy:
       automated:
         prune: false  # Disable automatic pruning for safety
         selfHeal: true
       syncOptions:
       - PruneLast=true
       - ApplyOutOfSyncOnly=true
       - RespectIgnoreDifferences=true
       
   ---
   # Project-level protection
   apiVersion: argoproj.io/v1alpha1
   kind: AppProject
   metadata:
     name: database-systems
   spec:
     description: "Database applications with data protection"
     # Project specs...
     
     # Critical resources exclusion
     resourceExclusions:
     - group: ""
       kind: PersistentVolumeClaim
       resourceNames:
       - "data-*"
     
     # Scheduled sync windows to prevent updates during peak hours
     syncWindows:
     - kind: deny
       schedule: "0 9 * * 1-5"  # No updates weekdays 9am
       duration: 9h             # For 9 hours (business day)
       applications:
       - "*-database"
```

#### 2. **Large-Scale Deployments**
   - `ApplyOutOfSyncOnly=true` for performance optimization
   - `Retry=true` for handling transient failures
   - Careful propagation policy selection for complex dependencies

#### 3. **Legacy Application Support**
   - `SkipSchemaValidation=true` for non-standard resources
   - `RespectIgnoreDifferences=true` for handling dynamic fields
   - Custom finalizer management for special cleanup requirements

### Best Practices

#### 1. **Resource Protection**
   - Carefully consider pruning policies
   - Use finalizers for critical applications
   - Implement appropriate hooks for data backup before deletion

#### 2. **Performance Optimization**
   - Selectively apply sync options
   - Balance automation vs. manual control
   - Monitor sync operation impact

#### 3. **Troubleshooting**
   - Monitor sync operation logs
   - Understand propagation policies
   - Track finalizer completion

#### 4. **Common Pitfalls**
   - Remember that `Prune=false` annotations don't protect resources during Application deletion
   - Consider dependencies when setting propagation policies
   - Test complex sync strategies in non-production first

## Custom Health Checks

Custom health checks are written in **lua** scripting language and  allow for sophisticated application health monitoring beyond default Kubernetes probes.


### Where to add Custom Health Checks ?

Custom health checks in ArgoCD are added in one of two places:

  **Resource Customizations ConfigMap**: For cluster-wide custom health checks
```bash
   kubectl edit configmap argocd-cm -n argocd
```

  **Application-specific Configuration**: For individual application health checks
```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     # ... other configs
     ignoreDifferences:
     # ... other configs
     resourceCustomizations: |
       customresources.example.com/MyCustomResource:
         health.lua: |
           -- custom health check logic here
```

### Example Health Checks

Here are some examples of custom health checks written in lua scripting language. 

#### Example 1: Kafka Topic Health Check

This checks if a Kafka topic has the correct number of partitions and replication factor:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kafka-app
spec:
  # ... other configs
  resourceCustomizations: |
    kafka.strimzi.io/KafkaTopic:
      health.lua: |
        health_status = {}
        if obj.spec == nil then
          health_status.status = "Degraded"
          health_status.message = "Missing spec"
          return health_status
        end
        
        if obj.status == nil then
          health_status.status = "Progressing"
          health_status.message = "Waiting for status"
          return health_status
        end
        
        -- Check if partitions match desired count
        if obj.spec.partitions ~= obj.status.partitions then
          health_status.status = "Progressing"
          health_status.message = "Partition count doesn't match desired state"
          return health_status
        end
        
        -- Check if replication factor is correct
        if obj.spec.replicas ~= obj.status.replicas then
          health_status.status = "Degraded"
          health_status.message = "Replication factor doesn't match desired state"
          return health_status
        end
        
        health_status.status = "Healthy"
        health_status.message = "Topic is healthy"
        return health_status
```

#### Example 2: Database Custom Resource Health Check

For a custom database resource that reports detailed status:

```yaml
resourceCustomizations: |
  databases.example.com/PostgreSQL:
    health.lua: |
      health_status = {}
      
      if obj.status == nil then
        health_status.status = "Progressing"
        health_status.message = "Status not reported yet"
        return health_status
      end
      
      -- Check database status conditions
      if obj.status.conditions ~= nil then
        for i, condition in ipairs(obj.status.conditions) do
          if condition.type == "Ready" and condition.status == "False" then
            health_status.status = "Degraded"
            health_status.message = condition.message or "Database not ready"
            return health_status
          end
          
          if condition.type == "Syncing" and condition.status == "True" then
            health_status.status = "Progressing"
            health_status.message = "Database is syncing"
            return health_status
          end
        end
      end
      
      -- Check specific database metrics
      if obj.status.metrics ~= nil then
        if obj.status.metrics.connectionCount > 100 then
          health_status.status = "Degraded"
          health_status.message = "Too many connections: " .. obj.status.metrics.connectionCount
          return health_status
        end
        
        if obj.status.metrics.replicationLag > 300 then
          health_status.status = "Degraded"
          health_status.message = "Replication lag too high: " .. obj.status.metrics.replicationLag .. " seconds"
          return health_status
        end
      end
      
      -- Check database instance state
      if obj.status.phase == "Running" and obj.status.databaseInitialized == true then
        health_status.status = "Healthy"
        health_status.message = "Database is running normally"
      elseif obj.status.phase == "Creating" then
        health_status.status = "Progressing"
        health_status.message = "Database is being created"
      else
        health_status.status = "Degraded"
        health_status.message = "Database in unexpected state: " .. (obj.status.phase or "Unknown")
      end
      
      return health_status
```

#### Example 3: Custom Service Mesh Health Check

For an Istio VirtualService with advanced health criteria:

```yaml
resourceCustomizations: |
  networking.istio.io/VirtualService:
    health.lua: |
      health_status = {}
      
      -- Check if the virtual service has hosts defined
      if obj.spec.hosts == nil or #obj.spec.hosts == 0 then
        health_status.status = "Degraded"
        health_status.message = "No hosts defined"
        return health_status
      end
      
      -- Check if routes are defined
      if obj.spec.http == nil or #obj.spec.http == 0 then
        health_status.status = "Degraded"
        health_status.message = "No HTTP routes defined"
        return health_status
      end
      
      -- Check for specific traffic routing issues
      local totalWeight = 0
      local hasRoutes = false
      
      for _, route in ipairs(obj.spec.http) do
        if route.route ~= nil then
          hasRoutes = true
          for _, destination in ipairs(route.route) do
            if destination.weight ~= nil then
              totalWeight = totalWeight + destination.weight
            end
          end
          
          -- If using weighted routing, weights should add up to 100
          if totalWeight > 0 and totalWeight ~= 100 then
            health_status.status = "Degraded"
            health_status.message = "Route weights do not sum to 100: " .. totalWeight
            return health_status
          end
        end
      end
      
      if not hasRoutes then
        health_status.status = "Degraded"
        health_status.message = "No destinations defined in routes"
        return health_status
      end
      
      -- Check if service references exist in the cluster (requires custom status field)
      if obj.status ~= nil and obj.status.validationErrors ~= nil and #obj.status.validationErrors > 0 then
        health_status.status = "Degraded"
        health_status.message = "Validation errors: " .. obj.status.validationErrors[1]
        return health_status
      end
      
      health_status.status = "Healthy"
      health_status.message = "VirtualService configuration is valid"
      return health_status
```

### Why Custom Health Checks Matter

The default health checks in ArgoCD are basic and only look at standard Kubernetes resource states. Custom health checks provide several key advantages:

1. **Business Logic Awareness**: They can check application-specific health criteria that generic checks can't understand
2. **Deep Integration**: They can interpret custom resource status fields that only make sense for specific applications
3. **Advanced Validation**: They can implement complex validation logic that goes beyond "exists/doesn't exist"
4. **Predictive Health**: They can detect conditions that might lead to failures before they happen
5. **Multi-resource Health**: They can aggregate health status across related resources

Without custom health checks, ArgoCD might show a resource as "Healthy" even when it's not functioning correctly from an application perspective. These custom checks bridge the gap between Kubernetes state and actual application health.

### Health Check Best Practices

#### 1. **Comprehensive Checks**
   - Verify all critical dependencies
   - Check both internal and external endpoints
   - Monitor resource utilization

#### 2. **Performance Considerations**
   - Keep checks lightweight
   - Implement appropriate timeouts
   - Cache results when possible

#### 3. **Error Handling**
   - Define clear failure conditions
   - Implement graceful degradation
   - Provide detailed health status messages