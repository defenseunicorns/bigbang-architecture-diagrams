# Big Bang Deployment Flow

> ************
> **_NOTE:_** The digram represents the sequence of events when deploying Big Bang, and only the HAProxy and Logging Big Bang components are depicted in this diagram for brevity
> ************

## Customer Big Bang Deployment Repo (built from template)

* Link: [https://repo1.dso.mil/platform-one/big-bang/customers/template](https://repo1.dso.mil/platform-one/big-bang/customers/template)
* Supports defining multiple environments (ex: prod and dev) through the use of Kustomize

## **[1]** Kustomize merge of Big Bang Repo ("base") and Customer Big Bang Deployment Repo ("overlay") per Environment and Flux Custom Resource (CR) creation

* Supports defining multiple environments (ex: prod and dev) through the use of Kustomize
* From within the Customer Repo the Kustomize "base" is defined with a remote reference to the Big Bang Repo - [https://repo1.dso.mil/platform-one/big-bang/bigbang/-/blob/master/base/kustomization.yaml](https://repo1.dso.mil/platform-one/big-bang/bigbang/-/blob/master/base/kustomization.yaml)
* The Customer Repo defines a Kustomize "overlay" for each specific customer environment (ex: prod) - [https://repo1.dso.mil/platform-one/big-bang/customers/template/-/blob/main/prod/kustomization.yaml](https://repo1.dso.mil/platform-one/big-bang/customers/template/-/blob/main/prod/kustomization.yaml)
* The Kustomize "base" is merged with the "overlay" for an environment **[1a]**
* The resulting rendered Kustomize base+overlay is used to create the Big Bang Flux HelmRelease CR **[1b]** as well as the Big Bang Flux GitRepository CR **[1c]**
* The Big Bang Flux HelmRelease CR uses the Big Bang Flux GitRepository CR as a source **[1d]** which is kept in sync with the Big Bang git repo **[1e]**

## **[2]** Additional Flux Custom Resources (CRs) created by the Customer Big Bang Deployment Repo

* Link: [https://repo1.dso.mil/platform-one/big-bang/customers/template/-/blob/main/prod/bigbang.yaml](https://repo1.dso.mil/platform-one/big-bang/customers/template/-/blob/main/prod/bigbang.yaml)
* Per the repo definition, it creates a Flux Customer Big Bang Kustomization CR **[2a]**
* Also it creates a Flux Customer Big Bang GitRepository CR **[2b]**
* The Flux Customer Big Bang Kustomization uses the Flux Customer Big Bang GitRepository as a source **[2c]** that keeps in sync with the repo **[2d]**

## **[3]** Flux Big Bang HelmRelease CR creates the Big Bang Helm Release

* Link: [https://repo1.dso.mil/platform-one/big-bang/bigbang/base](https://repo1.dso.mil/platform-one/big-bang/bigbang/base)
* The Flux Big Bang HelmRelease CR renders and creates the Big Bang Helm Release **[3]**
* This implements an instance of the Big Bang Repo resources (informed by the values configured)

> ************
> **_NOTE:_** Code for the core flux components are maintained here as well
> ************

## **[4]** Big Bang Helm Release which orchestrates installation of the Big Bang components

* Link: [https://repo1.dso.mil/platform-one/big-bang/bigbang/-/tree/master/chart](https://repo1.dso.mil/platform-one/big-bang/bigbang/-/tree/master/chart)
* The Big Bang Helm Release is umbrella chart of Flux references to components (ex: HAProxy, Logging, etc) defined in other charts
* This Helm Release will create the following Flux CRs for each component
  * Flux HelmRelease CRs **[4a]**
  * Flux GitRepository CRs **[4b]** that reference git repositories **[4c]** containing each component (ex: HAProxy)
* The Flux HelmRelease CRs utilize the Flux GitRepository CRs as a source **[4d]** to create Flux HelmChart CRs **[4e]**

## **[5]** Big Bang Component Helm Releases

* Link: [https://repo1.dso.mil/platform-one/big-bang/bigbang/-/tree/master/chart/templates](https://repo1.dso.mil/platform-one/big-bang/bigbang/-/tree/master/chart/templates)
* The Flux HelmRelease CRs renders the Helm Charts and deploys them as Helm Releases **[5]**

## **[6]** Kubernetes Namespace Resources

* Helm Releases of each Big Bang component apply the Kubernetes resources contained in the rendered Helm Release **[6]**
  * ConfigMaps
  * Services
  * Starts up Kubernetes Pods as part of the spec (ReplicaSet, DaemonSet, StatefulSet, Deployment)

## **[7]** Container Images

* Pods defined in the Helm Releases will pulls the container images **[7]** required from the container registry
* Now the Big Bang workloads are finally running!
