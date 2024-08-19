---
description: >-
  Learn about connecting a managed PostgreSQL service with Seldon Enterprise
  Platform.
---

# Managed PostgreSQL

You can connect a managed PostgreSQL service in AWS and GCP with Seldon Enterprise Platform. After you have a running PostgreSQL instance, with a database and a user created, you can configure Seldon Enterprise Platform by adding the `metadata-postgres` secret. For more information, see [PostgreSQL with Seldon Enterprise Platform](./).


## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).

### Amazon RDS¶

Amazon RDS provides a managed PostgreSQL solution that can be used for storing Seldon Enterprise Platform Model metadata. For information about setting up RDS, managing backups, and other best practices, see [AWS RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html). Ensure the following while setting up RDS:

* The instance is accessible from Seldon Enterprise Platform. If Seldon Enterprise Platform is not on the same VPC, make sure the VPC used by RDS has a public subnet.
* The security group used for accessing the RDS instances allow inbound and outbound traffic from and to Seldon Enterprise Platform. For more information, see Setting up security groups for RDS.

### Cloud SQL for PostgreSQL¶

Google provides a managed PostgreSQL solution that can be used for Seldon Enterprise Platform’s Model Metadata Storage. For information about setting up RDS, managing backups, and other best practices, see [Cloud SQL for PostgreSQL](https://cloud.google.com/sql/docs/postgres).

Ensure that the instance is accessible from Seldon Enterprise Platform. If using the public IP generated for the instance make sure the network that runs Seldon Enterprise Platform is part of the Cloud SQL authorized networks.
