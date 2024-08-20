---
description: >-
  Installing Kube-Prometheus-stack in the same Kubernetes cluster that hosts the Seldon
  Enterprise Platform.
---

# Observability

You can install `kube-prometheus` to monitor Seldon components, and ensure that the appropriate `ServiceMonitors` are in place for Seldon deployments. The analytics component is configured with the Prometheus integration. The monitoring for Seldon Enterprise Platform is based on the Prometheus Operator and the related `PodMonitor` and `PrometheusRule` resources. 

## Prerequisites

1. Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).
2. Install [Ingress Controller](../ingress-controller/).
3. Install [Docker](https://docs.docker.com/engine/install/)

## Installing Kube-