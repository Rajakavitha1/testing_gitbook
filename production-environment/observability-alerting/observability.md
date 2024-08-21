---
description: >-
  Installing kube-prometheus-stack in the same Kubernetes cluster that hosts the Seldon Enterprise Platform.
---

# Monitoring

`kube-prometheus`, also known as Prometheus Operator, is a popular open-source project that provides complete monitoring and alerting solutions for Kubernetes clusters. It combines tools and components to create a monitoring stack for Kubernetes environments.

{% hint style="info" %}
**Note**: Always install Prometheus within the same Kubernetes cluster as the Seldon Enterprise Platform.
{% endhint %}

The Seldon Enterprise Platform, along with any deployed models, automatically exposes metrics to Prometheus. By default, certain alerting rules are pre-configured, and an alertmanager instance is included.

You can install `kube-prometheus` to monitor Seldon components, and ensure that the appropriate `ServiceMonitors` are in place for Seldon deployments. The analytics component is configured with the Prometheus integration. The monitoring for Seldon Enterprise Platform is based on the Prometheus Operator and the related `PodMonitor` and `PrometheusRule` resources. 

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).
3. Install [Docker](https://docs.docker.com/engine/install/)

## Installing kube-prometheus

1. Download the `seldon-deploy-install.tar` file that contains required installation resources. For example, to download the installation resources for version `2.3.1` of Seldon Enterprise Platform run the following:
   ```
   TAG=2.3.1 && \
    docker create --name=tmp-sd-container seldonio/seldon-deploy-server:2.3.1 && \
    docker cp tmp-sd-container:/seldon-deploy-dist/seldon-deploy-install.tar.gz . && \
    docker rm -v tmp-sd-container
   ```
1. Extract the contents of the `seldon-deploy-install.tar` file.
   ```
   tar -xzf seldon-deploy-install.tar.gz
   ```
1. Create a namespace for the monitoring components of Seldon Enterprise Platform.
   ```
   kubectl create ns seldon-monitoring || echo "Namespace seldon-monitoring already exists"
   ```
1. Create a YAML file to specify the initial configuration. For example, create the `prometheus-values` file. Use your preferred text editor to create and save the file with the following content: 
   ```
   fullnameOverride: seldon-monitoring
   kube-state-metrics:
     extraArgs:
       metric-labels-allowlist: pods=[*]
   ``` 
   **Note**: Make sure to include `metric-labels-allowlist: pods=[*]` in the Helm values file. If you are using your own Prometheus Operator installation, ensure that the pods labels, particularly `app.kubernetes.io/managed-by=seldon-core`, are part of the collected metrics. These labels are essential for calculating deployment usage rules.

1. Change to the directory that contains the `prometheus-values` file and run the following command to install version `9.5.12` of `kube-prometheus`. 
   ```
   helm upgrade --install prometheus kube-prometheus \
    --version 9.5.12 \
    --namespace seldon-monitoring \
    --values prometheus-values.yaml \
    --repo https://charts.bitnami.com/bitnami
    ```
   When the installation is complete, you should see this:
   ```
   WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
     - alertmanager.resources
     - blackboxExporter.resources
     - operator.resources
     - prometheus.resources
     - prometheus.thanos.resources
   +info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

   ```
1. Check the status of the installation.
   ```
   kubectl rollout status -n seldon-monitoring deployment/seldon-monitoring-operator
   ```
   When the installation is complete, you should see this:
   ```
   Waiting for deployment "seldon-monitoring-operator" rollout to finish: 0 of 1 updated replicas are available...
   deployment "seldon-monitoring-operator" successfully rolled out
   ```
1. To configure monitoring create a dedicated `PodMonitor` and `PrometheusRule` resources. Copy the installation resource files from the `seldon-deploy-install/reference-configuration/metrics/` directory to the current directory.
   ```
   cp seldon-deploy-install/reference-configuration/metrics/seldon-monitor.yaml seldon-monitor.yaml
   cp seldon-deploy-install/reference-configuration/metrics/drift-monitor.yaml drift-monitor.yaml
   cp seldon-deploy-install/reference-configuration/metrics/deploy-monitor.yaml deploy-monitor.yaml
   cp seldon-deploy-install/reference-configuration/metrics/metrics-server-monitor.yaml metrics-server-monitor.yaml
   cp seldon-deploy-install/reference-configuration/metrics/deployment-usage-rules.yaml deployment-usage-rules.yaml
   ```
1. Apply the configurations to the Kubernetes cluster that is running the Seldon Enterprise Platform.
   ```
   kubectl apply -n seldon-monitoring -f seldon-monitor.yaml
   kubectl apply -n seldon-monitoring -f drift-monitor.yaml
   kubectl apply -n seldon-monitoring -f deploy-monitor.yaml
   kubectl apply -n seldon-monitoring -f metrics-server-monitor.yaml
   kubectl apply -f deployment-usage-rules.yaml -n seldon-monitoring
   ```
   When the configuration is complete, you should see this:
   ```
   podmonitor.monitoring.coreos.com/seldon-core created
   podmonitor.monitoring.coreos.com/seldon-drift-detector created
   podmonitor.monitoring.coreos.com/seldon-deploy created
   podmonitor.monitoring.coreos.com/metrics-server created
   prometheusrule.monitoring.coreos.com/seldon-deployment-usage-rules created
   ```
1. You can access Prometheus from outside the cluster by running the following commands:

    ```
    echo "Prometheus URL: http://127.0.0.1:9090/"
    kubectl port-forward --namespace seldon-monitoring svc/seldon-monitoring-prometheus 9090:9090
    ```

1. You can access Alertmanager from outside the cluster by running the following commands:

    ```
    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace seldon-monitoring svc/seldon-monitoring-alertmanager 9093:9093
    ```
1. Add the following to your `install-values.yaml`file.
    ```
    prometheus:
     seldon:
       namespaceMetricName: namespace
       activeModelsNamespaceMetricName: exported_namespace
       serviceMetricName: service
       url: http://seldon-monitoring-prometheus.seldon-monitoring:9090/api/v1/
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

## Next
You may now explore the [Usage Monitor](https://deploy.seldon.io/en/v2.3/contents/operations/usage-monitoring/index.html) feature in Seldon Enterprise Platform.
      

