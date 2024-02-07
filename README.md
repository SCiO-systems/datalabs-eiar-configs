Cluster has installed Metrics Server, Traefik & cert-manager (same way as [sciocore](https://bitbucket.org/sciocore/sciocore-eks-configs/src/main/)).

Karpenter will not be installed and utilized atm.

## Kubernetes configuration

### Private container registry

Create secret for `imagePullSecrets` (namespaced object): 

`kubectl create secret docker-registry registry-creds --docker-server=https://index.docker.io/v1/ --docker-username=xxx --docker-password=YYY --docker-email=fff@koukos.gr -n dev`

## Jupyterhub

`helm upgrade jupyterhub jupyterhub/jupyterhub -n jupyterhub --cleanup-on-fail --install --version 3.2.1 --values apps/jupyterhub/helm-chart/values.yml --set hub.config.Auth0OAuthenticator.client_id=xxx --set hub.config.Auth0OAuthenticator.client_secret=yyy --set hub.config.Auth0OAuthenticator.oauth_callback_url=zzz --set hub.config.Auth0OAuthenticator.auth0_subdomain=ppp.eu --set hub.db.url="mysql+pymysql://<db-username>:<db-password>@<db-hostname>:<db-port>/<db-name>" --timeout 1200s`