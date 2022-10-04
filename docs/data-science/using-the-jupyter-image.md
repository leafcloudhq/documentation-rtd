# Using the Jupyter image

For convenience, Leafcloud offers a pre-built image with Nvidia CUDA and CuDNN, TensorFlow and PyTorch and Jupyter Notebook pre-installed.

## Launching the instance

To start an instance using this image, go to the instance view of the [dashboard](https://create.leaf.cloud/project/instances/).
1. Press Launch intance
1. in the Source tab select "Image" and the boot source
1. select the `ubuntu-pytorch-tensorflowf-jupyter` image
1. set the volume size to 40GB
1. as the flavor select any gpu flavor, starting with `eg1.`
1. Select a network, to give the instance a public IP address use the `external` network
1. Select a key pair, this should will be used to `SSH` into the machine
1. Select a security group that exposes the `8888` port
1. Press "Launch Instance"

## Open Jupyter Notebook

In the [instances](https://create.leaf.cloud/project/instances/) view, copy the IP address of your new instance.

### Obtaining the Jupyter Notebook token
From a terminal `SSH` into the instance:
```
ssh ubuntu@<ip-address>
sudo systemctl status jupyter
```

Near the bottom of the output of the above command will be a line simila to:
```
http://45.135.56.154:8888/?token=<your-token>
```
Copy this url and open it in a web browser.

This is your Jupyter Notebook server!

## Validating the installation

To validate that the machine is using a GPU you can create a new notebook and run the following

### For TensorFlow
```
import tensorflow as tf 

if tf.test.gpu_device_name(): 
    print('Default GPU Device:{}'.format(tf.test.gpu_device_name()))
else:
   print("Please install GPU version of TF")
```

This should output something like:
```
Default GPU Device:/device:GPU:0
```

### For PyTorch
```
import torch
torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.current_device()
torch.cuda.device(0)
torch.cuda.get_device_name(0)
```

This should output something like:
```
'NVIDIA A30'
```
