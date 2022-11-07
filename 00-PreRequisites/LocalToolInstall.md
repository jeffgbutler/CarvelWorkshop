# Install Carvel Tools

This workshop requires Docker and several other tools. Using package manager (Homebrew or Chocolatey) makes it
easier to install some components, but is not strictly required. We will only write instructions based on
package managers.

You will need to have Internet access to run the exercises in the workshop.

To complete the pre-requisites, follow the platform specific initial setup instructions below for your platform:
   [MacOS](#install-prerequisites---macos), [Windows](#install-prerequisites---windows), or [Linux](#install-prerequisites---linux)

## Install Prerequisites - MacOS

Install required tools:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Homebrew: https://brew.sh/
- Kind
   ```shell
   brew install kind
   ```
- Kubernetes CLI:
   ```shell
   brew install kubernetes-cli
   ```
- JQ (JSON Formatter):
   ```shell
   brew install jq
   ```
- Carvel Tools:
   ```shell
   brew tap vmware-tanzu/carvel
   brew install kapp kbld kctrl imgpkg vendir ytt
   ```
- Kubectl Krew Plugin Manager (Optional, but helpful):
   ```shell
   brew install krew
   ```
- Kubectl tree plugin (Optional, but helpful. Requires Krew plugin manager from previous step):
   ```shell
   kubectl krew install tree
   ```

Next step: [Configure a Kind Cluster](KindClusterConfiguration.md)

## Install Prerequisites - Windows

Install required tools:

- Docker Desktop: https://www.docker.com/products/docker-desktop/
- Chocolatey: https://chocolatey.org/install
- Kind
   ```shell
   choco install kind
   ```
- Kubernetes CLI:
   ```shell
   choco install kubernetes-cli
   ```
- JQ (JSON Formatter):
   ```shell
   choco install jq
   ```
- Carvel Tools:
   ```shell
   choco install kapp ytt
   ```
- Carvel Kbld: Go to https://github.com/vmware-tanzu/carvel-kbld/releases, download latest Windows binary, rename it to "kbld.exe", add it to the path
- Carvel Kctrl: https://github.com/vmware-tanzu/carvel-kapp-controller/releases, download latest Windows binary, rename it to "kctrl.exe", add it to the path
- Carvel Imgpkg: Go to https://github.com/vmware-tanzu/carvel-imgpkg/releases, download latest Windows binary, rename it to "imgpkg.exe", add it to the path
- Carvel Vendir: Go to https://github.com/vmware-tanzu/carvel-vendir/releases, download latest Windows binary, rename it to "vendir.exe", add it to the path
- Kubectl Krew Plugin Manager (Optional, but helpful): https://krew.sigs.k8s.io/docs/user-guide/setup/install/
- Kubectl tree plugin (Optional, but helpful. Requires Krew plugin manager from previous step):
   ```shell
   kubectl krew install tree
   ```

Next step: [Configure a Kind Cluster](KindClusterConfiguration.md)

## Install Prerequisites - Linux

Install required tools:

- Docker Engine: https://docs.docker.com/engine/install/
- Homebrew: https://brew.sh/
- Kind
   ```shell
   brew install kind
   ```
- Kubernetes CLI:
   ```shell
   brew install kubernetes-cli
   ```
- JQ (JSON Formatter):
   ```shell
   brew install jq
   ```
- Carvel Tools:
   ```shell
   brew tap vmware-tanzu/carvel
   brew install kapp kbld kctrl imgpkg vendir ytt
   ```
- Kubectl Krew Plugin Manager (Optional, but helpful):
   ```shell
   brew install krew
   ```
- Kubectl tree plugin (Optional, but helpful. Requires Krew plugin manager from previous step):
   ```shell
   kubectl krew install tree
   ```

Next step: [Configure a Kind Cluster](KindClusterConfiguration.md)
