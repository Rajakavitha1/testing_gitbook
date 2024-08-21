---
description: >-
  Installing kube-prometheus-stack in the same Kubernetes cluster that hosts the
  Seldon Enterprise Platform.
---

# Alerting

The Seldon Enterprise Platform, along with any deployed models, automatically exposes metrics to Prometheus. By default, certain alerting rules are pre-configured, and an alertmanager instance is included.

You can configure Alertmanager to send alerts through email or Slack. It can also be integrated into an incident response tool. To receive alerts when using Seldon Enterprise Platform you need to:
1. Configure alerts
2. Integrate into an incident response

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).
3. Install [kube-prometheus](observability.md#installing-kube-prometheus)

## Configuring alerts in Seldon Enterprise Manager

1.  To configure default alerting rules, copy the installation resource files from the `seldon-deploy-install/reference-configuration/metrics/` directory to the current directory. To configure custom alerts, see the custom alerts section.

    ```
     cp seldon-deploy-install/reference-configuration/metrics/user-alerts.yaml user-alerts.yaml
     cp seldon-deploy-install/reference-configuration/metrics/infra-alerts.yaml infra-alerts.yaml
     cp seldon-deploy-install/reference-configuration/metrics/drift-alerts.yaml drift-alerts.yaml
    ```
2.  Apply the configurations to the Kubernetes cluster that is running the Seldon Enterprise Platform.

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
3.  Create a YAML file to specify the initial configuration. For example, create the `alertmanager.yaml` file. Use your preferred text editor to create and save the file with the following content:

    ```yaml
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
    For more information about configuring alerts during authetication, see Authentication alerts section.
4.  Apply the Altermanager configurations in the Kubernetes cluster that is running Seldon Enterprise Platform:

    ```
    kubectl delete secret -n seldon-monitoring alertmanager-seldon-monitoring-alertmanager || echo "Does not yet exist"
    kubectl apply -f alertmanager.yaml -n seldon-monitoring
    ```

    When the configurations are applied, you should see this:

    ```
    secret "alertmanager-seldon-monitoring-alertmanager" deleted
    secret/alertmanager-seldon-monitoring-alertmanager created
    ```
5.  You can access Alertmanager from outside the cluster by running the following commands:

    ```
    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace seldon-monitoring svc/seldon-monitoring-alertmanager 9093:9093
    ```
6.  Get the Pod that is running Seldon Enterprise Platform in the cluster and save it as `$POD_NAME`.

    ```
    export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
    ```
7.  You can use port-forwarding to access your application locally.

    ```
    kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
8. Open your browser and navigate to `http://127.0.0.1:8000/seldon-deploy/` to access Seldon Enterprise Platform.

### Custom alerts¶

You can also define your own custom alerting rules in Prometheus.

1. Create a file called `custom-alert.yaml` that contains your new rules. You can find some examples in the file `user-alerts.yaml` file located in the `seldon-deploy-install/reference-configuration/metrics/` folder.
2.  Apply the alerts using:

    ```
    kubectl create -f custom-alert.yaml
    ```
 ### Authentication alerts

 * If you are using App Level Authentication you need to add `http_config` in the `webhook_configs` section of `alertmanager.yaml`. This needs a client that has been configured to access the [Seldon Enterprise Platform API](https://deploy.seldon.io/en/v2.3/contents/product-tour/api/index.html#authentication). The token_url value may vary, depending on your OIDC provider.
 ```yaml
    webhook_configs:
      - url: "http://seldon-deploy.seldon-system:80/seldon-deploy/api/v1alpha1/webhooks/firing-alert"
        http_config:
          oauth2:
            client_id: "${OIDC_CLIENT_ID}"
            client_secret: "${OIDC_CLIENT_SECRET}"
            scopes: [openid]
            token_url: "${OIDC_HOST}/auth/realms/${OIDC_REALM}/protocol/openid-connect/token"
            # Note: only needed if using a self-signed certificate on your OIDC provider
            tls_config:
              insecure_skip_verify: true
 ```
* If you are using a self-signed certificate on your OIDC provider then you need to set `insecure_skip_verify` in the `tls_config` of the `oauth2` block. Alternatively, you can mount your CA certificate onto the Alertmanager instance to validate the server certificate using `ca_file`. For more information see, the [Prometheus documentation](https://prometheus.io/docs/alerting/latest/configuration/#tls_config).   


## Next

You may now explore the [Alerting Integrations](https://deploy.seldon.io/en/v2.3/contents/demos/general/alerting-integration/index.html) feature in Seldon Enterprise Platform.
