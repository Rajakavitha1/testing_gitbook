---
description: >-
  Learn about installing Istio ingress controller in a Kubernetes cluster
  running Seldon Enterprise Platform.
---

# Istio

Istio implements the Kubernetes ingress resource to expose a service and make it accessible from outside the cluster. You can install Istio in either a self-hosted Kubernetes cluster or a managed Kubernetes service provided by a cloud provider that is running the Seldon Enterprise Platform.

## Prerequisites

* Install [Seldon Enterprise Platform](../seldon-enterprise-platform.md).

## Installing Istio ingress controller

* [ ] Install Istio

1. Download the Istio installation package for the version you want to use. In the following command replace `<version>` with the version of Istio that you downloaded:

    ```
    curl -L https://istio.io/downloadIstio | sh -
    cd istio-<version>
    export PATH=$PWD/bin:$PATH
    ```
1. Install the Istio Custom Resource Definitions (CRDs) and Istio components in your cluster using the `istioctl`:
    ```
    ```
1. Create a namespace where you want to enable Istio automatic sidecar injection. For example in the namespace `istio-system`:
    ```
    ```

