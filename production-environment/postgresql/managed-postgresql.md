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

## Configure PostgreSQL with Seldon Enterprise Platform

To configure a PostgreSQL database with Seldon Enterprise Platform:

1.  Execute the following after substituting the variables `<your_user>`, `<your_password>`,`<metadata>`, and `<your.postgres.host>` with those from your database:

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

    {% hint style="info" %}
    **Note**: If you would like to configure PostgreSQL with SSL, see the [SSL Support section](managed-postgresql.md#ssl-support).
    {% endhint %}
2.  Add the following to your `install-values.yaml`file.

    ```
    metadata:
      pg:
       enabled: true
       secret: metadata-postgres
       clientTLSSecret: "postgres-client-certs"  # Optional, only needed for SSL verification
    ```
3.  Change to the directory that contains the `install-values.yaml` file and then upgrade the Seldon Enterprise Platform installation in the namespace `seldon-system`.

    ```
    helm upgrade seldon-enterprise seldon-charts/seldon-deploy --namespace seldon-system  -f install-values.yaml --version 2.3.1 --install
    ```
4.  Check the status of the installation seldon-enterprise-seldon-deploy.

    ```
    kubectl rollout status deployment/seldon-enterprise-seldon-deploy -n seldon-system
    ```

    When the installation is complete you should see this:

    ```
    deployment "seldon-enterprise-seldon-deploy" successfully rolled out
    ```
5.  Get the Pod that is running Seldon Enterprise Platform in the cluster and save it as `$POD_NAME`.

    ```
    export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
    ```
6.  You can use port-forwarding to access your application locally.

    ```
    kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
7. Open your browser and navigate to `http://127.0.0.1:8000/seldon-deploy/` to access Seldon Enterprise Platform.

### SSL Support¶

By default, Seldon Enterprise Platform does not perform any verification of the Postgres server certificate. To allow server certificate verification, change the SSL mode to `verify-ca` or `verify-full` as needed and place one or more root certificates in the `ca.crt` key in the kubernetes secret. Also add any intermediate certificates to the file if they are needed to link the certificate chain sent by the server to the root certificates stored on the client.



&#x20;Execute the following after substituting the variables `<your_user>`, `<your_password>`,`<metadata>`, and `<your.postgres.host>` with those from your database:

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

## Next

You may now explore the [Model Catalog](https://deploy.seldon.io/en/v2.3/contents/demos/general/model-catalog/index.html) function in Seldon Enterprise Platform.