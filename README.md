Cluster has installed Metrics Server, Traefik & cert-manager (same way as [sciocore](https://bitbucket.org/sciocore/sciocore-eks-configs/src/main/)).

Karpenter is installed too and monitoring is enabled through Grafana Cloud.

### Jupyterhub

`helm upgrade jupyterhub jupyterhub/jupyterhub -n datalabs --cleanup-on-fail --install --version 3.2.1 --values jupyterhub/helm-chart/values.yaml --set hub.config.Auth0OAuthenticator.client_id=xxx --set hub.config.Auth0OAuthenticator.client_secret=yyy --set hub.config.Auth0OAuthenticator.oauth_callback_url=zzz --set hub.config.Auth0OAuthenticator.auth0_subdomain=ppp.eu --set hub.db.url="mysql+pymysql://<db-username>:<db-password>@<db-hostname>:<db-port>/<db-name>" --timeout 1200s`