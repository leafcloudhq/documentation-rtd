# Connecting to your Instance

This tutorial will show you how to log in to your instance

### To log in using a key pair:

To add a key pair to your instance see [Launching an Instance](Launching-an-instance.md)

Open your terminal and enter the following:

``` shell
ssh -i <path to your private ssh key> [instance os_name]@[Instance Public IP Address]
```

The first time you connect with your instance the terminal might ask for permission, enter 'yes'

You have now logged in to your instance

### To log in with a username and password:

Open your terminal and enter the following:

``` shell
ssh [instance os_name]@[Instance Public IP Address]
```

So, for example: 

``` shell
ssh ubuntu@[Public IP]
```

Enter your username and password

You have now logged in to your instance
