---
description: >-
  Learn about connecting a managed PostgreSQL service with Seldon Enterprise
  Platform.
---

# Managed PostgreSQL

You can connect a managed PostgreSQL service in AWS and GCP with Seldon Enterprise Platform. After you have a running PostgreSQL instance, with a database and a user created, you can configure Seldon Enterprise Platform by adding the `metadata-postgres` secret. For more information, see [PostgreSQL with Seldon Enterprise Platform](./).

### Amazon RDS¶

Amazon RDS provides a managed PostgreSQL solution that can be used for storing Seldon Enterprise Platform Model metadata. For information about setting up RDS, managing backups, and other best practices, see [AWS RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html). Ensure the following while setting up RDS:

* The instance is accessible from Seldon Enterprise Platform. If Seldon Enterprise Platform is not on the same VPC, make sure the VPC used by RDS has a public subnet.
* The security group used for accessing the RDS instances allow inbound and outbound traffic from and to Seldon Enterprise Platform. For more information, see Setting up security groups for RDS.

### Cloud SQL for PostgreSQL¶

Google provides a managed PostgreSQL solution that can be used for Seldon Enterprise Platform’s Model Metadata Storage. For information about setting up RDS, managing backups, and other best practices, see [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres).

Ensure that the instance is accessible from Seldon Enterprise Platform. If using the public IP generated for the instance make sure the network that runs Seldon Enterprise Platform is part of the Cloud SQL authorized networks.

### SSL Support¶

By default, Seldon Enterprise Platform does not perform any verification of the Postgres server certificate. To allow server certificate verification, change the SSL mode to `verify-ca` or `verify-full` as needed and place one or more root certificates in the `ca.crt` key in the kubernetes secret. Also add any intermediate certificates to the file if they are needed to link the certificate chain sent by the server to the root certificates stored on the client.

```
kubectl create secret generic -n seldon-system metadata-postgres \
--from-literal=user=your_user \
--from-literal=password=your_password \
--from-literal=host=your.postgres.host \
--from-literal=port=5432 \
--from-literal=dbname=metadata \
--from-literal=sslmode=verify-ca \
--from-file=ca.crt=/path/to/caFile \
--dry-run=client -o yaml \
| kubectl apply -n seldon-system -f -
```

Further, if the server attempts to verify the identity of the client by requesting the client’s leaf certificates, create another kubernetes TLS secret with client certificates for the connection. You can create a secret named `postgres-client-certs` and update the `install-values.yaml` file.&#x20;

kubectl create secret tls -n seldon-system postgres-client-certs\
\--cert=`/path/to/cert`\
\--key=`/path/to/key`\
\--dry-run=client -o yaml\
\| kubectl apply -n seldon-system -f -
