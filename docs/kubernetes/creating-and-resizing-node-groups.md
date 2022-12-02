# Creating and resizing node groups

Once you've created a Kubernetes cluster in Openstack it has 2 node groups by default: worker and master

```bash
$ openstack coe nodegroup list ${cluster_name}
+--------------------------------------+----------------+------------+-----------+------------+-----------------+--------+
| uuid                                 | name           | flavor_id  | image_id  | node_count | status          | role   |
+--------------------------------------+----------------+------------+-----------+------------+-----------------+--------+
| 9ff1c10b-c82a-40de-be81-b34f12d493a0 | default-master | cc1.medium | CoreOS-33 |          1 | CREATE_COMPLETE | master |
| d7c301ad-1c6a-46da-b983-cd9216407c6b | default-worker | ec1.medium | CoreOS-33 |          1 | CREATE_COMPLETE | worker |
+--------------------------------------+----------------+------------+-----------+------------+-----------------+--------+
```

We can create a new node group with a new instances specification, and this can categorize our workers workload.

Check the fields in 'default-worker' 

```bash
$ openstack coe nodegroup show ${cluster_name} default-worker
```

Create a new node group with our new specifications

```bash
$ openstack coe nodegroup create ${cluster_name} big-worker \
    --docker-volume-size 60 \
    --node-count 1 \
    --min-nodes 1 \
    --role worker \
    --image CoreOS-33  \
    --flavor ec1.large  \
    --labels boot_volume_size=20,boot_volume_type="encrypted",kube_tag="v1.21.9",cloud_provider_tag="v1.20.2",auto_healing_enabled="False",auto_healing_controller="magnum-auto-healer",magnum_auto_healer_tag="v1.23.0",auto_scaling_enabled="False",autoscaler_tag="v1.22.0",selinux_mode="disabled",monitoring_enabled="False",cinder_csi_enabled="True",master_lb_floating_ip_enabled="True",container_infra_prefix="ghcr.io/stackhpc/"
```

Now the new node group has been created, with a separate stack, and one worker node deployed 

```bash
$ openstack coe nodegroup list ${cluster_name}
+--------------------------------------+----------------+------------+-----------+------------+--------------------+--------+
| uuid                                 | name           | flavor_id  | image_id  | node_count | status             | role   |
+--------------------------------------+----------------+------------+-----------+------------+--------------------+--------+
| 9ff1c10b-c82a-40de-be81-b34f12d493a0 | default-master | cc1.medium | CoreOS-33 |          1 | CREATE_COMPLETE    | master |
| d7c301ad-1c6a-46da-b983-cd9216407c6b | default-worker | ec1.medium | CoreOS-33 |          1 | CREATE_COMPLETE    | worker |
| eb82c736-e69b-4f73-bd5d-1cd2703b55ae | big-worker     | ec1.large  | CoreOS-33 |          1 | CREATE_IN_PROGRESS | worker |
+--------------------------------------+----------------+------------+-----------+------------+--------------------+--------+
```

Show node group info

```bash
$ openstack coe nodegroup show ${cluster_name} eb82c736-e69b-4f73-bd5d-1cd2703b55ae

+--------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field              | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
+--------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| uuid               | eb82c736-e69b-4f73-bd5d-1cd2703b55ae                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| name               | big-worker                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| cluster_id         | 195343d5-73bb-4c27-be73-6514d037f9ce                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| project_id         | 17e5a06a7ce34fb689b6e0ee062f663c                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| docker_volume_size | 60                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| labels             | {'boot_volume_size': '20', 'boot_volume_type': 'encrypted', 'kube_tag': 'v1.21.9', 'cloud_provider_tag': 'v1.20.2', 'auto_healing_enabled': 'False', 'auto_healing_controller': 'magnum-auto-healer', 'magnum_auto_healer_tag': 'v1.23.0', 'auto_scaling_enabled': 'False', 'autoscaler_tag': 'v1.22.0', 'selinux_mode': 'disabled', 'monitoring_enabled': 'False', 'cinder_csi_enabled': 'True', 'master_lb_floating_ip_enabled': 'True', 'container_infra_prefix': 'ghcr.io/stackhpc/'} |
| labels_overridden  | {}                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| labels_skipped     | {'metrics_server_enabled': 'True', 'availability_zone': 'europe-nl-ams1'}                                                                                                                                                                                                                                                                                                                                                                                                                 |
| labels_added       | {'monitoring_enabled': 'False'}                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| flavor_id          | ec1.large                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| image_id           | CoreOS-33                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| node_addresses     | []                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| node_count         | 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| role               | worker                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| max_node_count     | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| min_node_count     | 1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| is_default         | False                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| stack_id           | cae60638-86c9-474a-8039-5fe4b5ca9cd1                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| status             | CREATE_IN_PROGRESS                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| status_reason      | None                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
+--------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Show resources related to this new stack

```bash
$ openstack stack resource list cae60638-86c9-474a-8039-5fe4b5ca9cd1
+---------------------------+--------------------------------------+-------------------------+--------------------+----------------------+
| resource_name             | physical_resource_id                 | resource_type           | resource_status    | updated_time         |
+---------------------------+--------------------------------------+-------------------------+--------------------+----------------------+
| kube_minions              | 5ab1cd94-05b9-4ad7-9da1-cbef4b065909 | OS::Heat::ResourceGroup | CREATE_IN_PROGRESS | 2022-11-21T10:38:37Z |
| worker_nodes_server_group | 873b0b9d-543f-4d0e-89b3-f013b0a074fa | OS::Nova::ServerGroup   | CREATE_COMPLETE    | 2022-11-21T10:38:37Z |
+---------------------------+--------------------------------------+-------------------------+--------------------+----------------------+
```

To check all resources in this stack

```bash
$ openstack stack resource list 5ab1cd94-05b9-4ad7-9da1-cbef4b065909
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+-----------------+----------------------+
| resource_name | physical_resource_id                 | resource_type                                                                                                        | resource_status | updated_time         |
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+-----------------+----------------------+
| 0             | bc912fd8-f2eb-4845-b77f-d24dd00414a9 | file:///var/lib/kolla/venv/lib/python3.6/site-packages/magnum/drivers/k8s_fedora_coreos_v1/templates/kubeminion.yaml | UPDATE_COMPLETE | 2022-11-21T11:04:30Z |
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+-----------------+----------------------+
```

To add a new worker for this new node group, make it 2 instances:

```bash
$  openstack coe cluster resize ${cluster_name} 2 --nodegroup big-worker
Request to resize cluster test-galal-resize has been accepted.
```

Checking resources for minions

```bash
$ openstack stack resource list 5ab1cd94-05b9-4ad7-9da1-cbef4b065909
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+--------------------+----------------------+
| resource_name | physical_resource_id                 | resource_type                                                                                                        | resource_status    | updated_time         |
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+--------------------+----------------------+
| 0             | bc912fd8-f2eb-4845-b77f-d24dd00414a9 | file:///var/lib/kolla/venv/lib/python3.6/site-packages/magnum/drivers/k8s_fedora_coreos_v1/templates/kubeminion.yaml | UPDATE_COMPLETE    | 2022-11-21T11:06:47Z |
| 1             | 426e6084-a8b5-4193-8762-19071f3682a0 | file:///var/lib/kolla/venv/lib/python3.6/site-packages/magnum/drivers/k8s_fedora_coreos_v1/templates/kubeminion.yaml | CREATE_IN_PROGRESS | 2022-11-21T11:06:47Z |
+---------------+--------------------------------------+----------------------------------------------------------------------------------------------------------------------+--------------------+----------------------+
```

Removing specific instance from a node group

```bash
$ openstack coe cluster resize ${cluster_name} 1 --nodegroup big-worker \
    --nodes-to-remove 2a7ac557-19a4-4820-a008-0908b3ae6379
Request to resize cluster test-galal-resize has been accepted.
```

We can suspend our workers (instances) inside specific node group if we want, buy suspending the stack 

```bash
$ openstack stack suspend cae60638-86c9-474a-8039-5fe4b5ca9cd1
+--------------------------------------+------------------------------------------+---------------------+----------------------+----------------------+
| ID                                   | Stack Name                               | Stack Status        | Creation Time        | Updated Time         |
+--------------------------------------+------------------------------------------+---------------------+----------------------+----------------------+
| cae60638-86c9-474a-8039-5fe4b5ca9cd1 | test-galal-resize-big-worke-4xwak6eek5ln | SUSPEND_IN_PROGRESS | 2022-11-21T10:38:37Z | 2022-11-21T11:20:19Z |
+--------------------------------------+------------------------------------------+---------------------+----------------------+----------------------+
```

Show stack info

```bash
$ openstack stack show cae60638-86c9-474a-8039-5fe4b5ca9cd1 --fit-width
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                                                                                                                                                               |
+-----------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                    | cae60638-86c9-474a-8039-5fe4b5ca9cd1                                                                                                                                                                                                                |
| stack_name            | test-galal-resize-big-worke-4xwak6eek5ln                                                                                                                                                                                                            |
| description           | This template will boot a Kubernetes cluster with one or more minions (as specified by the number_of_minions parameter, which defaults to 1).                                                                                                       |
|                       |                                                                                                                                                                                                                                                     |
| creation_time         | 2022-11-21T10:38:37Z                                                                                                                                                                                                                                |
| updated_time          | 2022-11-21T11:20:19Z                                                                                                                                                                                                                                |
| stack_status          | SUSPEND_COMPLETE                                                                                                                                                                                                                                    |
| stack_status_reason   | Stack SUSPEND completed successfully                                                                                                                                                                                                                |
...
```

```bash
$ openstack stack resource list cae60638-86c9-474a-8039-5fe4b5ca9cd1 --fit-width
+---------------------------+--------------------------------------+-------------------------+------------------+----------------------+
| resource_name             | physical_resource_id                 | resource_type           | resource_status  | updated_time         |
+---------------------------+--------------------------------------+-------------------------+------------------+----------------------+
| kube_minions              | 5ab1cd94-05b9-4ad7-9da1-cbef4b065909 | OS::Heat::ResourceGroup | SUSPEND_COMPLETE | 2022-11-21T11:20:20Z |
| worker_nodes_server_group | 873b0b9d-543f-4d0e-89b3-f013b0a074fa | OS::Nova::ServerGroup   | SUSPEND_COMPLETE | 2022-11-21T10:38:37Z |
+---------------------------+--------------------------------------+-------------------------+------------------+----------------------+
```

To resume node group work again

```bash
$ openstack stack resume cae60638-86c9-474a-8039-5fe4b5ca9cd1

+--------------------------------------+------------------------------------------+--------------------+----------------------+----------------------+
| ID                                   | Stack Name                               | Stack Status       | Creation Time        | Updated Time         |
+--------------------------------------+------------------------------------------+--------------------+----------------------+----------------------+
| cae60638-86c9-474a-8039-5fe4b5ca9cd1 | test-galal-resize-big-worke-4xwak6eek5ln | RESUME_IN_PROGRESS | 2022-11-21T10:38:37Z | 2022-11-21T11:20:19Z |
+--------------------------------------+------------------------------------------+--------------------+----------------------+----------------------+

$ openstack stack resource list cae60638-86c9-474a-8039-5fe4b5ca9cd1 --fit-width
+---------------------------+--------------------------------------+-------------------------+-----------------+----------------------+
| resource_name             | physical_resource_id                 | resource_type           | resource_status | updated_time         |
+---------------------------+--------------------------------------+-------------------------+-----------------+----------------------+
| kube_minions              | 5ab1cd94-05b9-4ad7-9da1-cbef4b065909 | OS::Heat::ResourceGroup | RESUME_COMPLETE | 2022-11-21T11:20:20Z |
| worker_nodes_server_group | 873b0b9d-543f-4d0e-89b3-f013b0a074fa | OS::Nova::ServerGroup   | RESUME_COMPLETE | 2022-11-21T10:38:37Z |
+---------------------------+--------------------------------------+-------------------------+-----------------+----------------------+
```

Now all resources in this stack are back to work