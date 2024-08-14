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

1.  Download the Istio installation package for the version you want to use. In the following command replace `<version>` with the version of Istio that you downloaded:

    ```
    curl -L https://istio.io/downloadIstio | sh -
    cd istio-<version>
    export PATH=$PWD/bin:$PATH
    ```
2.  Install the Istio Custom Resource Definitions (CRDs) and Istio components in your cluster using the `istioctl`:

    ```
    istioctl install --set profile=default -y
    ```
3.  Create a namespace where you want to enable Istio automatic sidecar injection. For example in the namespace `istio-system`:

    ```
    kubectl label namespace istio-system istio-injection=enabled
    ```

* [ ] Install Istio Ingress Gateway

1.  Verify that Istio Ingress Gateway is installed:

    ```
    kubectl get svc istio-ingressgateway -n istio-system
    ```

    This should return details of the Istio Ingress Gateway, including the external IP address.
2.  Create a YAML file to specify Gateway resource in the `istio-system` namespace to expose your application. For example, create the `istio-seldon-gateway.yaml` file. Use your preferred text editor to create and save the file with the following content:

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
3.  Apply the configuration:

    ```
    kubectl apply -f istio-seldon-gateway.yaml
    ```

    When the configuration is applied, you should see this:

    ```
    gateway.networking.istio.io/seldon-gateway created
    ```
4.  Find the IP address of the Seldon Enterprise Platform instance running with Istio:

    ```
    ISTIO_INGRESS=$(kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    ISTIO_INGRESS+=$(kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

    echo "Seldon Enterprise Platform: http://$ISTIO_INGRESS/seldon-deploy/"

    ```

    \
    Make a note of the IP address that is displayed in the output.

* [ ] Install Istio Ingress Gateway

1. Update the configurations in the `install-values.yaml` file that you created when installing Seldon Enterprise.

{% hint style="info" %}
Ensure that you replace `<ip_address>` with the IP address that you made a note of when installing ingress gateway.
{% endhint %}

&#x20;      Use your preferred text editor to create and save the file with the following content:

1.  Check the status of the installation seldon-enterprise-seldon-deploy.

    ```
    kubectl rollout status deployment/seldon-enterprise-seldon-deploy -n seldon-system
    ```

    When the installation is complete you should see this:

    ```
    deployment "seldon-enterprise-seldon-deploy" successfully rolled out
    ```
2.  Get the Pod that is running Seldon Enterprise Platform in the cluster and save it as `$POD_NAME`.

    ```
    export POD_NAME=$(kubectl get pods --namespace seldon-system -l "app.kubernetes.io/name=seldon-deploy,app.kubernetes.io/instance=seldon-enterprise" -o jsonpath="{.items[0].metadata.name}")
    ```
3.  You can use port-forwarding to access your application locally.

    ```
    kubectl port-forward $POD_NAME 8000:8000 --namespace seldon-system
    ```
4. Open your browser and navigate to http://127.0.0.1:8000/seldon-deploy/ to access Seldon Enterprise Platform.
