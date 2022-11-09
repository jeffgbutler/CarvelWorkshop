# Create and Configure a Kind Cluster for the Workshop

Note: some of the commands below use the Carvel "kapp" command. We will cover the details about Kapp later in the workshop.

Create the cluster:
```shell
kind create cluster --config KindClusterConfig.yaml
```

Install the Carvel SecretGen controller:
```shell
kubectl apply -f https://github.com/vmware-tanzu/carvel-secretgen-controller/releases/latest/download/release.yml
```

Install the Carvel Kapp controller:
```shell
kubectl apply -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/latest/download/release.yml
```

Install Contour Ingress Controller
```shell
kubectl apply -f https://projectcontour.io/quickstart/contour.yaml
```

Install a test application:
```shell
kapp deploy -a kuard-test -f TestKuard.yaml
```

Make sure you can access Kuard at the following URL: http://kuard.kuard-test.127-0-0-1.nip.io/

Once you have verified access to Kuard, uninstall it with this command:

```shell
kapp delete -a kuard-test
```
