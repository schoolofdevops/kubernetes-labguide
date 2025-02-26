# Resource Lifecycle Management

Resource lifecycle management in ArgoCD encompasses a comprehensive set of sync options that control how resources are created, updated, and deleted during the sync process.

## Core Sync Options

### 1. **Pruning Control**
```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-options: Prune=false
```
   - `Prune=false`: Prevents ArgoCD from deleting resources
   - `PruneLast=true`: Deletes resources only after all other sync operations complete
   - `PrunePropagationPolicy=(foreground|background|orphan)`: Controls how dependent resources are handled during deletion
     - `foreground`: Dependent resources are deleted first (cascading delete)
     - `background`: Dependent resources deleted asynchronously
     - `orphan`: Dependent resources are not deleted

### 2. **Update Strategies**
```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-options: Replace=true
```
   - `Replace=true`: Forces resource replacement instead of patching
   - `ApplyOutOfSyncOnly=true`: Only sync resources that are out of sync
   - `Retry=true`: Attempts to retry failed sync operations

### 3. **Validation and Finalization**
```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-options: SkipSchemaValidation=true
       argocd.argoproj.io/sync-options: RespectIgnoreDifferences=true
```
   - `SkipSchemaValidation=true`: Bypasses Kubernetes schema validation
   - `RespectIgnoreDifferences=true`: Honors ignore difference configurations
   - `DeletePropagation=foreground`: Controls how deletions propagate to dependent objects

### 4. **Finalizer Management**
```yaml
   metadata:
     annotations:
       argocd.argoproj.io/sync-options: DeleteOptions.PropagationPolicy=background
       argocd.argoproj.io/sync-options: DeleteOptions.GracePeriodSeconds=10
```
   - `SetDeletionFinalizers=true/false`: Controls whether ArgoCD manages finalizers
   - Finalizers ensure proper cleanup before resource deletion


## Implementation Patterns

### 1. **Selective Pruning Strategy**
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

### 2. **Safe Update Pattern**
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

## Advanced Use Cases

### 1. **Stateful Application Management**
   - Use `Replace=false` to preserve PVC data
   - Implement `PruneLast=true` for safe database updates
   - Configure appropriate finalizers for data protection

### 2. **Large-Scale Deployments**
   - `ApplyOutOfSyncOnly=true` for performance optimization
   - `Retry=true` for handling transient failures
   - Careful propagation policy selection for complex dependencies

### 3. **Legacy Application Support**
   - `SkipSchemaValidation=true` for non-standard resources
   - `RespectIgnoreDifferences=true` for handling dynamic fields
   - Custom finalizer management for special cleanup requirements

## Best Practices

### 1. **Resource Protection**
   - Carefully consider pruning policies
   - Use finalizers for critical resources
   - Implement appropriate grace periods

### 2. **Performance Optimization**
   - Selectively apply sync options
   - Balance automation vs. manual control
   - Monitor sync operation impact

### 3. **Troubleshooting**
   - Monitor sync operation logs
   - Understand propagation policies
   - Track finalizer completion

### 4. **Common Pitfalls**
   - Avoid mixing incompatible sync options
   - Consider dependencies when setting propagation policies
   - Test complex sync strategies in non-production first

