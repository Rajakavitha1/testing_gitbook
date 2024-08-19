---
description: >-
  Learn about connecting a managed PostgreSQL service with Seldon Enterprise
  Platform.
---

# Managed PostgreSQL

You can connect a managed PostgreSQL service in [AWS RDS](./#amazon-rds) and [Google Cloud SQL](./#cloud-sql-for-postgresql) with Seldon Enterprise Platform. After you have a running PostgreSQL instance, with a database and a user created, you can configure Seldon Enterprise Platform by adding the `metadata-postgres` secret.

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).

To configure a PostgreSQL database with Seldon Enterprise Platform, execute the following after substituting the variables `<your_user>`, `<your_password>`,`<metadata>`, and `<your.postgres.host>` with those from your database:

```
kubectl create secret generic -n seldon-system metadata-postgres \
  --from-literal=user=<your_user> \
  --from-literal=password=<your_password> \
  --from-literal=host=<your.postgres.host> \
  --from-literal=port=5432 \
  --from-literal=dbname=<metadata >\
  --from-literal=sslmode=require \
  --dry-run=client -o yaml \
  | kubectl apply -n seldon-system -f -
```

## SSL Support¶

By default, Seldon Enterprise Platform does not perform any verification of the Postgres server certificate. To allow server certificate verification, change the SSL mode to `verify-ca` or `verify-full` as needed and place one or more root certificates in the `ca.crt` key in the kubernetes secret. Also add any intermediate certificates to the file if they are needed to link the certificate chain sent by the server to the root certificates stored on the client. Execute the following after substituting the variables `<your_user>`, `<your_password>`,`<metadata>`, and `<your.postgres.host>` with those from your database:

```
kubectl create secret generic -n seldon-system metadata-postgres \
--from-literal=user=<your_user> \
--from-literal=password=<your_password> \
--from-literal=host=<your.postgres.host> \
--from-literal=port=5432 \
--from-literal=dbname=<metadata> \
--from-literal=sslmode=verify-ca \
--from-file=ca.crt=/path/to/caFile \
--dry-run=client -o yaml \
| kubectl apply -n seldon-system -f -
```

Further, if the server attempts to verify the identity of the client by requesting the client’s leaf certificates, create another kubernetes TLS secret with client certificates for the connection. You can create a secret named `postgres-client-certs`.

```
kubectl create secret tls -n seldon-system postgres-client-certs\
\--cert=`/path/to/cert`\
\--key=`/path/to/key`\
\--dry-run=client -o yaml\
\| kubectl apply -n seldon-system -f -
```

Finally, update the `install-values.yaml` file.

```
metadata:
  pg:
    enabled: true
    secret: metadata-postgres
    clientTLSSecret: "postgres-client-certs"  # Optional, only needed for SSL verification
```
