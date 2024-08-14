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
    istioctl install --set profile=default -y
    ```
1. Create a namespace where you want to enable Istio automatic sidecar injection. For example in the namespace `istio-system`:
    ```
    kubectl label namespace istio-system istio-injection=enabled
    ```
* [ ] Install Istio Ingress Gateway

1. Verify that Istio Ingress Gateway is installed:
   ```
   kubectl get svc istio-ingressgateway -n istio-system
   ```
   This should return details of the Istio Ingress Gateway, including the external IP address.

1. Create a YAML file to specify Gateway resource in the `istio-system` namespace to expose your application. For example, create the `istio-seldon-gateway.yaml` file. Use your preferred text editor to create and save the file with the following content:
   ```
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: my-gateway
      namespace: istio-system
    spec:
      selector:
        istio: ingressgateway # Use Istio's default ingress gateway
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"
    ```
1. Apply the configuration:
   ```
   kubectl apply -f istio-seldon-gateway.yaml
   ```
   When the configuration is applied, you should see this:
   ```
   gateway.networking.istio.io/seldon-gateway created
   ```

1. Find the IP address of the Seldon Enterprise Platform instance running with Istio:
    ```
    ISTIO_INGRESS=$(kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    ISTIO_INGRESS+=$(kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

    echo "Seldon Enterprise Platform: http://$ISTIO_INGRESS/seldon-deploy/"

    ```


