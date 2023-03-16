# Disaster Recovery

Created: March 11, 2023 5:43 PM
Created By: barak lagziel
Last Edited By: barak lagziel
Last Edited Time: March 13, 2023 8:02 AM
Type: Technical Spec

# Summary

Disaster recovery is a critical aspect of any database deployment, will offers several options for disaster recovery to ensure that your data is protected in the event of a disaster. Here are some of the key options available for disaster recovery:

# Background

When designing a disaster recovery plan, it's important to consider factors such as Recovery Time Objective (RTO) and Recovery Point Objective (RPO) to ensure that your recovery plan meets your business requirements. You should also test your disaster recovery plan regularly to ensure that it works as expected in the event of a disaster.

# Goals

1. Multi-AZ Deployment: Multi-AZ deployment is the simplest form of disaster recovery available in RDS. With Multi-AZ, RDS automatically creates a replica of your database in a different Availability Zone. If the primary database instance fails, RDS automatically fails over to the replica instance, ensuring that your data remains available with minimal downtime.
2. Cross-Region Replication: Another option for disaster recovery is to use cross-region replication to replicate your database to a different region. This provides an additional layer of protection against regional disasters that may impact your primary region.
3. Database Snapshots: RDS allows you to create manual or automated snapshots of your database at any time. These snapshots can be used to restore your database in the event of a disaster.
4. Read Replicas: Read replicas can also be used for disaster recovery by promoting them to be the new primary instance in the event of a disaster. Read replicas can also be used to offload read traffic from the primary instance, improving overall database performance.
5. AWS Backup: AWS Backup is a fully managed backup service that allows you to centrally manage and automate backups for your RDS databases. It provides a simple and reliable way to backup your databases and restore them in the event of a disaster.

# Proposed Solution

Ensuring your database is safe and available is a key tenet for Crunchy Bridge. Disaster recovery is automatically enabled for all databases, ensuring that your database can be recovered in the event of a failure or disaster.

Disaster recovery is the process of archiving your data to an additional data store that is typically cold storage. The cold storage offers 11 9s (99.999999999% of durability) providing strong reliability for not losing data.

Disaster recovery is accomplished by capturing a base backup from Postgres and streaming the WAL (write-ahead-log) continuously. In the event of a disaster, we will automatically restore from the most recent base backup and then replay the WAL to the most recent moment in time.

Since disaster recovery is enabled on all databases, there are no additional steps or actions you have to take.

## **[Recovery time](https://docs.crunchybridge.com/concepts/disaster-recovery/#recovery-time)**

Disaster recovery times are not a guaranteed time as the amount of WAL that must be restored correlates to your transaction volume, not your database size.

For a rough guidance: disaster recovery takes about 1 hr per 100 GB of data, though it can take longer at times if you have a high throughput transaction database.

## Multi-AZ

Multi-AZ Deployment is a feature that provides high availability and increased durability for database instances. With Multi-AZ, will automatically creates a synchronous replica of your primary database instance in a different Availability Zone (AZ). An Availability Zone is a distinct location within a region that is engineered to be isolated from failures in other Availability Zones.

The replica instance is kept in sync with the primary instance using synchronous replication, which means that any writes to the primary instance are automatically replicated to the replica instance. If the primary instance becomes unavailable due to an instance failure, storage failure, or other issue, RDS automatically fails over to the replica instance, which becomes the new primary instance.

The failover process typically takes just a few minutes, and RDS handles all of the work involved in promoting the replica instance to the primary instance. Once the failover is complete, clients can resume connecting to the database using the same endpoint.

Multi-AZ Deployment provides several benefits, including:

1. High Availability: With Multi-AZ, your database instance is automatically failed over to a replica instance in the event of a failure, ensuring that your data remains available with minimal downtime.
2. Durability: Multi-AZ deployments are designed to be highly durable, with multiple copies of your data stored across different Availability Zones.
3. Simplified Management: Need handles all of the work involved in managing the replica instance, including monitoring, failover, and maintenance tasks.
4. Improved Performance: In some cases, read performance can be improved by offloading read traffic to the replica instance, which can reduce the load on the primary instance.

It's important to note that Multi-AZ Deployment does not provide automatic scaling or load balancing. If you need to scale your database instance, you can do so manually by upgrading the instance size or adding read replicas.

## Cross-Region Replication

Cross-Region Replication allows you to replicate a database to a different AWS region for disaster recovery, reporting, or other purposes. With cross-region replication, RDS automatically creates a read replica of your primary database instance in a different region. The replica instance is kept in sync with the primary instance using asynchronous replication, which means that there may be a delay between when data is written to the primary instance and when it is replicated to the replica instance.

Cross-Region Replication provides several benefits, including:

1. Disaster Recovery: With a replica instance in a different region, you have an additional layer of protection against regional disasters that may impact your primary region. In the event of a disaster, you can promote the replica instance to be the new primary instance, ensuring that your data remains available with minimal downtime.
2. Reporting: The replica instance can be used for reporting and analytics purposes without impacting the performance of the primary instance. This can be particularly useful if you need to run resource-intensive queries or generate reports that require a large amount of processing power.
3. Data Localization: If you have regulatory or compliance requirements that mandate data localization, you can use cross-region replication to replicate your data to a region that meets those requirements.

It's important to note that cross-region replication may introduce additional latency due to the distance between the primary and replica instances. Additionally, cross-region replication may incur additional costs for data transfer and storage in the replica region.

## Database Snapshots

Database snapshots allow you to create a point-in-time backup of your database instance. When you create a snapshot, RDS takes a copy of the database instance's storage volume and saves it as a snapshot. The snapshot can be used to restore the database instance to a specific point in time, which can be useful for disaster recovery or for creating a copy of the database for testing or development purposes.

Database snapshots are stored in Amazon S3, which provides high durability and availability. You can create and manage snapshots using the RDS console, the AWS CLI, or the RDS API.

Here are some key features of database snapshots in RDS:

1. Point-in-time Recovery: Database snapshots allow you to recover your database instance to a specific point in time. You can choose any point within the retention period, which is configurable from 1 to 35 days.
2. Automated Snapshots: You can configure RDS to take automated snapshots of your database instance at regular intervals. This can help ensure that you always have a recent backup of your data.
3. Manual Snapshots: You can also create manual snapshots at any time. This can be useful if you want to create a backup before making major changes to your database, such as upgrading to a new database engine version.
4. Snapshot Sharing: You can share snapshots with other AWS accounts, which can be useful for collaborating with other teams or for migrating databases between accounts.
5. Copying Snapshots: You can copy snapshots to other regions or accounts. This can be useful for disaster recovery or for creating copies of your database for testing or development purposes.

It's important to note that while database snapshots are a powerful backup and recovery tool, they are not a substitute for a comprehensive disaster recovery strategy. In addition to snapshots, you should also consider using Multi-AZ deployment or cross-region replication to ensure high availability and durability for your database.

## Read replicas

Read replicas  are copies of a database instance that can be used to offload read traffic from the primary instance. When you create a read replica,  creates a new database instance that is a copy of the primary instance. The replica instance is kept in sync with the primary instance using asynchronous replication, which means that there may be a delay between when data is written to the primary instance and when it is replicated to the replica instance.

Read replicas provide several benefits, including:

1. Improved Performance: By offloading read traffic to a replica instance, you can improve the performance of your application. This is particularly useful for read-heavy workloads, such as reporting or analytics.
2. Scalability: Read replicas can also be used to scale out your database. As your read traffic increases, you can add additional replica instances to handle the load.
3. High Availability: Read replicas can be used as a standby instance in the event of a primary instance failure. If the primary instance becomes unavailable, you can promote a replica instance to be the new primary instance, ensuring that your data remains available with minimal downtime.
4. Cost Savings: By offloading read traffic to replica instances, you can reduce the load on your primary instance, which can help you avoid the need to scale up to a larger, more expensive instance type.

It's important to note that while read replicas can improve the performance and availability of your database, they do not provide the same level of durability as the primary instance. In addition to read replicas, you should also consider using Multi-AZ deployment or cross-region replication to ensure high availability and durability for your database.

![Untitled](Disaster%20Recovery%209a709287ca424a21b9b22cf0f4840d49/Untitled.png)

## AWS Backups

Need to use automated backups and manual snapshots to back up your database instances.

1. Automated Backups: RDS can automatically back up your database instance daily, which allows you to restore your database instance to any point in time within a retention period of up to 35 days. Automated backups are enabled by default when you create a new RDS instance, and you can modify the backup settings to suit your needs.
2. Manual Snapshots: In addition to automated backups, you can create manual snapshots of your RDS instance at any time. Manual snapshots are point-in-time backups that you create manually, and you can keep them for as long as you need them.

Here are some key features of backups in RDS:

1. Encryption: RDS backups are encrypted by default using AWS KMS. You can also use your own KMS key to encrypt your backups for added security.
2. Storage: Backups are stored in Amazon S3, which provides high durability and availability. You can also copy backups to other regions for disaster recovery or data migration purposes.
3. Restore: You can restore your database instance to any point in time within the retention period using automated backups or manual snapshots. You can also restore to a new RDS instance or to an existing instance.
4. Cross-Region Replication: You can replicate your RDS backups to another region using the Cross-Region Snapshot Copy feature. This can be useful for disaster recovery or for migrating databases to another region.

It's important to note that backups in RDS are not a substitute for a comprehensive disaster recovery strategy. In addition to backups, you should also consider using Multi-AZ deployment or read replicas to ensure high availability and durability for your database.

Break down the solution into key tasks and their estimated deadlines. 

- 

### Open Questions

Ask any unresolved questions about the proposed solution here.

- 

# Follow-up Tasks

What needs to be done next for this proposal? 

- [ ]