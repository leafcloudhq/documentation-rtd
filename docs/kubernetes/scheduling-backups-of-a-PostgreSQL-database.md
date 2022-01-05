# Scheduling backups of a PostgreSQL-database

Scheduling the backups of your PostgreSQL-database allows you to back-up when it suits your needs. You can do this by configuring Leafcloud S3 compatible Object Storage in pgo.yml. 
To do this, follow the steps below:

## Step 1: create credentials for the ec2 compatible Leafcloud API

First, you need to create credentials for the ec2 compatible Leafcloud API:

```
openstack ec2 credentials create
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | 17e5a06a7ce34sjfu5b6e0eejt00663c |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | d407b0kckdu3994j5jjg8403c0973cf9 |
| name                | myuser                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

```
Cluster:
BackrestS3Bucket: my-postgresql-backups-example 

BackrestS3Endpoint: leafcloud.store

BackrestS3Region: europe-nl-ams1 

BackrestS3URIStyle: host 

BackrestS3VerifyTLS: true 
```
  
  Visit here for more information:
  https://access.crunchydata.com/documentation/postgres-operator/4.7.2/architecture/disaster-recovery/#using-s3

  ## Step 2: Schedule your backups

 Second, schedule your backups as follows:
```
pgo create schedule hacluster --schedule="0 1 * * *" \
  --schedule-type=pgbackrest  --pgbackrest-backup-type=full

pgo create schedule hacluster --schedule="0 */3 * * *" \
  --schedule-type=pgbackrest  --pgbackrest-backup-type=incr
```

Visit here for more information:
https://access.crunchydata.com/documentation/postgres-operator/4.7.2/pgo-client/common-tasks/#disaster-recovery-backups-restores

You are now able to schedule the backups of your PostgreSQL-database