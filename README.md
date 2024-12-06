This repository contains configurations and resources for deploying and managing DataLabs on Kubernetes. The components include JupyterHub for interactive computing, Karpenter for Kubernetes cluster scaling, and Traefik for ingress and routing.

---

## Project Structure

## Components Overview

### 1. **cert-manager**:

**cert-manager** is a powerful SSL certificate controller for Kubernetes. It manages lifecycle of certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry.



```
helm repo add jetstack https://charts.jetstack.io

helm repo update

helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace --version 1.15.3 --set crds.enabled=true
```

Create a ClusterIssuer component which is responsible for making HTTP challenges to verify the ownership of the domain:

```
kubectl apply -f ./kube/cert-manager/clusterIssuer-letsencrypt-http.yaml
```

### 2. **Karpenter**
Karpenter is used to dynamically scale Kubernetes nodes based on workload requirements. The `karpenter` directory includes:
- **default-template/**: Configurations for default node pools.
  - **default-nodeclass.yml**: Base node class configuration.
  - **high-profile-nodepool.yml**: High-profile node pool settings.
  - **medium-profile-nodepool.yml**: Medium-profile node pool settings.
  - **low-profile-nodepool.yml**: Low-profile node pool settings.


```
kubectl apply -f ./karpenter/default-template/
```

### 3. **Traefik**
Traefik is used as the ingress controller for managing external traffic to services. The `traefik` directory includes:
- **helm-chart/values.yml**: Configuration file for deploying Traefik via Helm.
- **jupyterhub-ingress-route.yml**: Ingress route for accessing JupyterHub.


Deploy Traefik using the Helm chart:

```
helm upgrade --install traefik ./traefik/helm-chart -f ./traefik/helm-chart/values.yml
```

Apply the JupyterHub ingress route:

```
kubectl apply -f ./traefik/jupyterhub-ingress-route.yml
```

### 4. **JupyterHub**
JupyterHub is configured to provide interactive notebook environments for users. The `jupyterhub` directory contains resources for its deployment:
- **certificate.yml**: Defines TLS certificates for secure communication.
- **helm-chart/values.yaml**: Configuration file for deploying JupyterHub using Helm.
- **volumes/shared-data**:
  - **pv-shared-data.yaml**: Defines a PersistentVolume for shared storage.
  - **pvc-shared-data.yaml**: Creates a PersistentVolumeClaim to request shared storage.
  - **nfs-mount-job.yaml**: Sets up an NFS mount for shared data between users.

Configure `certificate.yml` with valid TLS credentials.

```
kubectl apply -f ./jupyterhub/certificate.yml
```

Create shared volume resource
```
kubectl apply -f ./jupyterhub/volumes/shared-data/
```

Deploy JupyterHub using the Helm chart:
```
helm upgrade --install jupyterhub ./jupyterhub/helm-chart -f ./jupyterhub/helm-chart/values.yaml
```
The `extra_pod_config` within the JupyterHub Helm chart's `profileList` section establishes a setup ensuring that only one user pod can be scheduled per node, thereby preventing simultaneous scheduling of multiple user pods on the same node.

This is accomplished through the `podAntiAffinity` configuration, which prohibits the scheduling of pods with the label `component=singleuser-server` on the same node. This specific label is exclusive to user pods provided by JupyterHub.  

Moreover, rather than imposing resource `limits` and `guarantees` directly on user pods, they are instead scheduled on designated nodes based on their EC2 instance type. This is facilitated by the `nodeAffinity` configuration, which restricts pod scheduling to nodes labeled with `node.kubernetes.io/instance-type=t3a.xlarge`.  

Two Karpenter `nodepools` have been configured in order to spawn a "low" and a "high" profile node when all of the available ones are reserved. In such instances, the incoming user pod is considered as unschsduleable from the `kube-scheduler` and Karpenter initiates the creation of a new node to accommodate it. Subsequently, upon termination of the user pod, either manually or due to inactivity, the associated node is getting removed from the cluster and terminated as well.