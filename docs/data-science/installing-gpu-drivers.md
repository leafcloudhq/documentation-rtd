# Installing NVIDIA Drivers and CUDA on Ubuntu 24.04

This guide provides a short tutorial on how to install NVIDIA drivers and CUDA on Ubuntu 24.04 (Noble Numbat).

## Prerequisites

* An Ubuntu 24.04 system with an NVIDIA graphics card.
* An active internet connection.
* Basic familiarity with the Linux command line.

## Steps 1. Install NVIDIA Drivers (if not already installed)

#### Update and Upgrade Package Lists:

```shell
sudo apt update && sudo apt upgrade -y
```

#### Install `ubuntu-drivers-common`:

The `ubuntu-drivers` command is part of the `ubuntu-drivers-common` package. If you encounter the "command not found" error, install it:

```shell
sudo apt install ubuntu-drivers-common
```

#### Identify Your NVIDIA Card and Recommended Driver:

```shell
ubuntu-drivers devices
```

This command will output information about your NVIDIA card and recommended driver packages. Note the recommended driver version.

#### Install the Recommended Driver (or a Specific Version):**

* **Install the recommended driver:**

```shell
sudo ubuntu-drivers autoinstall
```

* **Or install a specific driver version:**

```shell
sudo apt install nvidia-driver-XXX
```

**Replace `XXX` with the desired version.**

#### Reboot Your System:

```shell
sudo reboot
```

#### Verify the Installation:

```shell
nvidia-smi
```

## Step 2. Install CUDA (Specific Version: 12.8.0)

This section details installing CUDA version 12.8.0 using the local `.deb` repository method.

#### Download CUDA Repository Pin and Local Installer:

```shell
wget [https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin](https://www.google.com/search?q=https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin)
sudo mv cuda-ubuntu2404.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget [https://developer.download.nvidia.com/compute/cuda/12.8.0/local_installers/cuda-repo-ubuntu2404-12-8-local_12.8.0-570.86.10-1_amd64.deb](https://developer.download.nvidia.com/compute/cuda/12.8.0/local_installers/cuda-repo-ubuntu2404-12-8-local_12.8.0-570.86.10-1_amd64.deb)
```

#### Install the CUDA Repository:

```shell
sudo dpkg -i cuda-repo-ubuntu2404-12-8-local_12.8.0-570.86.10-1_amd64.deb
```

#### Add CUDA Repository Key:

```shell
sudo cp /var/cuda-repo-ubuntu2404-12-8-local/cuda-*-keyring.gpg /usr/share/keyrings/
```

#### Update Package Lists and Install CUDA Toolkit:

```shell
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-8
```

#### Set Environment Variables:

Add the CUDA bin and lib directories to your `PATH` and `LD_LIBRARY_PATH` environment variables. Open your `~/.bashrc` file with a text editor:

```shell
nano ~/.bashrc
```

Add the following lines to the end of the file:

```shell
export PATH=/usr/local/cuda-12.8/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.8/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Save the file and apply the changes:

```shell
source ~/.bashrc
```

#### Verify the CUDA Installation:

Check the CUDA version:

```shell
nvcc --version
```