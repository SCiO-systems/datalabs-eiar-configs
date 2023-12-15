## Metrics Server

The Metrics Server is a scalable, efficient source of container resource metrics for Kubernetes built-in auto scaling pipelines.

Installation: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`

Conrifm: `kubectl get apiservice v1beta1.metrics.k8s.io -o json | jq '.status'`

## Traefik

1. Install Traefik Helm chart (Same command can be used for future updates or changes):

`helm repo add traefik https://traefik.github.io/charts`

`helm repo update`

`helm upgrade --install traefik traefik/traefik --create-namespace --namespace=traefik --values=traefik/helm-chart/values.yaml --version 24.0.0`

2. Get the created AWS domain record of Kubernetes service (EXTERNAL-IP) by executing: `kubectl get svc -n traefik`

## cert-manager

### CRDs installation (https://github.com/cert-manager/cert-manager/releases/tag/v1.12.3)
`kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.3/cert-manager.crds.yaml`

### Helm chart installation
`helm repo add jetstack https://charts.jetstack.io`

`helm repo update`

`helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace --version v1.12.3 --values=cert-manager/helm-chart/values.yml`

### Cluster Issuer Configuration (https://cert-manager.io/docs/configuration/acme/dns01/route53/)
1. Create an IAM policy with the given JSON on the documentation page. Name it: `cert-manager-dns-challenge` and attach it to "EKS" user account.
3. Create Kubernetes secret for storing the AWS keys of the user account: 

    `kubectl create secret generic aws-route53-creds --from-literal=access-key-id=XXXX --from-literal=secret-access-key=YYYY -n cert-manager`

4. Create a ClusterIssuer component which is responsible for making DNS challenges to verify the ownership of the domain: 
    
    `kubectl apply -f cert-manager/clusterIssuer-letsencrypt-production.yaml` (Production)

## Kubernetes configuration

### Private container registry

Create secret for `imagePullSecrets` (namespaced object): 

`kubectl create secret docker-registry registry-creds --docker-server=https://index.docker.io/v1/ --docker-username=xxx --docker-password=YYY --docker-email=fff@koukos.gr -n dev`

### Services setup

Setup `deployment`, `service`, `ingress-route`, `certificate` manifests at the `apps` directory in the form shown for `qvantum-api` service.

This setup will create a deployment and a correspodning service to it of type `ClusterIP` which then will be exposed through Traefik from the `ingress-route` manifest. By providing the created `certificate` as secret it will also have SSL termination.
