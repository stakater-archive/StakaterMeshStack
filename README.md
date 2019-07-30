# Stakater Mesh Stack

## Overview
This document provide guideline regarding the installation and usage of [Istio](https://istio.io/) on kubernetes cluster for creating service mesh.

## Installation

This section provides step by step guideline regarding istio installation.

* Create a namespace using the manifest given below in which istio and its dependencies will be installed:

```json
{ 
  "kind": "Namespace", 
  "apiVersion": "v1", 
  "metadata": { 
    "name": "mesh", 
    "labels": { 
      "name": "mesh" 
    }
  }
}
```

Create the namespace using the command given below:
```bash
$ sudo kubectl apply -f namespace-creation-manifest.yaml
```


* Istio can be installed in two ways:

  * `Method 1`: Follow the guidelines given in this [link](https://istio.io/docs/setup/kubernetes/install/kubernetes/) to install istio using kubernetes manifests.

  * `Method 2`: By using HelmRelease, its manifest is given below:

```yaml
apiVersion: flux.weave.works/v1beta1
kind: HelmRelease
metadata:
  name: istio
  namespace: mesh
spec:
  releaseName: istio
  chart:
    repository: https://gcsweb.istio.io/gcs/istio-prerelease/daily-build/release-1.1-latest-daily/charts/
    name: istio
    version: 1.1.0
  values:
    global:
      enableTracing: false
    tracing:
      enabled: false
    pilot:
      traceSampling: 100
    prometheus:
      enabled: false
```

Currently, this [chart](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio) is being used for Istio deployment.






## Notes
This section explain the problems that has been faced during installation, deployment and usage of Istio.

### Nginx-ingress Controller

* Initially, we tried to use nginx-ingress controller to enable ingress for istio services(like jaeger) but the problem was with the traces because the requests done through nginx-ingress was not able to generate traces and no way was found on how to configure nginx-ingress controller with Istio. 

  Although, we were able to get traces by using Istio ingress gateway on jaeger.

### Proxy(sidecar) Container Injection
There are two ways to inject sidecar(proxy) container:

* To manually inject sidecar use the instruction given on this [link](https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/#manual-sidecar-injection).  

* Istio by default monitors all namespaces but it only add sidecars(envoy) to those namespaces that have this label `istio-injection:enabled` assigned. To assign this label to a namespace use the command given below:

#### Disable sidecar injection in a pod
* By default istio inserts sidecar containers in each pods(only if namespace has `istio-injection: enabled`) automatically, to disable sidecar injection in a pod add this annotation `sidecarInjectorWebhook.enabled: "false"` to pod annotations of a deployment.

```bash
$ sudo kubectl label namespace <namespace-name> istio-injection=enabled
```

### CRD deletion issue

* Sometimes due to ungraceful deletion of istio helm release CRDs will not be removed. There are two ways to delete remaining CRDs. 

  * `Method-1`: Use the command given below to get all the CRDs and delete them one by one:
  ```bash
  $ sudo kubectl get crd

  $ sudo kubectl delete crd <crd-name>
  ```

  * `Method-2`: In this method we will use the manifest for crd creation to delete all the corresponding CRDs. First of all down the istio release. Move inside this folder (istio-X/install/kubernetes/helm/istio-init/files/) and run the command given below on each file:
  ```bash
  $ sudo kubectl delete -f <filename>.yaml
  ```

