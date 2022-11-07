# Secretgen-Controller Overview

The secretgen-controller is part of the [Carvel](https://carvel.dev/) tool suite. It is pre-installed on all
Tanzu clusters, and can be installed on any other Kubernetes cluster.

The secretgen-controller does a few things:

1. It can generate secrets, keys, and certificates for use by other Kubernetes resources (we will not cover that usage in this workshop)
2. It provides an operator makes it easy to share secrets across Kubernetes namespaces (we will cover this usage)
3. As of version 0.9.0, it includes a CRD `SecretTemplate` that can be used to create secrets with arbitrary fields, and
   secrets composed of other secrets. There are interesting use cases associated with this facility related to service
   binding. We will not cover that use in this workshop, but you can read a good blog post about it here:
   https://tanzu.vmware.com/developer/guides/tanzu-service-secret-sauce/

The Tanzu CLI includes a function that will create a registry secret. The Tanzu CLI secret functionality
requires the secretgen-controller to be installed!

Full details about the secretgen-controller are here: https://github.com/vmware-tanzu/carvel-secretgen-controller

**Important:** We will not use the secretgen-controller in this workshop, so you can safely skip this section. Using the
secretgen-controller is useful if you want to set up different namespaces for different developers.

## Installing the Secretgen-Controller

The Secretgen-Controller is pre-installed in all Tanzu clusters. You can verify the installation in a few ways:

1. `tanzu package installed list -A` should show the secretgen-controller package installed (likely in the tkg-system namespace)
1. `kapp list -A` should show an application named "secregen-controller-ctrl"
1. `kctrl app list -A` should show an application named "secregen-controller"
1. `kubectl get all -n secretgen-controller` should will show several items installed if the secretgen-controller was installed
   manually with the defaults

(If you are not familiar with the "kapp" and "kctrl" commands, don't worry - we will cover them later)

If the secretgen-controller is not already installed in your cluster, you can easily install it with one of the following commands:

```shell
kapp deploy -a secretgen-controller -f https://github.com/vmware-tanzu/carvel-secretgen-controller/releases/latest/download/release.yml
```

```shell
kubectl create -f https://github.com/vmware-tanzu/carvel-secretgen-controller/releases/latest/download/release.yml
```

Once the system reconciles, you should see items in the secretgen-controller namespace:

```shell
kubectl get all -n secretgen-controller
```

## Basics of secretgen-controller

The secretgen-controller adds several CRDs to your cluster. For our purposes, we are interested in two of them:
`SecretExport` and `SecretImport`.

A `SecretExport` is used to offer secrets in one namespace to other or all namespaces. For example, suppose
we create the following secret in the default namespace:

```shell
kubectl create secret docker-registry some-secret \
  --docker-server=harbor.tanzuathome.net \
  --docker-username=admin \
  --docker-password=Harbor12345
```

We can create a `SecretExport` to make the secret available to all other namespaces:

```yaml
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretExport
metadata:
  name: some-secret
spec:
  toNamespace: '*' # we can also specify a specific namespace, or a list of namespaces
```

At this point nothing has been shared. To share the secret we create a `SecretImport` in another namespace. For
example, create a namespace called `ns2`:

```shell
kubectl create ns ns2
```

Then import the secret with the following:

```yaml
apiVersion: secretgen.carvel.dev/v1alpha1
kind: SecretImport
metadata:
  name: some-secret
  namespace: ns2
spec:
  fromNamespace: default
```

When the `SecretImport` in `ns2` reconciles, there will also be a copy of the secret from the `default` namespace.
If the secret changes in the `default` namespace, then the secret in `ns2` will be updated by the operator. If you delete
the `SecretImport` in `ns2`, the corresponding secret will also be deleted. If you delete the secret in the `default` namespace,
the secret in the `ns2` namespace will also be deleted (the `SecretExport` and `SecretImport` will remain).

## Secrets and the Tanzu CLI

The Tanzu CLI is also capable of creating secrets when the secretgen-controller is installed in a cluster.

The following command will create both the `Secret`, and the `SecretExport` in a single command:

```shell
tanzu secret registry add another-secret \
  --server harbor.tanzuathome.net \
  --username=admin \
  --password=Harbor12345 \
  --export-to-all-namespaces
```

You can see the `SecretExport` with this command:

```shell
kubectl get SecretExports.secretgen.carvel.dev
```

You will still need to create the `SecretImport` in the other namespaces using regular kubectl commands as shown above.

If you delete the secret with the Tanzu CLI, the secret export will also be deleted.

[Next (YTT Overview) -&gt;](../ytt/README.md)
