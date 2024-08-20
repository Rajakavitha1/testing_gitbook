# PostgreSQL

You can install PostgreSQL through [managed cloud services](managed-postgresql.md) or by deploying it [directly within a Kubernetes cluster](self-hosted-postgresql.md). You can also opt to use an existing PostgreSQL instance outside of the Kubernetes cluster that runs Seldon Enterprise Platform.



{% hint style="info" %}
**Note**: PostgreSQL operates as an external component outside of the main Seldon stack. As a result, the administration and management of the PostgreSQL instance used by Seldon are the responsibilities of the cluster administrator.
{% endhint %}

### Amazon RDS¶

Amazon RDS provides a managed PostgreSQL solution that can be used for storing Seldon Enterprise Platform Model metadata. For information about setting up RDS, managing backups, and other best practices, see [AWS RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html). Ensure the following while setting up RDS:

* The instance is accessible from Seldon Enterprise Platform. If Seldon Enterprise Platform is not on the same VPC, make sure the VPC used by RDS has a public subnet.
* The security group used for accessing the RDS instances allow inbound and outbound traffic from and to Seldon Enterprise Platform. For more information, see Setting up security groups for RDS.

### Cloud SQL for PostgreSQL¶

Google provides a managed PostgreSQL solution that can be used for Seldon Enterprise Platform’s Model Metadata Storage. For information about setting up RDS, managing backups, and other best practices, see [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres).

Ensure that the instance is accessible from Seldon Enterprise Platform. If using the public IP generated for the instance make sure the network that runs Seldon Enterprise Platform is part of the Cloud SQL authorized networks.



#### Additional Resources

* [AWS RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)
* [Cloud SQL for PostgreSQL Documentation](https://cloud.google.com/sql/docs/postgres).

