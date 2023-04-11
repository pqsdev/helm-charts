# Keycloak with Openshift

This example shows how to configure Keycloak to use a Microsoft SQL database.
## Requirements
OpenShift Cluster or OpenShift Local

## Deploy Keycloak
Using chart source code
```
helm upgrade --install keycloak .\charts\keycloak\  --values .\charts\keycloak\examples\openshift\keycloak-openshift-values.yaml -n keycloak-openshift --dry-run
```

Using chart from PQS Helm Repo

```
helm repo add pqs https://pqsdev.github.io/helm-charts
helm repo pqs update
helm upgrade --install keycloak pqs/keycloak --values .\charts\keycloak\examples\openshift\keycloak-openshift-values.yaml -n keycloak-openshift --dry-run
```

# Access Keycloak
Once Keycloak is running, forward the HTTP service port to 8080.

```
kubectl port-forward service/keycloak-keycloakx-http 8080:80
```

You can then access the Keycloak Admin-Console via `https://keycloak.localdev.me/auth` with
username: `admin` and password: `secret`.

# Remove Keycloak-mssql napespace

```
kubectl delete ns keycloak-os
```
