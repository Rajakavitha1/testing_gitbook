# Managed PostgreSQL

## Configuring an existing PostgreSQL with Seldon Enterprise Platform 
You can also opt to use an existing PostgreSQL instance outside of the Kubernetes cluster that runs Seldon Enterprise Platform. If you have a database already set up on-premises or in the cloud, you can integrate it with Seldon Enterprise Platform by adding the connection details to the metadata-postgres secret in the namespace where Seldon Enterprise Platform is running. 

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
## Connecting managed PostgreSQL services in AWS and GCP with Seldon Enterprise Platform.

### Amazon RDS¶

Amazon RDS provides a managed PostgreSQL solution that can be used for Seldon Enterprise Platform’s Model Metadata Storage. For setting up RDS for the first time you can follow the docs here.

Some important points to remember while setting up RDS:

Make sure the instance is accessible from Seldon Enterprise Platform. If Seldon Enterprise Platform is not on the same VPC, make sure the VPC used by RDS has a public subnet as discussed here.

Make sure the security group used for accessing the RDS instances allow inbound and outbound traffic from and to Seldon Enterprise Platform. Setting up security groups for RDS is discussed here.

After you have a running PostgreSQL instance, with a database and a user created, you can configure Seldon Enterprise Platform by adding the metadata-postgres secret as discussed in the previous section.

To manage backups see the official documentation. Here is more documentation on other best practices around RDS.

### Google SQL¶

GCP provides a managed PostgreSQL solution that can be used for Seldon Enterprise Platform’s Model Metadata Storage. For setting up Google SQL for the first time you can follow the docs here.

For connection instructions follow the official documentation. Make sure that the instance is accessible from Seldon Enterprise Platform. If using the public IP generated for the instance make sure the network that runs Seldon Enterprise Platform is part of the Cloud SQL authorized networks by following this guide.

After you have a running PostgreSQL instance, with a database and a user created, you can configure Seldon Enterprise Platform by adding the metadata-postgres secret as discussed in the previous section.

### SSL Support¶

By default, Seldon Enterprise Platform will not perform any verification of the Postgres server certificate. To allow server certificate verification, change the SSL mode to verify-ca or verify-full as needed and place one or more root certificates in the ca.crt key in the kubernetes secret. Intermediate certificates should also be added to the file if they are needed to link the certificate chain sent by the server to the root certificates stored on the client.

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
Further, if the server attempts to verify the identity of the client by requesting the client’s leaf certificates, create another kubernetes TLS secret with client certificates for the connection. Here, we create a secret named postgres-client-certs for this purpose. See helm chart configuration section for details on usage of these secrets created.

kubectl create secret tls -n seldon-system postgres-client-certs \
--cert=`/path/to/cert` \
--key=`/path/to/key` \
--dry-run=client -o yaml \
| kubectl apply -n seldon-system -f -