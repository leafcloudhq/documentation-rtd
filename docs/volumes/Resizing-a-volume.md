# Resizing (extending) a volume

**Note: before you start this tutorial, please create a backup of your volume. You can find a tutorial for that [here](https://docs.leaf.cloud/en/latest/volumes/Creating-a-volume-backup/).**

Sometimes you find yourself in the situation that the disk you provisioned for your server is not large enough. It can then be really useful to resize the volume. 

This works best if your server runs a compatible filesystem and OS. We do not support Windows at this time.


There are generally situations: 
1. The volume is a secondary attached volume on your instance.
2. The volume you want to resize is the root-disk of an instance


## If the volume is a secondary attached volume

In this case you can unmount and detach the volume before resizing. The following steps will describe the process.

1. Unmount the volume
    Login to the server and find which disk is the one in question `df -h`. It will probably be called something like /dev/vdb
    
    ```
    unmount /dev/<drive>
    ```

2. Identify which is the correct volume to detach.
    Now using the command line on your local machine you can find the details

    ``` 
    openstack server show <server_id>
    ```

    Look for the section that says: Volumes attached. 
    
    Check that this is the correct volume 
    
    ```
    openstack volume show <volume_id>
    ```

3. Detach the volume
    Now you can detach the volume

    ```
    openstack server remove volume <server> <volume>
    ```

4. Resize the volume
    Now you can specify the new size:

    ```
    openstack volume set --size <new_size> <volume>
    ```

5. Re-attach te volume

    ```
    openstack server add volume <server> <volume>
    ```

6. Resize the volume from within the server
    Now you should be able to see the disk again from the shell in your server.

    ```
    lsblk
    ```

    Depending on the filesystem you have used the command will be different. For ext3, the command is: 
    
    ```
    # not specifying any parameters typically will grow to the maximum size.
    resize2fs /dev/<disk>
    ```

7. Re-mount the disk on a location of your choice
    As a last step you need to re-mount you disk
    
    ```
    # if it was attached before (your drive was still configured in /etc/fstab):
    mount --all

    # otherwise mount it to a new location (this will not persist across reboots)
    mount /dev/<disk_number> /mnt/<disk_number>
    ```

You have now successfully expanded a volume.


## If the volume is your bootdisk

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

