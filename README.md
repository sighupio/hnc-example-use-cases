# HNC *(Hierarchical Namespace Controller)* - Example use cases

The repository contains some examples of HNC use cases.

## What is HNC?

> Hierarchical namespaces make it easier to share your cluster by making namespaces more powerful. For example, you can
> create additional namespaces under your team's namespace, even if you don't have cluster-level permission to create
> namespaces, and easily apply policies like RBAC and Network Policies across all namespaces in your team
> (e.g. a set of related microservices).

*Source: [multi-tenancy sig repository](https://github.com/kubernetes-sigs/multi-tenancy/tree/hnc-v0.5.1/incubator/hnc)*

## Requirements

You will need:

- [docker](https://www.docker.com/).
- [kind](https://github.com/kubernetes-sigs/kind/releases/tag/v0.8.1): Kubernetes in docker
  - `kubectl`.
- OS tools like `curl`, `make`, `git`.
- [direnv](https://direnv.net/): direnv is an extension for your shell. It augments existing shells with a new feature
that can load and unload environment variables depending on the current directory.
- golang

There are two ways to spin up the HNC, if you are running linux:

```bash
$ kind create cluster
# It can take a couple of minutes
$ HNC_VERSION=v0.5.1
# Latest stable release
$ kubectl apply -f https://github.com/kubernetes-sigs/multi-tenancy/releases/download/hnc-${HNC_VERSION}/hnc-manager.yaml
$ curl -L https://github.com/kubernetes-sigs/multi-tenancy/releases/download/hnc-${HNC_VERSION}/kubectl-hns -o ./kubectl-hns
$ chmod +x ./kubectl-hns
$ export PATH=${PWD}:${PATH}
```

In the other hand while running MacOS you have to compile your own `hns` kubectl plugin.
*(It is also compatible while running linux)*

```bash
$ HNC_VERSION=v0.5.1
# Latest stable release
$ git clone --depth 1 --branch hnc-${HNC_VERSION} git@github.com:kubernetes-sigs/multi-tenancy.git
$ cd multi-tenancy/incubator/hnc
$ mkdir -p ../../tmp/go/bin/ ../../tmp/go/pkg/ # Fix an issue while compiling the plugin
$ direnv allow .
$ source devenv
$ make kind-reset kind-deploy
# It can take a couple of minutes to finish it
```

Test the setup:

```bash
$ kubectl get ns hnc-system
NAME         STATUS   AGE
hnc-system   Active   30s
$ kubectl get deploy,pods -n hnc-system
NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hnc-controller-manager   1/1     1            1           50s

NAME                                          READY   STATUS    RESTARTS   AGE
pod/hnc-controller-manager-58888d54cb-5zhvl   2/2     Running   0          50s
$ kubectl hns
Manipulates hierarchical namespaces provided by HNC

Usage:
  kubectl-hns [command]

Available Commands:
  config      Manipulates the HNC configuration
  create      Creates a subnamespace under the given parent.
  describe    Displays information about the hierarchy configuration
  help        Help about any command
  set         Sets hierarchical properties of the given namespace
  tree        Display one or more hierarchy trees
```

### Cluster-Admin

I've seen some issues with the pre-configured permissions given to the controller.
So I've opted to grant cluster-admin to the service account assigned to this controller.

```bash
$ kubectl create clusterrolebinding hnc-cluster-admin --clusterrole=cluster-admin  --serviceaccount=hnc-system:default
```

This should be improved in the near future.

## Use cases

### Namespace Self Provisioning

One of the use cases this controller enables is the ability to define namespace templates as a
cluster administrator, users does not need to have any cluster level permissions to provision a new ready to use
namespace.

[Click here to continue reading about this use case](use-cases/self-provision)

### Application templates

Imagine you are building an application with a common pattern:

- Frontend
- Backend
- DB

If you are involved in the development of a new **frontend** feature, you could be benefict of having a stable backend
templated to being able to test your changes in a stable and isolated environment *(reproducible)*.

[Click here to continue reading about this use case](use-cases/application-template)

## License

[License :bookmark_tabs:](LICENSE)

OSS baked with :heart: at [SIGHUP](https://sighup.io).

