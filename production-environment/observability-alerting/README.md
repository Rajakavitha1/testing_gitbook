# Monitoring and Alerting

`kube-prometheus-stack`, also known as Prometheus Operator, is a popular open-source project that provides complete monitoring and alerting solutions for Kubernetes clusters. It combines tools and components to create a monitoring stack for Kubernetes environments.

{% hint style="info" %}
**Note**: Always install Prometheus within the same Kubernetes cluster as the Seldon Enterprise Platform.
{% endhint %}

Running a monitoring stack, such as Prometheus, outside of your Kubernetes cluster introduces several challenges:

* Kubernetes API configuration: The external monitoring tool must be properly configured to communicate with the Kubernetes API.
* Access control: The monitoring tool needs appropriate permissions to access the API and monitor resources within the cluster.
* Resource accessibility: By default, in-cluster resources are not exposed externally, making it difficult to monitor them from outside the cluster. Exposing these resources can also introduce security risks.
* Bidirectional access: Prometheus requires access to in-cluster resources, and the Seldon Enterprise Platform also needs to access Prometheus to query metrics. Facilitating this bidirectional flow of traffic may require additional network and security configurations.
* Reliability concerns: Scraping metrics from outside the cluster can be less reliable than from within, potentially leading to lost data and false-positive alerts.

The Seldon Enterprise Platform, along with any deployed models, automatically exposes metrics to Prometheus. By default, certain alerting rules are pre-configured, and an Alertmanager instance is included.

Alertmanager notifies the Enterprise Platform frontend whenever an SLO (Service Level Objective) is breached within the platform infrastructure or a deployed model. The Enterprise Platform also features an alerts page where all currently active alerts can be viewed.

For a hands-on experience, you can explore the alerting functionality through the [alerting demo](./) after installating [monitoring](observability.md) and [alterting](alerting.md) components of Seldon Enterprise Platform.
