<!-- markdownlint-disable MD041 -->
<img src="https://camo.githubusercontent.com/5b298bf6b0596795602bd771c5bddbb963e83e0f/68747470733a2f2f692e696d6775722e636f6d2f7031527a586a512e706e67" align="left" width="144px" height="144px"/>

# Home Kubernetes cluster managed by GitOps

<br/>
<br/>

<div align="center">

[![Discord](https://img.shields.io/discord/673534664354430999?style=for-the-badge&label=discord&logo=discord&logoColor=white&color=teal)](https://discord.gg/k8s-at-home)
[![k3s](https://img.shields.io/badge/k3s-v1.24.2-blue?style=for-the-badge&logo=kubernetes&logoColor=white)](https://k3s.io/)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled?logo=pre-commit&logoColor=white&style=for-the-badge&color=brightgreen)](https://github.com/pre-commit/pre-commit)
[![renovate](https://img.shields.io/badge/renovate-enabled?style=for-the-badge&logo=renovatebot&logoColor=white&color=brightgreen)](https://github.com/renovatebot/renovate)

</div>

---

[k3s](https://k3s.io) cluster deployed with [Ansible](https://www.ansible.com), backed by [Flux](https://toolkit.fluxcd.io/) and [SOPS](https://toolkit.fluxcd.io/guides/mozilla-sops/).

## Components

- [flux](https://toolkit.fluxcd.io/) - GitOps operator for managing Kubernetes clusters from a Git repository
- [kube-vip](https://kube-vip.io/) - Load balancer for the Kubernetes control plane nodes
- [metallb](https://metallb.universe.tf/) - Load balancer for Kubernetes services
- [cert-manager](https://cert-manager.io/) - Operator to request SSL certificates and store them as Kubernetes resources
- [calico](https://www.tigera.io/project-calico/) - Container networking interface for inter pod and service networking
- [external-dns](https://github.com/kubernetes-sigs/external-dns) - Operator to publish DNS records to Cloudflare (and other providers) based on Kubernetes ingresses
- [k8s_gateway](https://github.com/ori-edge/k8s_gateway) - DNS resolver that provides local DNS to the Kubernetes ingresses
- [traefik](https://traefik.io) - Kubernetes ingress controller used for a HTTP reverse proxy of Kubernetes ingresses
- [local-path-provisioner](https://github.com/rancher/local-path-provisioner) - provision persistent local storage with Kubernetes

_Additional applications include [hajimari](https://github.com/toboshii/hajimari), [error-pages](https://github.com/tarampampam/error-pages), [echo-server](https://github.com/Ealenn/Echo-Server), [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller), [reflector](https://github.com/emberstack/kubernetes-reflector), [reloader](https://github.com/stakater/Reloader), and [kured](https://github.com/weaveworks/kured)_

Provisioning is performed using [Ansible](https://www.ansible.com).

## 📂 Repository structure

The Git repository contains the following directories under `cluster` and are ordered below by how Flux will apply them.

```sh
📁 cluster      # k8s cluster defined as code
├─📁 flux       # flux, gitops operator, loaded before everything
├─📁 crds       # custom resources, loaded before 📁 core and 📁 apps
├─📁 charts     # helm repos, loaded before 📁 core and 📁 apps
├─📁 config     # cluster config, loaded before 📁 core and 📁 apps
├─📁 core       # crucial apps, namespaced dir tree, loaded before 📁 apps
└─📁 apps       # regular apps, namespaced dir tree, loaded last
```

## 🚀 Installation

### 🔐 Setting up Age

Create a Age Private and Public key. Using [SOPS](https://github.com/mozilla/sops) with [Age](https://github.com/FiloSottile/age) allows us to encrypt secrets and use them in Ansible and Flux.

1. Create a Age Private / Public Key

    ```sh
    age-keygen -o age.agekey
    ```

2. Set up the directory for the Age key and move the Age file to it

    ```sh
    mkdir -p ~/.config/sops/age
    mv age.agekey ~/.config/sops/age/keys.txt
    ```

3. Export the `SOPS_AGE_KEY_FILE` variable in `bashrc`, `zshrc` or `config.fish` and source it, e.g.

    ```sh
    export SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt
    source ~/.bashrc
    ```

4. Fill out the Age public key in the `.config.env` under `BOOTSTRAP_AGE_PUBLIC_KEY`, **note** the public key should start with `age`

### 📄 Configuration

The `.config.env` file contains necessary configuration that is needed by Ansible and Flux.

1. Copy the `.config.sample.env` to `.config.env` and start filling out all the environment variables:

    **All are required** unless otherwise noted in the comments.

    ```sh
    cp .config.sample.env .config.env
    ```

2. Once that is done, verify the configuration is correct by running:

    ```sh
    task verify
    ```

3. If you do not encounter any errors, wire up the templated files and place them where they need to be.

    ```sh
    task configure
    ```

### ⚡ Preparing the host OS with Ansible

1. Ensure you are able to SSH into your nodes using a private SSH key **without a passphrase**.

2. Install the Ansible deps:

    ```sh
    task ansible:init
    ```

3. Verify Ansible can view the config:

    ```sh
    task ansible:list
    ```

4. Verify Ansible can ping the nodes:

    ```sh
    task ansible:ping
    ```

5. Run the Prepare Ansible playbook:

    ```sh
    task ansible:prepare
    ```

6. Reboot the nodes:

    ```sh
    task ansible:reboot
    ```

### ⛵ Installing k3s with Ansible

☢️ If you run into problems, you can run `task ansible:nuke` to destroy the k3s cluster and start over.

1. Verify Ansible can view your config:

    ```sh
    task ansible:list
    ```

2. Verify Ansible can ping your nodes:

    ```sh
    task ansible:ping
    ```

3. Install k3s with Ansible:

    ```sh
    task ansible:install
    ```

4. Verify the nodes are online:

    ```sh
    task cluster:nodes
    # NAME           STATUS   ROLES                       AGE     VERSION
    # k8s-0          Ready    control-plane,master      4d20h   v1.21.5+k3s1
    # k8s-1          Ready    worker                    4d20h   v1.21.5+k3s1
    ```

### 🔹 GitOps with Flux

1. Verify Flux can be installed:

    ```sh
    task cluster:verify
    # ► checking prerequisites
    # ✔ kubectl 1.21.5 >=1.18.0-0
    # ✔ Kubernetes 1.21.5+k3s1 >=1.16.0-0
    # ✔ prerequisites checks passed
    ```

2. Install Flux and sync the cluster to the Git repository:

    ```sh
    task cluster:install
    # namespace/flux-system configured
    # customresourcedefinition.apiextensions.k8s.io/alerts.notification.toolkit.fluxcd.io created
    ```

3. Verify Flux components are running in the cluster:

    ```sh
    task cluster:pods -- -n flux-system
    # NAME                                       READY   STATUS    RESTARTS   AGE
    # helm-controller-5bbd94c75-89sb4            1/1     Running   0          1h
    # kustomize-controller-7b67b6b77d-nqc67      1/1     Running   0          1h
    # notification-controller-7c46575844-k4bvr   1/1     Running   0          1h
    # source-controller-7d6875bcb4-zqw9f         1/1     Running   0          1h
    ```

### 🎤 Verification Steps

1. View the Flux Git Repositories:

    ```sh
    task cluster:gitrepositories
    ```

2. View the Flux kustomizations:

    ```sh
    task cluster:kustomizations
    ```

3. View all the Flux Helm Releases:

    ```sh
    task cluster:helmreleases
    ```

4. View all the Flux Helm Repositories:

    ```sh
    task cluster:helmrepositories
    ```

5. View all the Pods:

    ```sh
    task cluster:pods
    ```

6. View all the certificates and certificate requests:

    ```sh
    task cluster:certificates
    ```

7. View all the ingresses:

    ```sh
    task cluster:ingresses
    ```

All the commands above can be run with one task:

```sh
task cluster:resources
```
