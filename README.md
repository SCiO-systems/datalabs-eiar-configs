### Jupyterhub

  

The `extra_pod_config` within the JupyterHub Helm chart's `profileList` section establishes a setup ensuring that only one user pod can be scheduled per node, thereby preventing simultaneous scheduling of multiple user pods on the same node.

  

This is accomplished through the `podAntiAffinity` configuration, which prohibits the scheduling of pods with the label `component=singleuser-server` on the same node. This specific label is exclusive to user pods provided by JupyterHub.

  

Moreover, rather than imposing resource `limits` and `guarantees` directly on user pods, they are instead scheduled on designated nodes based on their EC2 instance type. This is facilitated by the `nodeAffinity` configuration, which restricts pod scheduling to nodes labeled with `node.kubernetes.io/instance-type=t3a.xlarge`.

  

Two Karpenter `nodepools` have been configured in order to spawn a "low" and a "high" profile node when all of the available ones are reserved. In such instances, the incoming user pod is considered as unschsduleable from the `kube-scheduler` and Karpenter initiates the creation of a new node to accommodate it. Subsequently, upon termination of the user pod, either manually or due to inactivity, the associated node is getting removed from the cluster and terminated as well.