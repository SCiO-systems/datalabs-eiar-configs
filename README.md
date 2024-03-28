
Cluster has installed Metrics Server, Traefik & cert-manager (same way as [sciocore](https://bitbucket.org/sciocore/sciocore-eks-configs/src/main/)).

  

Karpenter is installed in order to provision additinal servers when requested and monitoring is enabled through Grafana Cloud.
 

### EKS

#### Not Enough Free IP Addresses In Subnet Fix
  
https://niravshah2705.medium.com/release-secondary-ip-production-fix-eks-cf883c31d6dc
  

```
kubectl patch daemonset aws-node -n kube-system --type=json -p='[
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "ENABLE_POD_ENI",
      "value": "true"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "POD_SECURITY_GROUP_ENFORCING_MODE",
      "value": "standard"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG",
      "value": "false"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "AWS_VPC_K8S_CNI_EXTERNALSNAT",
      "value": "true"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "WARM_ENI_TARGET",
      "value": "0"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "WARM_IP_TARGET",
      "value": "8"
    }
  },
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/env/-",
    "value": {
      "name": "MINIMUM_IP_TARGET",
      "value": "15"
    }
  }
]'

```


  

### Jupyterhub

  

The `extra_pod_config` within the JupyterHub Helm chart's `profileList` section establishes a setup ensuring that only one user pod can be scheduled per node, thereby preventing simultaneous scheduling of multiple user pods on the same node.

  

This is accomplished through the `podAntiAffinity` configuration, which prohibits the scheduling of pods with the label `component=singleuser-server` on the same node. This specific label is exclusive to user pods provided by JupyterHub.

  

Moreover, rather than imposing resource `limits` and `guarantees` directly on user pods, they are instead scheduled on designated nodes based on their EC2 instance type. This is facilitated by the `nodeAffinity` configuration, which restricts pod scheduling to nodes labeled with `node.kubernetes.io/instance-type=t3a.xlarge`.

  

Two Karpenter `nodepools` have been configured in order to spawn a "low" and a "high" profile node when all of the available ones are reserved. In such instances, the incoming user pod is considered as unschsduleable from the `kube-scheduler` and Karpenter initiates the creation of a new node to accommodate it. Subsequently, upon termination of the user pod, either manually or due to inactivity, the associated node is getting removed from the cluster and terminated as well.