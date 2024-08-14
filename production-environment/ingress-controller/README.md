---
description: >-
  Learn more about the Ingress controllers that Seldon Enterprise Platform
  supports.
---

# Ingress Controller

An ingress controller functions as a reverse proxy and load balancer, implementing a Kubernetes Ingress. It adds an abstraction layer for traffic routing by receiving traffic from outside the Kubernetes platform and load balancing it to Pods running within the Kubernetes cluster.

The Seldon Enterprise Platform supports Istio ingress controllers. Both Seldon Core 1 and Seldon Core 2 integrate most effectively with your Seldon Enterprise Platform ecosystem when using an Istio ingress controller.

This table lists the features and the ingress controllers for Seldon Core 1 and Seldon Core 2.

<table><thead><tr><th width="252">Seldon Core 1</th><th width="415">Seldon Core 2</th></tr></thead><tbody><tr><td>The features of Seldon Core will not function without the Istio ingress controller.</td><td>Seldon Core 2 is by design service mesh-agnostic and as a result you can choose any ingress controller you want. However, If you choose a non-Istio ingress controller it may restrict you to the more basic or limited functionality of Seldon Core 2.</td></tr><tr><td>Seldon Core uses Istio because it relies on a service mesh for traffic splitting and leverages Knative for drift and outlier detection, as well as inference event logging.</td><td>Seldon Core 2 offers features such as  <a href="https://www.seldon.io/news/seldon-deploy-advanced-released">multi-model serving and overcommit functionality</a> and uses Kafka as a message queue for <a href="https://www.seldon.io/news/seldon-deploy-advanced-released">data-centric pipelines</a>. These features work best with Istio ingress controller.</td></tr></tbody></table>

### Features of Istio ingress controller  <a href="#ingress-controller-comparison" id="ingress-controller-comparison"></a>

* Service mesh with advanced traffic management capabilities.
* Required for effective use of Seldon Core.
* Automatic configuration of ingress routes for Seldon Core deployments.
* Enables authorization of Seldon Core deployments.
* Manages traffic for rollout strategies, drift and outlier detection, and inference data logging for Seldon Core.
* Supports both HTTP/1 and HTTP/2 (gRPC) traffic.
* Provides TLS support for HTTPS and gRPCS.
* Supports ingress routes for  ML deployments.
