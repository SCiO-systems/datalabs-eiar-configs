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

## Jupyterhub

`helm upgrade jupyterhub jupyterhub/jupyterhub -n jupyterhub --cleanup-on-fail --install --version 3.2.1 --values apps/jupyterhub/helm-chart/values.yml --set hub.config.Auth0OAuthenticator.client_id=xxx --set hub.config.Auth0OAuthenticator.client_secret=yyy --set hub.config.Auth0OAuthenticator.oauth_callback_url=zzz --set hub.config.Auth0OAuthenticator.auth0_subdomain=ppp.eu --set hub.db.url="mysql+pymysql://<db-username>:<db-password>@<db-hostname>:<db-port>/<db-name>" --timeout 1200s`