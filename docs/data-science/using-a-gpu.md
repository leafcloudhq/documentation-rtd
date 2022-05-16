# Using a GPU
In this tutorial we're going to: 
 - Start an instance
 - Install NVIDIA drivers
 - Run a Tensorflow becnhmark

## Start an Instance

We are going to create an Ubuntu 20.04 based Virtual Machine instance.
You can start an instance either using the OpenStack CLI, or using the Horizon web interface.

On the CLI:
```shell
nova boot --flavor eg1.a30x1.V8-32 --key-name my_key --nic net-name=external my-gpu-instance --block-device source=image,dest=volume,size=30,id=0e98efbf-de1e-4e58-bf77-1ca80668e305,shutdown=preserve,bootindex=0
```
Replace `my_key` with the name of a key pair from your account.

This will start a new Ubuntu 20.04 VM, with 8 CPU cores, 32GB of RAM and a single NVIDIA A30 Tensor Core GPU.
We have flavors with up to 8 of these GPUs.

You can obtain your IP address by running:
```shell
openstack server show my-gpu-instance
```

We SSH into our VM:
```shell
ssh ubuntu@<ip_address>
```
Replace `<ip_address>` with the ip address of your VM. 


## Install drivers
The next step is to install the NVIDIA drivers.
To make sure we install the correct version of the drivers, we will check the current Tensorflow CUDA and cuDNN compatibility:
https://www.tensorflow.org/install/source#gpu
We will use CUDA 11.2 and cuDNN 8.1

Then we will install the GPU drivers and CUDA on the VM:
```shell
sudo apt update
sudo apt install gcc make
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda_11.2.0_460.27.04_linux.run
sudo sh cuda_11.2.0_460.27.04_linux.run
```
Only install the cuda toolkit

Install the nvidia driver:
```shell
sudo apt install nvidia-driver-460-server
```

Add the libraries to environment variables.
At the bottom of the `~/.bashrc` file add
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.2/lib64
export PATH=$PATH:/usr/local/cuda-11.2/bin
```

```shell
nano ~/.bashrc
source ~/.bashrc
```

We will download cuDNN 8.1 runtime library from https://developer.nvidia.com/rdp/cudnn-archive 
Then we SCP it to our server:
```shell
scp ./libcudnn8_8.1.1.33-1+cuda11.2_amd64.deb ubuntu@45.135.56.94:libcudnn8_8.1.1.33-1+cuda11.2_amd64.deb
```
There we will install it:
```shell
sudo apt install ./libcudnn8_8.1.1.33-1+cuda11.2_amd64.deb
```

and reboot the VM
```shell
sudo reboot
```

Check that the drivers are correctly installed and active:
```shell
nvidia-smi
```

## Run a Tensorflow benchmark

First we'll create a virtual env
```shell
sudo apt install python3-venv
python3 -m venv venv
source venv/bin/activate
```

Then we'll install the tensorflow package.
```shell
pip install wheel
pip install tensorflow
```

clone the becnhmarks repo:
```shell
git clone https://github.com/tensorflow/benchmarks.git
cd benchmarks/scripts/tf_cnn_benchmarks
```

run the benchmark:

```shell
python tf_cnn_benchmarks.py --num_gpus=1 --batch_size=32 --model=resnet50 --variable_update=parameter_server
```
