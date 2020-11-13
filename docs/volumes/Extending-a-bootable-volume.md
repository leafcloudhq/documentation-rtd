# Extending a bootable volume

Open your terminal and enter the following to look up the volume ID 

```shell 
openstack volume list
```

You can use the Cinder API to extend the size of your volume. For the purpose of this tutorial we will extend the volume we created earlier to 20 GB

```shell
OS_VOLUME_API_VERSION=3.42 cinder extend [volume ID] 20
```

To use the new space, shut down the instance and start it again

First: 

```shell
openstack server stop [Server name]
```

Then start it again: 

```shell
openstack server start [Server name]
```

Note that simply restarting your server won't apply the extension

To make sure the new volume size has been applied, log in through SSH and use the following command

```shell
lsblk
```

You have now extended the size of your bootable volume

