---
description: Installing Seldon Enterprise Platform in a learning environment.
---

# Learning Environment

You can install Seldon Enterprise Platform on your local computer that is running a Kubernetes cluster using [kind](https://kubernetes.io/docs/tasks/tools/#kind).

{% hint style="info" %}
**Note**: These instructions are for installing the Seldon Enterprise Platform on a local Kubernetes cluster, with an emphasis on ease of learning. For installing Seldon Enterprise Platform in a production environment, see [cluster requirements](production-environment/#cluster-requirements).
{% endhint %}

## Prerequisites

* Install a Kubernetes cluster that is running version 1.23 or later.
* Install [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl), the Kubernetes command-line tool.
* Install [Helm](https://helm.sh/docs/intro/install/), the package manager for Kubernetes.
* Seldon Enterprise Platform license key. You can reach out to Seldon [support team](https://www.seldon.io/contact) to get a trial license key.

### Installating Seldon Enterprise Platform on a Kubernetes cluster

To install Seldon Enterprise Platform:

1. Create Namespaces:
   *   Create a namespace to contain the main components of Seldon. For example, create the namespace `seldon-system`:

       ```bash
       kubectl create ns seldon-system || echo "Namespace seldon-system already exists"
       ```
   *   Create a namespace to contain the components related to request logging. For example, create the namespace `seldon-logs`:

       ```bash
       kubectl create ns seldon-logs || echo "Namespace seldon-logs already exists"
       ```
   *   Create a namespace that is accessible by Seldon Enterprise Platform. For example, create the namespace `seldon`:

       ```bash
       kubectl create ns seldon || echo "Namespace seldon already exists"
       ```
   *   Annotate the namespace `seldon` so that it is accessible in the Seldon Enterprise Platform UI:

       ```bash
       kubectl label ns seldon seldon.restricted=false --overwrite=true
       ```
2.  Add and update the Helm charts `seldon-charts` to the repository.

    ```bash
    helm repo add seldon-charts https://seldonio.github.io/helm-charts/
    helm repo update seldon-charts
    ```
3.  Create a YAML file to specify the initial configuration. For example, create the `install-values` file. Use your preferred text editor to create and save the file with the following content:

    ```yaml
    image:
      image: seldonio/seldon-deploy-server:2.3.1

    rbac:
     opa:
       enabled:  false
     nsLabelsAuth:
       enabled:  true

    virtualService:
      create: false

    requestLogger:
      create: false

    gitops:
      argocd:
        enabled: false

    enableAppAuth: false

    elasticsearch:
      basicAuth: false

    seldon:
      enabled: false
      knativeEnabled: false

    seldonCoreV2:
      enabled: true
    ```

    \


    {% hint style="info" %}
    **Note**: These configurations do not enable features such as Request logging and GitOps.
    {% endhint %}
4.  Change to the directory that contains the `install-values.yaml` file and then install Seldon Enterprise Platform in the namespace `seldon-system`.

    ```bash
    helm install seldon-enterprise seldon-charts/seldon-deploy --namespace seldon-system  -f install-values.yaml --version 2.3.1
    ```

    When the installation is successful, you should see this:

    ```bash
    NAME: seldon-enterprise
    LAST DEPLOYED: Fri Jul 26 16:43:44 2024
    NAMESPACE: seldon-system
    STATUS: deployed
    REVISION: 1
    NOTES:
    1. Get the application URL by running these commands:
      export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
      echo "Visit http://127.0.0.1:8000/seldon-deploy/ to use your application"
      kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
5.  Check the status of the installation `seldon-enterprise-seldon-deploy`.

    ```bash
    kubectl rollout status deployment/seldon-enterprise-seldon-deploy -n seldon-system
    ```

    When the installation is complete you should see this:

    ```bash
    deployment "seldon-enterprise-seldon-deploy" successfully rolled out
    ```
6.  Get the Pod that is running Seldon Enterprise Platform in the cluster and save it as `$POD_NAME`.

    ```bash
    export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
    ```
7.  You can use port-forwarding to access your application.

    ```bash
    kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
8.  Open your browser and navigate to `http://127.0.0.1:8000/seldon-deploy/` to access Seldon Enterprise Platform.

    ![Seldon Enterprise Platform](sep-welcome-page.png)
9. Apply the trial license and click **Activate**.

{% hint style="info" %}
**Note**: After confirming that port-forwarding to the application is configured, you can open your browser and navigate to `http://127.0.0.1:8000/seldon-deploy/` to access the Seldon Enterprise Platform deployed in the Kubernetes cluster at any time.
{% endhint %}

## Additional Resources

* Seldon Core Documentation
* Seldon Enterprise Documentation
