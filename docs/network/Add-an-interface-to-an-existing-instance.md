# Add an interface to an existing instance

Let's say you want to add an interface to an existing instance on a network that you created newly.

You can create a port on that network which is attached to the instance like this:

```shell
openstack port create --network new-network my-interface
openstack server port add <instance-id> my-interface
```

Now there is a new interface attached to your instance, but it is still in status `down`
To bring the port up do:

```shell
sudo ip addr add <ip-cidr> dev ens7
ip link set dev <new-device> up
```

Now your instance has connectivity on the new network, but this is not persisted.

Depending on your linux distribution, the way persist this may very. On ubuntu, netplan is used by default. 

To add the new interface to netplan, create a new file:

```shell
sudo nano /etc/netplan/99-config.yaml
```

Add the following contents to the file and save it, 
replace <device-name> and <your-mac-address> with the appropiate values.

```shell
network:
    version: 2
    ethernets:
        <device-name>:
            dhcp4: true
            match:
                macaddress: <you-mac-address>
            mtu: 1500
            set-name: <device-name>
```

Reboot the instance.
