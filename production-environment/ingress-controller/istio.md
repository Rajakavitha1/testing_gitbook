# Istio

Istio implements the Kubernetes ingress resource to expose a service and make it accessible from outside the cluster. You can install Istio in either a self-hosted Kubernetes cluster or a managed Kubernetes service provided by a cloud provider that is running the Seldon Enterprise Platform. 

## Prerequisites
* Install Seldon Enterprise Platform.
* Download and  Install istioctl, the Istio configuration command line utility tool.

## Installing Istio ingress controller
1. Install Istio
    a. Download the Istio installation package for the version you want to use. In the following command replace `<version>` with the version of Istio that you downloaded:
     ```curl -L https://istio.io/downloadIstio | sh -
        cd istio-<version>
        export PATH=$PWD/bin:$PATH
     ```
    b. Install the Istio Custom Resource Definitions (CRDs) and Istio components in your cluster using the `istioctl`:
     ```istioctl install --set profile=default -y
     ```
    c. Create a namespace where you want to enable Istio automatic sidecar injection. For example in the namespace `istio-system`:
     ```kubectl label namespace istio-system istio-injection=enabled       

