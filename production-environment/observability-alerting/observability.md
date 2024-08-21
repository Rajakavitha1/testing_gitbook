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
