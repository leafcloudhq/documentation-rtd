
### Setting up the usage of object storage
In order to get credentials for the Openstack Object storage you'll need to install and setup the Openstack CLI

Follow the instructions to install and configure the Openstack CLI  [Using the Openstack CLI - Leafcloud Tutorials](https://docs.leaf.cloud/en/latest/Getting-Started/Using-Openstack-CLI/)


### Making an object storage container (bucket)
Objects (files) are organized in so-called containers, or buckets, there is a limit to how many containers you can create, and a bucket is typically created for each project or service.

Containers need to have a name that is unique for all Leafcloud users. In other words, it may be that a name is already taken. 

```sh
openstack container create <container_name>
```

Storage containers can also be viewed using the [dashboard](https://create.leaf.cloud/project/containers/)  under Object Store

### Creating S3 compatible credentials
In Leafcloud, for each project an EC2 credential can be made. These credentials have access to your project and the storage containers within.

```sh
openstack ec2 credentials create
```

And example result contains:
```text
access     | 67fd5d4526114c97b87ace3981bd75cb
secret     | 9da048146f604573bef0fa97afb3509d                                                                                                         
```

These 'access' and 'secret' strings are the username and password for the object storage containers. Keep them safe. You cannot retrieve them later. But you can create new ones.

### Access you bucket from your local computer using a GUI
There are many different clients available which support "Amazon S3", And Leafcloud object storage is fully compatible with them. One of the best (free) clients is [Cyberduck](https://cyberduck.io/download/). It's open-source and payment is optional. 

1. Use **leafcloud.store** as the server address.
2. Use your **access** key for Access Key ID
3. Use your **secret** for Secret Access Key


[image:configure_cyberduck.png]


### Access your bucket from your server / using a CLI 
Again, there are many different clients available which support S3. One of the best clients that we like is the [Minio Client](https://docs.min.io/docs/minio-client-complete-guide.html). You don't need to use the Minio server to use the Minio client.

Once you have installed the client you can use the following command to setup Leafcloud object storage:
```sh
mc alias set leafcloud https://leafcloud.store [YOUR-ACCESS-KEY] [YOUR-SECRET-KEY]
```
You can then find the configuation file in ~/.mc/config.json

Test if you can list your buckets with:
```sh
$ mc ls leafcloud
```

You can copy things to the bucket like so:
```sh
mc copy <local_file> leafcloud/<bucket_name>/<path>/
```
