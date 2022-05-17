# Running jupyter

Create a VM with NVIDIA drivers, CUDA, cuDNN, and Tensorflow or Pytorch.

Create a security group for jupyter:
From you local machine run
```shell
openstack security group create jupyter --description "Jupyter notebook"
openstack security group rule create jupyter --protocol TCP --dst-port 8888:8888 --remote-ip 0.0.0.0/0
```

The ssh to the VM and activate the virtualenvironment within which you installed your machine learning library:
```shell
source venv/bin/activate
```

Install jupyter:

```shell
pip install jupyter
```

And run it on the VM's public ip:
```shell
jupyter notebook --ip=<public ip>
```

Then you can open the link that will be printed in the browser:
```shell
...
    Or copy and paste one of these URLs:
        http://45.135.56.94:8888/?token=0e097ece1b81642483ce3b4624d91dc6209181351b98cd53
...
```

Enjoy your notebook editing!
