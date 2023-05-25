# Deploying Claris Filemaker on Ubuntu

This tutorial will guide you through the steps to set up a Claris Filemaker server on Ubuntu. Once you have completed this tutorial you will have access to the web-interface.   

## Create an instance
First, create an instance with the following minimum requirements:

*Note: if you do not know how to set up an instance, please use the 'Launching an instance tutorial', which you can find [here](https://docs.leaf.cloud/en/latest/Getting-Started/Launching-an-instance/).*

|Users|Vcpu |Ram  |
|--|--|--|
|5-9  |2  |6  |
|10-39 |4  |16  |
|40-99  |8  |32  |
|100+  |8  |32  |

create a large instance with a 500GB disk & ubuntu 20 (required by the [sys req](https://support.claris.com/s/article/Claris-FileMaker-19-4-19-5-Technical-Specifications?language=en_US))

Use SSH to connect to the instance using the keypair specified when creating the instance and user name 'ubuntu'. For Linux and MacOS enter the following command in the terminal:

    ssh ubuntu@<your_ip>

For Windows: use your prefered client, for instance [Putty](https://www.putty.org/)

Once you've connected to your instance you can proceed to update it:

    sudo apt update &&
    sudo apt -y upgrade &&
    sudo apt -y dist-upgrade &&
    sudo apt -y autoremove &&
    sudo apt -y autoclean

Now Install the dependencies:

    sudo apt install -y wget unzip apache2 php openjdk-11-jre-headless

## Downloading and installing Filemaker

Create a new folder and download the filemaker package into it. Then open the folder:

    mkdir fminstaller && cd fminstaller

By the time you're reading this, this software version might be outdated. Check your registration email to confirm your using the latest version. 

Download the package using the information from the registration e-mail:

    wget https://downloads.claris.com/esd/fms_19.5.4.400_Ubuntu20.zip

Unzip and remove the archive:

    unzip fms_19.5.4.400_Ubuntu20.zip && rm fms_19.5.4.400_Ubuntu20.zip

Now install the debian package:

    sudo apt install ./*.deb

Press **'y'** to start installing the dependencies and wait for installation to finish.

Once the installation has finished, press 'y' to accept the terms of service.

Next, press **'0'** [zero] to make this the primary filemaker machine
* Pick a username for the admin account
* Pick a password for the admin account
* Pick a 4-digit pin (this is used to reset the admin credentials from the CLI)

In the create.leaf.cloud environment create a security group with the following ports:

* ingress TCP 80 (http)
* ingress TCP 443 (https)
* ingress TCP 16000 (filemaker web)
* ingress TCP 50004 (filemaker script engine)


*Note: You can find a step by step guide for configuring security groups [here](https://docs.leaf.cloud/en/latest/Getting-Started/Configuring-security-groups/)*

Now add the new security group to your instance. 

## Verifying  

*Note: It is advised to you Google Chrome (or a Chrome-based) browser for this step.*

Open your preferred browser in a way that makes it ignore invalid certificates. This is an example where chromium browser is opened from the terminal: 

    chromium-browser --ignore-certificate-errors

On Windows:
First launch the Chrome browser.

Next, in the addresss bar enter the following:

    chrome://flags/

Type **'secure'** in the search box and click enter, then scroll down to the **'Allow invalid certificates for resources loaded from localhost'** flag. 

Click on the dropdown list and select Enabled, then restart the browser.

*Note: If you don't ignore certificate errors your browser's built in protection will refuse to present you the web interface.*

browse to 
> _https://<your_ip>/admin-console_, replacing <your_ip> with your instance IP

For now, accept the usage of the claris default certificate, once a DNS record is created a proper SSL certificate can be loaded from the web interface, you could get one for free by using certbot and manually creating one, however this is not recommended since these are only valid for 3 months and autorenewal is not yet working. 
