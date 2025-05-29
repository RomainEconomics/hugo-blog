---
title: Kubernetes Setup
slug: kubernetes-setup
description: A guide to setting up Kubernetes with essential tools like kubectl, kubectx, and stern.
date: "2025-05-25"
tags: ["kubernetes", "setup", "devops", "cli"]
subjects: ["kubernetes", "devops"]
showTaxonomies: true
---

# Kubernetes Setup for Local Development

This post covers my go-to setup for working with Kubernetes from the command line. It’s a lightweight stack that helps me manage multiple clusters and namespaces efficiently, and debug faster using CLI tools.

Note that this setup is tailored for Linux and Ubuntu, but the tools can be adapted for other Linux distributions or macOS.

## Must-Have

### `kubectl`

The essential CLI tool to interact with any Kubernetes cluster. Almost every other tool you'll use is just a wrapper around this one.

To install `kubectl`, you can follow the [official guide](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/) or use the steps below:

```bash
# Download the binary
curl -LO https://dl.k8s.io/release/v1.30.0/bin/linux/amd64/kubectl

# Make it executable
chmod +x ./kubectl

# Move it to your PATH
sudo mv ./kubectl /usr/local/bin/kubectl
```

If the `curl` command fails, you can manually get the latest version using:

```bash
curl -Ls https://dl.k8s.io/release/stable.txt
```

Then plug the version into the download URL.

Verify the installation:

```bash
kubectl version --client
```

## Recommended Tools

### `kubectx` / `kubens`

When working with multiple clusters and namespaces, `kubectx` and `kubens` make switching between them seamless.

- `kubectx`: for switching between **contexts** (i.e., clusters).
- `kubens`: for switching between **namespaces**.

#### Installation

First, install the required dependency:

```bash
sudo apt install fzf
```

Then install the tools manually to avoid outdated versions:

```bash
# Clone the repo
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx

# Create symlinks
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

#### Configuring Cluster Credentials

Get your cluster credentials (typically YAML files) and place them into `~/.kube/`.

```bash
mkdir -p ~/.kube
mv ~/Downloads/CREDS1.yml ~/Downloads/CREDS2.yml ~/.kube/
```

Update your `~/.zshrc` to load these files:

```bash
export KUBECONFIG=~/.kube/CREDS1.yml:~/.kube/CREDS2.yml

# OR (if your files follow a pattern)
export KUBECONFIG=$KUBECONFIG:$(find ~/.kube/ -iname "kubeconfig*" | tr "\n" ":")
```

Reload your shell:

```bash
source ~/.zshrc
```

#### Helpful Aliases

Add these to your `~/.zshrc`:

```bash
alias k="kubectl"
alias kx="kubectx"
alias kn="kubens"
```

Now you can easily:

```bash
kx                # select cluster
kn                # select namespace
k get pods        # list pods in current namespace
kubectl exec -it POD_NAME -- /bin/bash  # access a pod
```

### `stern`: Live Logs in Your Terminal

[`stern`](https://github.com/stern/stern) is a CLI tool for tailing logs from multiple pods and namespaces.

Download the latest [release here](https://github.com/stern/stern/releases).

Example:

```bash
cd ~/Downloads
tar xzvf stern_1.28.0_linux_amd64.tar.gz
mv stern ~/.local/bin
```

> Ensure `~/.local/bin` is in your `PATH`, or move the binary elsewhere.

Now you can:

```bash
stern .                            # logs for current namespace
stern . -n ns1,ns2,ns3             # logs for multiple namespaces
```

## Conclusion

This setup helps me stay efficient while working with Kubernetes. It's minimal, fast, and works well when juggling multiple environments. If you're dealing with frequent context/namespace switches or want better log visibility, these tools are worth the effort.
