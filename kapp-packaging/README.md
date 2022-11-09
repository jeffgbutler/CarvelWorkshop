# Packaging for the Kapp Controller

We have seen how we can use the kapp controller to manage applications on Kubernetes. This is useful, but it
is only the beginning of what's possible with the kapp controller. A more powerful capability is the capability
to package and publish software applications in such a way that they are easy to install and upgrade on
any cluster. This is a key capability of the kapp controller and it forms the basis for software delivery in
all of the Tanzu products. In this sections, we'll take a quick look at how this works.

First, we'll say a few words about two other Carvel tools that we haven't touched yet: vendir, and imgpkg.

## About vendir

The Carvel tool vendir is a general purpose file downloader and directory composer. You create a YAML manifest
to describe the desired contents of a directory structure and where the contents come from. Then vendir
syncs the directory to your desired state. One nice thing about vendir is that it understands how to
retrieve content from many sources - from Git repos, GitHub release, image repositories, and plain old
directories. With vendir you can describe content from a wide variety of sources and not worry about using different
tools to retrieve content from different sources.

When we build software packages for the kapp controller, we will use vendir under the covers to retrieve the contents
of the packages.

## About imgpkg

The Carvel tool imgpkg is used to build OCI images from arbitrary content. With kapp packging, we will
publish an image containing the configuration of an application - and imgpkg will be used under the covers
to create the image.

## Building Application Packages

There are different ways to build software packages for the Kaapp Controller:

1. You can hand code all the YAML yourself. This exposes the full power of Carvel packaging and
   is an appropriate strategy for fully supported software releases
1. You can use the "kctrl" CLI to create a simple package. This is a simpler way to get started
   and it is what we will use in this workshop

You can read all about Carvel software packaing here: https://carvel.dev/kapp-controller/docs/v0.42.0/packaging/

## Building a Package for Kuard

The [kuard-app](kuard-app/) directory in this repo contains YTT templates for deploying Kuard on a cluster.
It includes a namespace, deployment, service, and ingress as we have seen in many other exercises in this workshop.
It also contains a `schema.yaml` file that defines the variables available in this application, namely
the target namespace and base domain name for the application.

Let's use "kctrl" to build a software package. To start, open a terminal window in the same directory as this
README file.

Make a directory structure for the workspace...
```shell
mkdir -p packaging-exercise/kuard-app
```

Change into the new directory...
```shell
cd packaging-exercise/kuard-app
```

Now we wil start the package creation utility. Remember that our source files are on GitHub in the 
repo https://github.com/jeffgbutler/CarvelWorkshop/ and in directory `kapp-packaging/kuard-app`.

```shell
kctrl package init
```

This starts a small UI that will ask some basic questions about our app. Here is what I used:

| Wizard Item | Answer |
|---|---|
|Package reference name| kuard.jeffgbutler.github.io |
|Content Location | Git Repository (#4) |
|Git URL | https://github.com/jeffgbutler/CarvelWorkshop |
|Git reference | origin/main |
|Paths | kapp-packaging/kuard-app/*.yaml |

Once these values are entered, the wizard completes and builds a directory containing
the downloaded application configuration files (in the "upstream" directory), as well
some other configuration files for the software package.

Notice that there is now a `vendir.yml` file in your working directory - this is a configuration
file for the Carvel "vendir" tool we mentioned earlier. You can take a look at it to see how
vendir is configured.

## Publish the Package

The next step is to publish the package to an image repository. I will use Docker Hub for the exercise,
but any repository is supported.

First, make sure you are logged in to your repository...
```shell
docker login
```

Start the package release wizard...
```shell
kctrl package release --version 1.0.0
```

Enter the name of the image you wish to publish. I used "index.docker.io/jeffgbutler/kuard-app"

The wizard will run. You can see in the output that there are three steps to the publish: ytt, kbld,
and imgpkg. Of these, imgpkg is the most interesting for now - you can see the files it includes in the
published image.

## Install the Package in a Cluster

The package is now published, we need to make it available in our cluster. First, make a namespace for
the package definitions:

```shell
kubectl create ns custom-packages
```

Now deploy the package to our cluster. Note that the package is deployed on Dockerhub, but the package
definition is still only on our local machine:
```shell
kapp deploy -a kuard-app -f carvel-artifacts/packages/kuard.jeffgbutler.github.io -n custom-packages
```

You can see the packages available on your cluster with the following commands (both kctrl and tanzu
versions are shown):
```shell
kctrl package available list -A
```

```shell
tanzu package available list -A
```

You can dee details about a package with the following commands:
```shell
kctrl package available get -p kuard.jeffgbutler.github.io -n custom-packages
```

```shell
tanzu package available list kuard.jeffgbutler.github.io -n custom-packages
```

```shell
tanzu package available get kuard.jeffgbutler.github.io -n custom-packages
```

You can see what variables are available in the package with this command - you can see the three variables
we defined in our ytt schema:
```shell
kctrl package available get -p kuard.jeffgbutler.github.io/1.0.0 -n custom-packages --values-schema
```

```shell
tanzu package available get kuard.jeffgbutler.github.io/1.0.0 -n custom-packages --values-schema
```

## Install the Application

Once the package is available in the cluster, we can install an instance with a command like the following
(this will install the application with default values for all the variables - namely the app will be in the
"kuard-app" namespace):

```shell
kctrl package install -i simple-kuard -p kuard.jeffgbutler.github.io --version 1.0.0 -n custom-packages
```

```shell
tanzu package install simple-kuard --package-name kuard.jeffgbutler.github.io --version 1.0.0 -n custom-packages
```

You should see the application at this URL: http://kuard.kuard-app.127-0-0-1.nip.io/

You can delete the application with the following command:

```shell
kctrl package installed delete -i simple-kuard -n custom-packages
```

```shell
tanzu package installed delete simple-kuard -n custom-packages
```

If you want to override any of the default variable names, you can create a values.yaml file and supply it when
creating the application. Assume a values.yml file like this:

```yaml
namespace: jgb-ns
```

You can pass the file when creating the application like this:
```shell
kctrl package install -i jgb-kuard -p kuard.jeffgbutler.github.io --version 1.0.0 -n custom-packages --values-file values.yaml
```

Now the application is in a different namespace ("jgb-ns") and the URL has also changed: http://kuard.jgb-ns.127-0-0-1.nip.io/
