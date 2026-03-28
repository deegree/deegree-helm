# Collection of helm charts for deegree project

## deegree webservices

There are currently multiple ways to supply database connections to your HELM installation.

### Database configuration with environment variables

Example configuration (`config.yaml`):
```yaml
# enable ingress (previous standard)
ingress:
  enabled: true

# enable httpRoute (new standard)
#httpRoute:
#  enabled: true

# database configuration via JNDI
context: environment

# extra environment variables
environment:
  #DB_DRIVER: oracle.jdbc.OracleDriver
  #DB_URL: jdbc:oracle:thin:@//database.example.org:1521/SERVICE_NAME
  DB_DRIVER: postgresql.jdbc.Driver
  DB_URL: jdbc:postgresql://database.example.org:5432/postgis
  DB_HOSTNAME: database.host.example.org
  DB_USERNAME: username
  DB_PASSWORD: secure-password
  DB_INSTANCE: deegree

```

Install with:

```bash
helm upgrade --install --values config.yaml deegree-webservices deegree-webservices
```

### Database configuration with a Kubernetes secret

Create a K8s secret (example):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: deegree-database-secret
type: Opaque
stringData:
  #DRIVER: oracle.jdbc.OracleDriver
  #URL: jdbc:oracle:thin:@//database.example.org:1521/SERVICE_NAME
  DRIVER: postgresql.jdbc.Driver
  URL: jdbc:postgresql://database.example.org:5432/postgis
  HOSTNAME: "database.host.example.org"
  USERNAME: "username"
  PASSWORD: "secure-password"
  INSTANCE: "deegree"
```

Example configuration (`config.yaml`):
```yaml
# enable ingress (previous standard)
ingress:
  enabled: true

# enable httpRoute (new standard)
#httpRoute:
#  enabled: true

# database configuration via JNDI
context: secret

volumes:
  - name: databasse-secrets
    secret:
      secretName: deegree-database-secret

volumeMounts:
  - name: databasse-secrets
    readOnly: true
    mountPath: "/etc/secrets/database"
```

```bash
helm upgrade --install --values config.yaml deegree-webservices deegree-webservices
```

## Set-up an ingress or httpRoute controller for Docker Desktop

Add traefik to your HELM repositories.

```bash
helm repo add traefik https://traefik.github.io/charts
helm repo update
```

Create a configuration for your ingress/httpRoute controller (``):

```yaml
ingressRoute:
  dashboard:
    enabled: true
    matchRule: Host(`dashboard.localhost`)
    entryPoints:
      - web
providers:
  kubernetesGateway:
    enabled: true
gateway:
  listeners:
    web:
      namespacePolicy:
        from: All
```

Install an instance of traefik as an ingress/httpRoute controller:

```bash
helm install traefik traefik/traefik --values helm-traefik-values.yaml --wait
```
