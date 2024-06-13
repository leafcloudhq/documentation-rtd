# Installation of gardenctl

gardenctl is a command-line tool used for interacting with Gardener clusters. It allows you to manage clusters and target specific ones to run kubectl commands on. This tutorial will guide you through the steps to install and configure gardenctl. In this tutorial, we will use Ubuntu, but macOS and Windows are also supported.

## Prerequisites

Before you begin, ensure you have the following:

- A system with sudo privileges.
- Access to the internet for downloading the required packages.


## Step 1: Update System Packages

```bash
sudo apt update
sudo apt upgrade -y

```

## Step 2: Install Required Dependencies

Install the necessary dependencies for building gardenctl from source:

```bash
sudo apt install -y curl git gcc make jq

```

## step 3: Install gardenctl
Follow these steps to install gardenctl. You will need to set the operating system and architecture for your specific environment.

```bash
# Example for linux

# set operating system and architecture
os=linux # choose between darwin, linux, windows
arch=amd64 # choose between amd64, arm64

# Get latest version. Alternatively set your desired version
version=$(curl -s https://raw.githubusercontent.com/gardener/gardenctl-v2/master/LATEST)

# Download gardenctl
curl -LO "https://github.com/gardener/gardenctl-v2/releases/download/${version}/gardenctl_v2_${os}_${arch}"

# Make the gardenctl binary executable
chmod +x "./gardenctl_v2_${os}_${arch}"

# Move the binary in to your PATH
sudo mv "./gardenctl_v2_${os}_${arch}" /usr/local/bin/gardenctl

```

## step 4: Install gardenlogin

```bash
# set operating system and architecture
os=linux # choose between darwin, linux, windows
arch=amd64

# Get latest version. Alternatively set your desired version
version=$(curl -s https://raw.githubusercontent.com/gardener/gardenlogin/master/LATEST)

# Download gardenlogin
curl -LO "https://github.com/gardener/gardenlogin/releases/download/${version}/gardenlogin_${os}_${arch}"

# Make the gardenlogin binary executable
chmod +x "./gardenlogin_${os}_${arch}"

# Move the binary in to your PATH
sudo mv "./gardenlogin_${os}_${arch}" /usr/local/bin/gardenlogin

# create kubectl-gardenlogin symlink
sudo ln -s /usr/local/bin/gardenlogin /usr/local/bin/kubectl-gardenlogin


```

## step 5: Install kubelogin

```bash
wget https://github.com/int128/kubelogin/releases/download/v1.28.1/kubelogin_linux_amd64.zip
unzip kubelogin_linux_amd64.zip
sudo mv kubelogin /usr/local/bin/kubectl-oidc_login
```

## step 6: Download personal kubeconfig file

- Login to Gardener dashboard with your Leafcloud credentials
   https://dashboard.gardener.leaf.cloud/
- Select the account button top right and click on "My account" 
- Download the Kubeconfig file from the "Download kubeconfig file" section.

**Important**: The downloaded Kubeconfig file provides access to your Gardener clusters and should be treated with the same level of care as your other cloud credentials. Avoid sharing this file with anyone and store it securely.

## step 7: Create a gardenctl config file

Create a new file named gardenctl-v2.yaml under the directory ~/.garden. This file will store the configuration for your Gardener clusters.

```yaml
gardens:
  - identity: leafcloud-production
    kubeconfig: "path to downloaded file from step 6
```

## step 8: Add the following to your .bashrc or .zshrc file

```bash
source <(gardenctl completion bash)


[ -n "$GCTL_SESSION_ID" ] || [ -n "$TERM_SESSION_ID" ] || export GCTL_SESSION_ID=$(uuidgen)

if [ -z "$GCTL_SESSION_ID" ] && [ -z "$TERM_SESSION_ID" ]; then
  export GCTL_SESSION_ID=$(uuidgen)
fi

eval $(gardenctl kubectl-env bash)
```
 
refresh your .bashrc or .zshrc file.
```bash
source ~/.bashrc

```


# Using gardenctl

Using Gardenctl, you can efficiently manage your Kubernetes clusters within a Gardener-managed environment. By targeting your garden, you can list and manage all clusters. Additionally, Gardenctl allows you to target a specific cluster (shoot) to run kubectl commands directly on that cluster, enabling precise and effective cluster management.


## target your garden to list clusters

Run the following command to target your garden:

```bash
gardenctl target --garden leafcloud-production --project <your-project-id>
```

To list all clusters in your garden, run:

```bash
kubectl get shoots
```

To get the full details of a specific cluster, you can use the following command:

```bash
kubectl get shoots <cluster-name> -o yaml
```

## target a specific cluster to run kubectl commands on it.

To target a specific cluster (shoot), run:

```bash
gardenctl target --garden prod-leaftest --project rbqlcbxiav  --shoot clustername 
```

Now, you can use kubectl commands to manage the selected cluster.