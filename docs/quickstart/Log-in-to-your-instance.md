# Log in to your instance

This tutorial will show you how to log in to your instance

Open your terminal and enter the following:

```shell script
ls /
ssh [OS_name]@[Instance Public IP Address]
```

Note: the name of the OS should be entered in lowercase

In the case of our example instance this would be

```shellscript
ls /
ssh cirros@[Public IP]
```

Enter your username and password

When the "are you sure you want to continue" warning pops up, type *yes*

You have now logged in to your instance

## Log in using an imported Key Pair

To log into your instance using a specific SSH key, type the following in your terminal:

```shell script
ssh -i .ssh/id_rsa [OS_name]@[Public IP]
```

## Log in using a generated Key Pair

To log into your instance using a generated key, download your keypair file (.pem) locally

Navigate to the downloaded .pem file using your terminal

First secure your key file by entering:

```shellscript
chmod 400 [name of your key pair].pem
```

Now log in using the following command: 

```shell script
ssh -1 [key pair name].pem [OS_name]@[Public IP]
```

You have logged in to your instance