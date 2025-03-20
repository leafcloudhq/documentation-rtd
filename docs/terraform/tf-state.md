# Using Object Storage as a Terraform Backend

By default, Terraform stores its state on the local disk of the developer or operator. This approach limits collaboration, risks data loss, and hinders automation.

[Read more about what Terraform state is and why it exists here.](https://developer.hashicorp.com/terraform/language/state)
Terraform supports various backends that allow you to store state in external systems. In this tutorial, we'll be setting up our Terraform project to use [S3 compatible object storage](https://developer.hashicorp.com/terraform/language/backend/s3) as backend state storage.


### 0. Requirements

This tutorial expects you to have set up the openstack command line tool. See [Getting Started -> How to use the Openstack CLI](../../Getting-Started/Using-Openstack-CLI) for a detailed guide.

## 1. Cloud Resources

Create an object storage container to host the statefiles and ec2-compatible credentials to access this container programmatically:
```bash
openstack container create tf-state
```
```bash
openstack ec2 credentials create
```
The latter command will output the access credentials and secret we'll plug into our terraform backend configuration in the next step.


## 2. Terraform Code

What follows is a bare minimum Terraform configuration to get you started on your LeafCloud. Pay special attention to the `backend` block, which is where the credentials we generated in our last step are to be plugged in.
```C
terraform {
required_version = ">= 0.14.0"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.53.0"
    }
  }
}

provider "openstack" {
  auth_url    = "https://create.leaf.cloud:5000"
  region      = "europe-nl"
  use_octavia = true # support octavia loadbalancer
  tenant_id   = "project ID: see create.leaf.cloud/project/api_access -> view credentials"
}

terraform {
  backend "s3" {
    bucket                      = "tf-state"
    key                         = "cool-site/terraform.tfstate"
    region                      = "required-but-not-used"
    access_key                  = "$$ access credential $$"
    secret_key                  = "$$ secret $$"
    skip_credentials_validation = true
    skip_requesting_account_id  = true
    skip_s3_checksum            = true
    skip_region_validation      = true # Region is not required for Openstack S3 Swift
    use_path_style              = true # Prevent using the bucketname as the hostname
    endpoints = {
      s3  = "https://leafcloud.store"
      sts = "https://leafcloud.store"
    }
  }
}
```

Once you have this in an updated `main.tf` file, you should be able to run `terraform init` in the repository and get started managing your LeafCloud infrastructure using Terraform!

!!! warning "Credential Management"
    For simplicity, we've explicitly defined the access credentials (`access_key` and `secret_key`) in the Terraform backend block. However, storing credentials in plaintext —especially in a Terraform file that may end up in a Git repository— is a poor security practice.

    Instead, credentials should be provided externally such as through environment variables (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`), which are automatically recognized by the S3 backend provider.
