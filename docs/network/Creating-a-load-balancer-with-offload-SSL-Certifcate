# Creating a Load Balancer with offload SSL Certificate

In this tutorial we will create a load balancer to distribute traffic to specific backend servers, while offloading the SSL certificate on this load balancer 

Openstack load balancer (Octavia) based on HAproxy, and we will store the SSL certificate in the Openstack key manager (Barbican)

## Create the SSL self sign certificate

Normally we will have our certificate and key, but in this case we will use a self sign certificate

```
$ su - root
$ mkdir Cert && cd $_
$ openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes \
  -out certificate1.crt -keyout privatekey.key
```

## Import SSL Cert to Barbican

We want to import this new SSL cert to our secrets

- Listing our own secrets

```
$ openstack secret list
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| Secret href                                                               | Name        | Created                   | Status | Content types                           | Algorithm | Bit length | Secret type | Mode | Expiration |
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| https://10.11.55.254:9311/v1/secrets/f2d12e51-6691-4090-93c2-f68821202829 | tls_secret1 | 2020-11-20T15:18:16+00:00 | ACTIVE | {'default': 'application/octet-stream'} | aes       |        256 | opaque      | cbc  | None       |
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
```

- Generating pkcs12 file

PKCS12 files define an archive file format for storing cryptographic objects as a single file. API Connect supports the P12 file format for uploading a keystore and truststore. The keystore should contain both a private and public key along with intermediate CA certificates.

```
$ openssl pkcs12 -export -in certificate1.crt -inkey privatekey.key -out test1.p12
```

Importing SSL p12 file to our secrets 

```
$ openstack secret store --name='tls_secret2' -t 'application/octet-stream' \
  > -e 'base64' --payload="$(base64 < test1.p12)"
+---------------+---------------------------------------------------------------------------+
| Field         | Value                                                                     |
+---------------+---------------------------------------------------------------------------+
| Secret href   | https://10.11.55.254:9311/v1/secrets/a7ca3d58-400b-499c-aeb9-fbd53da78c78 |
| Name          | tls_secret1                                                               |
| Created       | None                                                                      |
| Status        | None                                                                      |
| Content types | {'default': 'application/octet-stream'}                                   |
| Algorithm     | aes                                                                       |
| Bit length    | 256                                                                       |
| Secret type   | opaque                                                                    |
| Mode          | cbc                                                                       |
| Expiration    | None                                                                      |
+---------------+---------------------------------------------------------------------------+
```
[NOTE] Make sure that you change name than other secrets you already have

```
$ openstack secret list
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| Secret href                                                               | Name        | Created                   | Status | Content types                           | Algorithm | Bit length | Secret type | Mode | Expiration |
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| https://10.11.55.254:9311/v1/secrets/db6391bf-b56e-4ff4-9ec9-2a2a6660c06f | tls_secret2 | 2023-02-09T11:17:05+00:00 | ACTIVE | {'default': 'application/octet-stream'} | aes       |        256 | opaque      | cbc  | None       |
| https://10.11.55.254:9311/v1/secrets/f2d12e51-6691-4090-93c2-f68821202829 | tls_secret1 | 2020-11-20T15:18:16+00:00 | ACTIVE | {'default': 'application/octet-stream'} | aes       |        256 | opaque      | cbc  | None       |
+---------------------------------------------------------------------------+-------------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
```

- Make sure that Octavia (LB) has read access on this new secret

```
$ secret_id=$(openstack secret list | awk '/ tls_secret2 / {print $2}')
$ openstack acl user add -u $(openstack user show octavia -c id -f value) $secret_id
+----------------+----------------+--------------------------------------+---------------------------+---------------------------+-------------------------------------------------------------------------------+
| Operation Type | Project Access | Users                                | Created                   | Updated                   | Secret ACL Ref                                                                |
+----------------+----------------+--------------------------------------+---------------------------+---------------------------+-------------------------------------------------------------------------------+
| read           | True           | ['192aea962f834d71b05110dcb820ebca'] | 2023-02-09T11:22:54+00:00 | 2023-02-09T11:22:54+00:00 | https://10.11.55.254:9311/v1/secrets/db6391bf-b56e-4ff4-9ec9-2a2a6660c06f/acl |
+----------------+----------------+--------------------------------------+---------------------------+---------------------------+-------------------------------------------------------------------------------+
```

Select which network we will attach the load balancer to, it should be located in the same network as your instances

```
$  openstack network list
```

Create new load balancer attached to our network

```
$ openstack loadbalancer create --name lb1 --vip-subnet-id external
$ openstack loadbalancer list
+--------------------------------------+------+----------------------------------+--------------+---------------------+------------------+----------+
| id                                   | name | project_id                       | vip_address  | provisioning_status | operating_status | provider |
+--------------------------------------+------+----------------------------------+--------------+---------------------+------------------+----------+
| 069f1059-8c6f-4ca7-b5ef-4bb2792f618d | lb1  | 4b658f06a9734994b7bcb2012b65d932 | 10.11.52.110 | PENDING_CREATE      | OFFLINE          | amphora  |
+--------------------------------------+------+----------------------------------+--------------+---------------------+------------------+----------+
```

Create a Listener on the load balancer, listening on 443 and attach the SSL certificate to it, this is the SSL offload cert

```
$ openstack loadbalancer listener create --protocol-port 443 --protocol TERMINATED_HTTPS \
  --name listener1 --default-tls-container=$secret_id lb1
```

Create a new pool for the newly created listener, we will use the Round_Robin algorithm, though there are other algorithms that can be used  

```
$ openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN \
  --listener listener1 --protocol HTTP
```

Now lets add VMs to this pool

```
$ openstack loadbalancer member create --subnet-id 536a6433-0610-4ad1-a005-8a3e2918bdbc \
  --address 10.11.52.191 --protocol-port 80 pool1

$ openstack loadbalancer member create --subnet-id 536a6433-0610-4ad1-a005-8a3e2918bdbc \
  --address 10.11.52.194 --protocol-port 80 pool1
```

- Listing members in our pool1

```
$ openstack loadbalancer member list pool1
+--------------------------------------+------+----------------------------------+---------------------+--------------+---------------+------------------+--------+
| id                                   | name | project_id                       | provisioning_status | address      | protocol_port | operating_status | weight |
+--------------------------------------+------+----------------------------------+---------------------+--------------+---------------+------------------+--------+
| 17d9df36-d3ad-43a6-965e-f2e7ef31b8a7 |      | 4b658f06a9734994b7bcb2012b65d932 | ACTIVE              | 10.11.52.191 |            80 | NO_MONITOR       |      1 |
| 8d536485-a81e-4f2a-9993-7b0b2af8411d |      | 4b658f06a9734994b7bcb2012b65d932 | ACTIVE              | 10.11.52.194 |            80 | NO_MONITOR       |      1 |
+--------------------------------------+------+----------------------------------+---------------------+--------------+---------------+------------------+--------+
```

## Creating a load balancer through horizon

Create new VMs 

![lboffcert1](../images/lboffcert1.png)

Creating new loadbalancer (under network section)

![lboffcert2](../images/lboffcert2.png)

[Note] When you create SSL Listener, Make sure to add TLS Cipher String is as the following

```
TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256
```

![lboffcert3](../images/lboffcert3.png)

![lboffcert4](../images/lboffcert4.png)

![lboffcert5](../images/lboffcert5.png)

If you want to make loadbalancer healthy check, configure Healthy monitor 

![lboffcert6](../images/lboffcert6.png)

![lboffcert7](../images/lboffcert7.png)
ß
## Time to test our cert

I installed nignx on both servers, and edited their welcome page to reflect VM name

![lboffcert8](../images/lboffcert8.png)

![lboffcert9](../images/lboffcert9.png)

You have now created a load balancer using horizon.