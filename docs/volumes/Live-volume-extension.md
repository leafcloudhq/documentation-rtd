# Live volume extension 

This tutorial will teach you how to extend the size of a volume, without having to deactivate it 

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