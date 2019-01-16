#### 1. Create a new K8s cluster and change its network plugin from `Weave` to `Cannel`.  
`Reference`: [Create K8s cluster using Kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm)

#### 2. When you create a K8s cluster using Kubeadm, you lost the init token accidently. How do you recover this init token again?
`Reference`: [Kubeadm-Token](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/)

#### 3. You have launched a K8s cluster of version 1.8 and want to upgrade it to version 1.9. How do you achieve that?
`Reference`: [Kubeadm Upgrade](https://kubernetes.io/docs/tasks/administer-cluster/upgrade-downgrade/kubeadm-upgrade-1-9/)

#### 4. While running kubeadm init comamnd, you see the following error. How would you fix it? [preflight] 
```
WARNING: ebtables not found in system path"
```

#### 5. kubeadm init command fails with following error. How woudl you fix it?
```
error: failed to run Kubelet: failed to create kubelet:
  misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
```
`Reference 1`: [Link 1](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-docker)
`Reference 2`: [Link 2](https://kubernetes.io/docs/setup/independent/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-master-node)

#### 6. You can also use a sandbox environment like https://labs.play-with-k8s.com/
Keep in mind that the environment that you create will only be available for 4 hours. So just use it for learning purposes.
`Reference`: [Play with K8s](https://labs.play-with-k8s.com/)

#### 7. Create another k8s cluster(you can use play-with-k8s for this) and use Calico as your pod network plugin. Use the following documentation for the instructions.
`Reference`: [Pod Networking](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)

#### 8. You have lost the init token and discovery hash provided by the kubeadm init command and you are not able to join any nodes to your cluster. How would you retrieve them?
`Reference`: [Recover Kubeadm Tokens](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#join-nodes)

#### 9. You have launched a K8s cluster of version 1.8 and want to upgrade it to version 1.9. How do you achieve that?
`Reference`:[Upgrade K8s Cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/#upgrading-your-control-plane)
