# ArgoCD Sync Policies and Options

## Sync Policy: Automatic vs Manual

### Automatic Sync
Automatic sync enables continuous deployment by automatically keeping the cluster state in sync with the Git repository.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    automated: {}  # Basic automatic sync
    # or with additional options:
    automated:
      prune: true     # Auto-delete resources
      selfHeal: true  # Auto-sync on drift detection
```

#### PRUNE RESOURCES
- Enables automatic deletion of resources that are no longer defined in Git
- **Warning**: Use carefully with stateful applications
- Common use cases:
  - Cleaning up deprecated resources
  - Maintaining environment consistency
  - Preventing resource sprawl

#### SELF HEAL
- Automatically syncs when drift is detected between Git and cluster state
- Useful for:
  - Preventing manual cluster changes
  - Maintaining GitOps compliance
  - Recovering from failed manual interventions

### SET DELETION FINALIZER
```yaml
spec:
  syncPolicy:
    finalizers:
    - resources-finalizer.argocd.argoproj.io
```
- Ensures proper cleanup of application resources
- Prevents orphaned resources during application deletion
- Important for:
  - Stateful applications
  - Resources with dependencies
  - Complex multi-resource applications

## Sync Options

### SKIP SCHEMA VALIDATION
```yaml
spec:
  syncPolicy:
    syncOptions:
    - Validate=false
```
- Bypasses Kubernetes schema validation
- Use cases:
  - Custom Resources without registered CRDs
  - Legacy applications
  - Development/testing scenarios
- **Warning**: Can lead to invalid resource states

### PRUNE LAST
```yaml
spec:
  syncPolicy:
    syncOptions:
    - PruneLast=true
```
- Deletes resources only after all other sync operations complete
- Crucial for:
  - Database migrations
  - Stateful application updates
  - Complex dependency chains

### RESPECT IGNORE DIFFERENCES
```yaml
spec:
  syncPolicy:
    syncOptions:
    - RespectIgnoreDifferences=true
```
- Honors the ignoreDifferences configuration
- Useful for:
  - Fields managed by controllers
  - Dynamic or auto-generated values
  - Status fields

### AUTO-CREATE NAMESPACE
```yaml
spec:
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```
- Automatically creates target namespace if it doesn't exist
- Benefits:
  - Simplified application deployment
  - Self-contained application definitions
  - Reduced manual setup

### APPLY OUT OF SYNC ONLY
```yaml
spec:
  syncPolicy:
    syncOptions:
    - ApplyOutOfSyncOnly=true
```
- Only syncs resources that differ from desired state
- Advantages:
  - Reduced API server load
  - Faster sync operations
  - Minimized disruption

### SERVER-SIDE APPLY
```yaml
spec:
  syncPolicy:
    syncOptions:
    - ServerSideApply=true
```
- Uses server-side apply instead of client-side
- Benefits:
  - Better conflict resolution
  - More accurate field ownership
  - Improved performance for large resources

### PRUNE PROPAGATION POLICY
```yaml
spec:
  syncPolicy:
    syncOptions:
    - PrunePropagationPolicy=foreground
```
Options:
- `foreground`: Cascading deletion, waits for dependents
- `background`: Async deletion of dependents
- `orphan`: Leaves dependent resources untouched

Use cases:
- `foreground`: Most applications (safe default)
- `background`: Large applications with many dependents
- `orphan`: Preserving shared resources

### REPLACE
```yaml
spec:
  syncPolicy:
    syncOptions:
    - Replace=true
```
- Forces complete resource replacement instead of patching
- Important for:
  - Breaking out of invalid states
  - Complete resource recreation
  - **Warning**: Can cause downtime

### RETRY
```yaml
spec:
  syncPolicy:
    syncOptions:
    - Retry=true
```
- Automatically retries failed sync operations
- Configurable with:
  - `retry.limit`: Maximum retry attempts
  - `retry.backoff.duration`: Time between retries
  - `retry.backoff.factor`: Exponential backoff factor
  - `retry.backoff.maxDuration`: Maximum backoff duration

## Best Practices

### Production Environments
1. Use selective automation:
   ```yaml
   spec:
     syncPolicy:
       automated:
         prune: false     # Prevent automatic deletion
         selfHeal: true   # Allow drift correction
   ```

2. Implement safe deletion:
   ```yaml
   spec:
     syncPolicy:
       syncOptions:
       - PruneLast=true
       - PrunePropagationPolicy=foreground
   ```

### Development Environments
1. Enable full automation:
   ```yaml
   spec:
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
       syncOptions:
       - CreateNamespace=true
   ```

### Performance Optimization
1. Selective sync:
   ```yaml
   spec:
     syncPolicy:
       syncOptions:
       - ApplyOutOfSyncOnly=true
       - ServerSideApply=true
   ```

## Common Scenarios and Solutions

### Scenario 1: Stateful Application
```yaml
spec:
  syncPolicy:
    automated:
      prune: false
    syncOptions:
    - PrunePropagationPolicy=orphan
    - RespectIgnoreDifferences=true
```

### Scenario 2: High-Scale Environment
```yaml
spec:
  syncPolicy:
    syncOptions:
    - ServerSideApply=true
    - ApplyOutOfSyncOnly=true
    - PrunePropagationPolicy=background
```

### Scenario 3: Critical Production Service
```yaml
spec:
  syncPolicy:
    automated:
      selfHeal: true
    syncOptions:
    - Validate=true
    - PruneLast=true
    - Retry=true
```

## Troubleshooting Guide

### Common Issues and Solutions


#### 1. **Resources Not Pruning**  
   - Check prune policy configuration  
   - Verify finalizer presence  
   - Review propagation policy  


#### 2. **Sync Failures**  
   - Enable retry mechanism  
   - Check schema validation  
   - Review server-side apply conflicts  


#### 3. **Performance Issues**  
   - Enable ApplyOutOfSyncOnly  
   - Use ServerSideApply  
   - Review resource batch size  


#### 4. **Namespace Issues**
   - Enable auto-create namespace  
   - Check RBAC permissions  
   - Verify namespace exists in cluster  

   