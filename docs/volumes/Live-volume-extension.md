# Live volume extension 

**Note: before you start this tutorial, please create a backup of your volume. You can find a tutorial for that [here](https://docs.leaf.cloud/en/latest/volumes/Creating-a-volume-backup/).**

Often, you can find yourself unsure what amount of storage an instance will need. You may start it at a certain size before realizing you need more space once it’s running. While adding more space to a volume is straightforward, doing so while keeping it in use can pose problems. This tutorial will teach you how to extend the size of a secondary volume, without having to unmount it first. This allows you to add more space to your instance while keeping it running. 
 
## Building and adding a secondary volume with LVM 

Start by creating an instance using a standard Ubuntu image, then create an additional volume and attach it to your instance. You can find the tutorials for this in the *Getting Started* section

Now, create an LVM Volume Group on the new volume: 

``
vgcreate /dev/MyVG01 /dev/vdb 
`` 

Once you've done that, create a Logical Volume in the new Volume Group: 

`` 
lvcreate -L +9G --name test MyVG01 
`` 

You have now built and added a secondary volume with an LVM Volume Group  

## Build and extend a partition containing a new Volume live using LVM 

First, create an *ext4* partition in the new Logical Volume: 

`` 
mkfs -t ext4 /dev/MyVG01/test 
`` 

Now mount the new partition: 

``
mkdir /mnt/tes
mount -rw /dev/MyVG01/test /mnt/test 
`` 

Create another new volume and attach it to the instance. Now, you can extend the Volume Group: 

`` 
vgextend /dev/MyVG01 /dev/vdc 
`` 

Followed by extending the Logical Volume: 

`` 
-L +11G /dev/MyVG01/test 
`` 

And finally, resize the partition to the new maximum size: 

`` 
resize2fs /dev/MyVG01/test 
`` 

You have now used LVM to build and extend a partition containing a new volume while live