# Keycloak with PostgreSQL

This example shows how to configure Keycloak to use a Microsoft SQL database.

# Prerequisits
- NGINX INGRESS
- CERTMANAGER

# Setup

## Create Namespace and Deploy MSSQL Server instance
```
kubectl create ns keycloak-mssql
kubectl apply -f .\charts\keycloak\examples\mssql\mssql.yaml -n keycloak-mssql
```

# Deploy Keycloak
Using chart source code
```
helm upgrade --install keycloak .\charts\keycloak\  --values .\charts\keycloak\examples\mssql\keycloak-mssql-values.yaml -n keycloak-mssql
```
Using chart from PQS Helm Repo

```
helm repo add pqs https://pqsdev.github.io/helm-charts
helm repo pqs update
helm install keycloak pqs\keycloak\  --values .\charts\keycloak\examples\mssql\keycloak-mssql-values.yaml -n keycloak-mssql
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
kubectl delete ns keycloak-mssql
```
