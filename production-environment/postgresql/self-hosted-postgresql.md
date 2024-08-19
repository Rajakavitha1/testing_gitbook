---
description: >-
  Installing PostgreSQL in the same Kubernetes cluster that hosts the Seldon
  Enterprise Platform.
---

# Self-hosted PostgreSQL

You can run PostgreSQL in the same Kubernetes cluster that hosts the Seldon Enterprise Platform. We recommend using the [Zalando PostgreSQL operator](https://github.com/zalando/postgres-operator?tab=readme-ov-file#documentation) for managing PostgreSQL installation and maintenance. Refer to their [installation matrix ](https://github.com/zalando/postgres-operator?tab=readme-ov-file#supported-postgres--k8s-versions)for details about the supported PostgreSQL releases.

{% hint style="info" %}
**Note**: These instructions help you quickly set up a PostgreSQL instance. However, this setup is intended for development purposes only and is not recommended for production use.
{% endhint %}

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).
3. Install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

### Installing PostgreSQL in a Kubernetes cluster

1. Clone the Zalando operator repository in your computer.
   `git clone https://github.com/zalando/postgres-operator.git`

1. Change to the `postgres-operator`directory.
    `cd postgres-operator`

1. Create a namespace where you want to install PostgreSQL. For example the name space `postgres`:
    `kubectl create namespace postgres || echo "namespace postgres exists"`

1. Install PostgreSQL using the Helm charts.
   `helm install postgres-operator ./charts/postgres-operator --namespace   postgres`
   After a successful installation, you should see::
   ```
   NAME: postgres-operator 
   LAST DEPLOYED: Tue Aug 13 15:22:02 2024
   NAMESPACE: postgres
   STATUS: deployed
   REVISION: 1
   TEST SUITE: None
   NOTES: To verify that postgres-operator has started, run:
   kubectl --namespace=postgres get pods -l "app.kubernetes.io/name=postgres-operator"
   ```
1.  To install a minimal PostgreSQL setup in the Kubernetes cluster, execute the following:  

    ```bash
    cat << EOF | kubectl apply -f -
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
        version: "15"
    EOF
    ```
    If you would like to install a more complex setup with additional users, databases, replicas, and others see the [official documentation](https://postgres-operator.readthedocs.io/en/latest/) of Zalando operator.

1.  Create the required secret using the auto-generated password:

    ```bash
    kubectl get secret seldon.seldon-metadata-storage.credentials.postgresql.acid.zalan.do -n postgres -o 'jsonpath={.data.password}' | base64 -d > db_pass
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
    After a successful configuration, you should see:
    ```secret/metadata-postgres configured```
1. After the PostgreSQL database and secrets with credentials are ready, add the following to your `install-values.yaml`file.
    ```
    metadata:
      pg:
       enabled: true
       secret: metadata-postgres
       clientTLSSecret: "postgres-client-certs"  # Optional, only needed for SSL verification
      ```
1.  Change to the directory that contains the `install-values.yaml` file and then upgrade the Seldon Enterprise Platform installation in the namespace `seldon-system`.

    ```
    helm upgrade seldon-enterprise seldon-charts/seldon-deploy --namespace seldon-system  -f install-values.yaml --version 2.3.1 --install
    ```
1.  Check the status of the installation seldon-enterprise-seldon-deploy.

    ```
    kubectl rollout status deployment/seldon-enterprise-seldon-deploy -n seldon-system
    ```

    When the installation is complete you should see this:

    ```
    deployment "seldon-enterprise-seldon-deploy" successfully rolled out
    ```
1.  Get the Pod that is running Seldon Enterprise Platform in the cluster and save it as `$POD_NAME`.

    ```
    export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
    ```
1.  You can use port-forwarding to access your application locally.

    ```
    kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
1. Open your browser and navigate to `http://127.0.0.1:8000/seldon-deploy/` to access Seldon Enterprise Platform.    
