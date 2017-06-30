# Kubernetes The Hard Way - Microsoft Azure Edition
#### A guide based on *kelseyhightower's* tutorial for Kubernetes on GCP and Azure specifics steps by *lostintangent*

This tutorial will walk you through setting up Kubernetes the hard way. This guide is not for people looking for a fully automated command to bring up a Kubernetes cluster. If that's you then check out [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/), or the [Getting Started Guides](http://kubernetes.io/docs/getting-started-guides/).

This tutorial is optimized for learning, which means taking the long route to help people understand each task required to bootstrap a Kubernetes cluster. This tutorial requires access to [Microsoft Azure](azure.microsoft.com).

> The results of this tutorial should not be viewed as production ready, and may receive limited support from the community, but don't let that prevent you from learning!

## Target Audience

The target audience for this tutorial is someone planning to support a production Kubernetes cluster and wants to understand how everything fits together. After completing this tutorial I encourage you to automate away the manual steps presented in this guide.

## Cluster Details

* Kubernetes 1.7.0
* Docker 17.05.0-CE
* etcd 3.2.1
* [CNI Based Networking](https://github.com/containernetworking/cni)
* Secure communication between all components (etcd, control plane, workers)
* Default Service Account and Secrets
* [RBAC authorization enabled](https://kubernetes.io/docs/admin/authorization)
* DNS add-on

### What's Missing

The resulting cluster will be missing the following features:

* Cloud Provider Integration
* [Logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
* [Cluster add-ons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)

## Labs

This tutorial assumes you have access to [Microsoft Azure](azure.microsoft.com) and the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). While Azure is used for basic infrastructure needs the things learned in this tutorial can be applied to every platform.

* [Cloud Infrastructure Provisioning](docs/01-infrastructure-azure.md)
* [Setting up a CA and TLS Cert Generation](docs/02-certificate-authority.md)
* [Setting up RBAC Authentication](docs/03-auth-configs.md)
* [Bootstrapping a H/A etcd cluster](docs/04-etcd.md)
* [Bootstrapping a H/A Kubernetes Control Plane](docs/05-kubernetes-controller.md)
* [Bootstrapping Kubernetes Workers](docs/06-kubernetes-worker.md)
* [Configuring the Kubernetes Client - Remote Access](docs/07-kubectl.md)
* [Managing the Container Network Routes](docs/08-network.md)
* [Deploying the Cluster DNS Add-on](docs/09-dns-addon.md)
* [Smoke Test](docs/10-smoke-test.md)
* [Cleaning Up](docs/11-cleanup.md)
