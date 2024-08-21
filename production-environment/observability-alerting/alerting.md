---
description: >-
  Installing kube-prometheus-stack in the same Kubernetes cluster that hosts the Seldon Enterprise Platform.
---

# Alerting
The Seldon Enterprise Platform, along with any deployed models, automatically exposes metrics to Prometheus. By default, certain alerting rules are pre-configured, and an alertmanager instance is included.

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).
3. Install [Docker](https://docs.docker.com/engine/install/)
4. Install [kube-prometheus](../observability-alerting/observability.md)

## Configuring alerts in Seldon Enterprise Manager

1. To configure default alerting rules, copy the installation resource files from the `seldon-deploy-install/reference-configuration/metrics/` directory to the current directory.
   ```
   cp seldon-deploy-install/reference-configuration/metrics/user-alerts.yaml user-alerts.yaml
    cp seldon-deploy-install/reference-configuration/metrics/infra-alerts.yaml infra-alerts.yaml
    cp seldon-deploy-install/reference-configuration/metrics/drift-alerts.yaml drift-alerts.yaml
   ```
1. Apply the configurations to the Kubernetes cluster that is running the Seldon Enterprise Platform.
   ```
    kubectl apply -n seldon-monitoring -f infra-alerts.yaml
    kubectl apply -n seldon-monitoring -f user-alerts.yaml
    kubectl apply -n seldon-monitoring -f drift-alerts.yaml
   ```
   When the configuration is complete, you should see this:
   ```
    prometheusrule.monitoring.coreos.com/deploy-infra-alerts created
    prometheusrule.monitoring.coreos.com/deploy-user-alerts created
    prometheusrule.monitoring.coreos.com/seldon-drift-alerts created
   ```
1. Create a YAML file to specify the initial configuration. For example, create the `alertmanager.yaml` file. Use your preferred text editor to create and save the file with the following content:
    ```
    kind: Secret
    apiVersion: v1
    metadata:
      name: alertmanager-seldon-monitoring-alertmanager
    stringData:
      alertmanager.yaml: |
        receivers:
          - name: default-receiver
          - name: deploy-webhook
            webhook_configs:
              - url: "http://seldon-deploy.seldon-system:80/seldon-deploy/api/v1alpha1/webhooks/firing-alert"
        route:
          group_wait: 10s
          group_by: ['alertname']
          group_interval: 5m
          receiver: default-receiver
          repeat_interval: 3h
          routes:
            - receiver: deploy-webhook
              matchers:
                - severity =~ "warning|critical"
                - type =~ "user|infra"
        ```
1. Apply the Altermanager configurations in the Kubernetes cluster that is running Seldon Enterprise Platform:
    ```
    kubectl delete secret -n seldon-monitoring alertmanager-seldon-monitoring-alertmanager || echo "Does not yet exist"
    kubectl apply -f alertmanager.yaml -n seldon-monitoring
    ```
    When the configurations are applied, you should see this:
    ```
    secret "alertmanager-seldon-monitoring-alertmanager" deleted
    secret/alertmanager-seldon-monitoring-alertmanager created
    ```

1. You can access Alertmanager from outside the cluster by running the following commands:

    ```
    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace seldon-monitoring svc/seldon-monitoring-alertmanager 9093:9093
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

### Custom alerts¶

You can also define your own custom alerting rules in Prometheus.

1. Create a file called `custom-alert.yaml` that contains your new rules. You can find some examples in the file `user-alerts.yaml` file located in the `seldon-deploy-install/reference-configuration/metrics/` folder.

1. Apply the alerts using:

   ```
   kubectl create -f custom-alert.yaml
   ```

## Next
You may now explore the [Alerting Integrations](https://deploy.seldon.io/en/v2.3/contents/demos/general/alerting-integration/index.html) feature in Seldon Enterprise Platform.