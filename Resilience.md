# Resilience

Created: March 11, 2023 3:56 PM
Created By: barak lagziel
Last Edited By: barak lagziel
Last Edited Time: March 13, 2023 10:01 AM
Type: Technical Spec

# Background

Will provides several resilience specifications to ensure high availability and durability of the database

# Goals

. These specifications include:

1. Multi-AZ Deployments: With multi-AZ deployments, automatically replicates data to a standby instance in a different availability zone (AZ) for failover in the event of a planned or unplanned outage.
2. Read Replicas: Read replicas enable the creation of one or more copies of the database instance to serve read-only queries. These replicas can be used to offload read traffic from the primary database, improve query performance, and provide additional resilience in case of a primary instance failure.
3. Automated Backups:  provides automated backups that can be configured to occur daily, weekly, or monthly. These backups can be retained for up to 35 days, providing point-in-time recovery options.
4. Database Snapshots:  also allows manual snapshots of the database to be created, which can be used to restore the database to a specific point in time.
5. Multi-Region Deployments:  allows for the creation of replicas in different regions to provide disaster recovery options in the event of a regional outage.
6. Elastic Load Balancing:  can be integrated with Elastic Load Balancing to distribute traffic across multiple database instances and improve fault tolerance.

# Proposed Solution

Describe the solution to the problems outlined above. Include enough detail to allow for productive discussion and comments from readers.

# High availability

Without High Availability, your instance will be backed up and monitored by our control plane. In the event of a loss of service, we will automatically restore your instance using the most recent backup. For small databases, this could be a matter of minutes, but for larger databases this can take several hours. For customers that have business critical applications and cannot tolerate downtime, we recommend using our High Availability feature. While disaster recovery is intended to ensure resiliency in an event of a failure, high availability is an additional mechanism to reduce the downtime in the event of a failure.

You can enable high availability for your database during provisioning for an additional charge. When you provision high availability, we provision a standby of the same instance type. We continuously stream the transactions from the primary to the standby as they are committed. This provides an up-to-date copy that can be promoted to the primary in the event of a failure.

In order to provide high availability, the  control plane continuously monitors the health of both your primary and standby. If a health check fails against your primary, we run a deeper set of checks. If the primary does prove to be unhealthy, we initiate a failover to your standby.

During the failover, your connections will drop, so it is important you have reconnection logic within your database library (most ORMs and drivers support this out of the box). Your connections should automatically reconnect in a matter of seconds once the failover has completed. During a failover, your connection string does not change.

You can enable and disable HA from the resize page. This change takes place immediately as there is no impact to the performance of the primary cluster.

# Automatic disk resizes

Overutilizing storage on a Postgres server can be operationally dangerous because there might not be enough disk space for the server to recover in case of an emergency.

To prevent such situations, Crunchy Bridge will:

- warn team administrators by email when a cluster’s disk reaches **75%**, **80%**, and **85%** utilization
- trigger a disk resize automatically when a cluster’s disk reaches **90%** utilization

When an automatic resize is triggered, the new size is calculated based on the original size.

- Disks **smaller than 1 TB** will be increased by **25%** (e.g. 100 GB becomes 125 GB).
- Disks **larger than 1 TB** will be increased by **10%** (e.g. 1000 GB becomes 1100 GB).

An email notification will be sent to team administrators when an automatic resize is triggered.

### Risks

- [ ]