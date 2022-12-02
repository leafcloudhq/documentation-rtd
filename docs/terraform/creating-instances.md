# Using Terraform
#leafcloud/tutorials

In this tutorial I'm going to show an example of how to setup a highly available service on Leafcloud, using Terraform.

## Configure the CLI if you have not done so already.
Login to Leafcloud, go to "identity", create and download "application credentials" as clouds.yaml. Now copy or move this file to ~/.config/openstack/clouds.yaml

Finally; tell the Openstack client to use your cloud: 

`export OS_CLOUD=openstack`

 I recommend putting this in your .bash_rc (linux) or .profile (mac) file, so it sets this in your shell on launch.

If you have not used the OpenStack CLI, this is also a good moment to install that. Installing is as simple as 

`pip install python-openstackclient`

Then try: 

`openstack image list`

If it returns a list of available server images your setup was correct.

## Setup Terraform with the OpenStack driver
Terraform will read all files with `.tf` a extension, but if you follow this guide your folder will look something like this:
```
main.tf
networks.tf
instances.tf
loadbalancer.tf
```

`main.tf`
```hcl
terraform {
required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.39.0"
    }
  }
}

# Configure the OpenStack Provider to use the (newer) Octavia loadbalancer
provider "openstack" {
  use_octavia = true
}
```

It's good to now run the setup. Tun `terraform init`. Terraform should now download the OpenStack driver and configure the relevant versions. 

## Create a private network and a Compute instance
Next, we're going to setup a (private) network, and a subnet.

`networks.tf`
```
resource "openstack_networking_network_v2" "private" {
  name = "private"
  admin_state_up = "true"
}

resource "openstack_networking_subnet_v2" "default_sn" {
  name = "default_sn"
  cidr = "192.168.42.0/24"
  ip_version = 4
  network_id = openstack_networking_network_v2.private.id
}
```

This would be a good time to do your first apply, to make it happen. Run `terraform apply`. Terraform should return you the plan it wants to execute.  Enter 'yes' to apply. If no errors occur you should have newly network.

You can check with `openstack network list`

## Define the instances you want to start
And now setup the actual compute instances. The following snippet will configure three instances to start.

Notice the "count = 3". This is a variable to indicate we want to have three of the same servers.

`instances.tf`
```
resource "openstack_compute_instance_v2" "blue" {
  # The amount of instances you want to start
  count = 3

  name            = "blue_${count.index}"
  flavor_name     = "ec1.large"
  key_pair        = "<your_key_name>"
  security_groups = ["default"]
  # user_data       = file("cloud-init-blue.yaml")

  network {
    name        = data.openstack_networking_network_v2.private.name
    # Here we get the value of the count and add 10, 
    # so that each instance gets it's own unique I
    fixed_ip_v4 = "192.168.42.${count.index + 10}"
  }
}
```

Now run `tf apply` again. Check with `openstack server list` that your servers were created.

## Using LoadBalancers
The next part is to create a LoadBalancer. These are great for splitting the load over the different servers you have created.

Note that here, we use the 'count' variable on the members, so that each instance will be added as a member to the default pool.

`loadbalancers.tf`
```
resource "openstack_lb_loadbalancer_v2" "demo_lb" {
  name           = "demo-loadbalancer"
  vip_network_id = data.openstack_networking_network_v2.external.id
}

resource "openstack_lb_listener_v2" "listener_1" {
  protocol        = "HTTP"
  protocol_port   = 80
  loadbalancer_id = openstack_lb_loadbalancer_v2.demo_lb.id

  insert_headers = {
    X-Forwarded-For = "true"
  }
}

resource "openstack_lb_pool_v2" "main_pool" {
  protocol    = "HTTP"
  lb_method   = "ROUND_ROBIN"
  listener_id = openstack_lb_listener_v2.listener_1.id
}

resource "openstack_lb_members_v2" "members_1" {
  count   = 3
  pool_id = openstack_lb_pool_v2.main_pool.id

  member {
    address       = "192.168.42.${count.index + 10}"
    protocol_port = 80
  }
}
```

Now check that these are started:
`openstack loadbalancer list`

If the Loadbalancer command is not found you can install it with `python-octaviaclient`. It will be installed as a plugin to your openstack CLI


