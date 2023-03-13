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
- Carvel Tools - some are supported by Chocolatey, but Chocolatey is not kept up to date so it is best to install manually. For each
  tool, download the latest Windows binary, rename it, and add it to your path. For simplicity, I recommend creating a folder
  named "\TanzuTools" and adding it to the path, then you can place all the downloaded files in a single directory.

   | Tool   |  Download Location                                            | File Name                | Rename To  |
   |--------|---------------------------------------------------------------|--------------------------|------------|
   | kapp   | https://github.com/carvel-dev/kapp/releases/latest/           | kapp-windows-amd64.exe   | kapp.exe   |
   | kbld   | https://github.com/carvel-dev/kbld/releases/latest            | kbld-windows-amd64.exe   | kbld.exe   |
   | kctrl  | https://github.com/carvel-dev/kapp-controller/releases/latest | kctrl-windows-amd64.exe  | kctrl.exe  |
   | imgpkg | https://github.com/carvel-dev/imgpkg/releases/latest          | imgpkg-windows-amd64.exe | imgpkg.exe |
   | vendir | https://github.com/carvel-dev/vendir/releases/latest          | vendir-windows-amd64.exe | vendir.exe |
   | ytt    | https://github.com/carvel-dev/ytt/releases/latest             | ytt-windows-amd64.exe    | ytt.exe    |

- Kubectl Krew Plugin Manager (Optional, but helpful):
   ```shell
   choco install krww
   ```
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
