---
description: >-
  Installing PostgreSQL in the same Kubernetes cluster that hosts the Seldon
  Enterprise Platform.
---

# Self-hosted PostgreSQL

You can run PostgreSQL in the same Kubernetes cluster that hosts the Seldon Enterprise Platform. We recommend using the [Zalando PostgreSQL operator](https://github.com/zalando/postgres-operator?tab=readme-ov-file#documentation) for managing PostgreSQL installation and maintenance. Refer to their [installation matrix ](https://github.com/zalando/postgres-operator?tab=readme-ov-file#supported-postgres--k8s-versions)for details about the supported PostgreSQL releases.

{% hint style="info" %}
**Note**: These instructions will help you quickly set up a PostgreSQL instance. However, this setup is intended for development purposes only and is not recommended for production use.
{% endhint %}

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install[ Ingress Controller](../ingress-controller/).
3. Install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).



### Installing PostgreSQL in a Kubernetes cluster

1. Install the Zalando operator.
2. Install a minimal PostgreSQL setup.
3. Configure Seldon Enterprise Platform.

### Install Zalando operator

1. Clone the Zalando operator repository in your computer.

&#x20;       `git clone https://github.com/zalando/postgres-operator.git`

2. Change to the `postgres-operator`directory.

&#x20;       `cd postgres-operator`

3. Create a namespace where you want to install PostgreSQL. For example the name space `postgres`:

&#x20;       `kubectl create namespace postgres || echo "namespace postgres exists"`

4. Install PostgreSQL using the Helm charts.

&#x20;      `helm install postgres-operator ./charts/postgres-operator --namespace   postgres`

&#x20;      When the installation is successful you should see this:



4.  **Install a minimal PostgreSQL setup:**

    ```bash
    bashCopy codecat << EOF | kubectl apply -f -
    apiVersion: "acid.zalan.do/v1"
    kind: postgresql
    metadata:
      name: seldon-metadata-storage
      namespace: postgres
    spec:
      teamId: "seldon"
      volume:
        size: 5Gi
      numberOfInstances: 2
      users:
        seldon:  # database owner
        - superuser
        - createdb
      databases:
        metadata: seldon  # dbname: owner
      postgresql:
        version: "13"
    EOF
    ```

    For a more complex setup with additional users, databases, replicas, etc., refer to the operator's [official documentation](https://postgres-operator.readthedocs.io/en/latest/).
5.  **Create the required secret using the auto-generated password:**

    ```bash
    bashCopy codekubectl get secret seldon.seldon-metadata-storage.credentials.postgresql.acid.zalan.do -n postgres -o 'jsonpath={.data.password}' | base64 -d > db_pass
    kubectl create secret generic -n seldon-system metadata-postgres \
      --from-literal=user=seldon \
      --from-file=password=./db_pass \
      --from-literal=host=seldon-metadata-storage.postgres.svc.cluster.local \
      --from-literal=port=5432 \
      --from-literal=dbname=metadata \
      --from-literal=sslmode=require \
      --dry-run=client -o yaml \
      | kubectl apply -n seldon-system -f -
    rm db_pass
    ```

#### Configuring Seldon Enterprise Platform

Once your PostgreSQL database and secrets with credentials are ready, add the following to your `deploy-values.yaml`file. Refer to the SSL support section for configuring client certificates for mutual TLS verification.

```yaml
yamlCopy codemetadata:
  pg:
    enabled: true
    secret: metadata-postgres
    clientTLSSecret: "postgres-client-certs"  # Optional, only needed for SSL verification
```

**Warning:** Enabling `metadata.pg.enabled` will cause the request logger to automatically attempt to retrieve metadata from the Enterprise Platform. Ensure that your configuration is correct for this functionality to work properly.
