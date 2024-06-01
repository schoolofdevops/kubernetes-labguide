# Cleanup Checklist


[x] Delete EKS Cluster

```
eksctl delete cluster --force -f cluster.yaml
```  

[x] Delete Cloudformation Stacks - check for reaming Cloudformation Stacks if any and delete those.

[x] Delete S3 Bucket

[x] Delete EC2 Volumes

[x] Delete Load Balancers + Target Groups  
