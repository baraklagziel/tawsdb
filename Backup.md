# Backup

Created: March 5, 2023 8:31 PM
Created By: barak lagziel
Last Edited By: barak lagziel
Last Edited Time: March 5, 2023 8:51 PM
Type: Technical Spec

# Summary

Database backups are essential for any database management system, including TimescaleDB, because they ensure that your data is protected against various risks such as hardware failures, software errors, human errors, malicious attacks, and disasters. Backups allow you to restore your database to a previous state in case of data loss or corruption, and to recover from unexpected events that affect the availability or integrity of your data.

In the context of TimescaleDB, backups are particularly important because it is designed to handle large-scale, high-performance, and time-series data workloads, which require fast and efficient data ingestion, analysis, and visualization. TimescaleDB provides advanced features such as hypertables, continuous aggregates, and data retention policies, which enhance the scalability, flexibility, and usability of time-series data, but also increase the complexity and criticality of backup operations.

# Background

Without proper backups, you risk losing valuable data that is crucial for your business or operations, and you may face legal, financial, or repetitional consequences. Therefore, it is important to implement a backup strategy that meets your backup and recovery goals, and to test and validate your backups regularly to ensure their reliability and effectiveness. TimescaleDB provides various backup and recovery options, such as logical backups, physical backups, and replication, which allow you to choose the most suitable approach for your use case and environment.

# Proposed Solution

## **Time scale DB  instance Backup:**

In summary, to implement backups for a TimescaleDB cluster, you need to choose a backup strategy, configure backup settings, set up a backup solution, test backup and restore procedures, and monitor backup status and performance. By following these steps, you can ensure the availability, reliability, and recoverability of your data, and minimize the risk of data loss or corruption.

To implement backups for a TimescaleDB cluster, you can use a combination of built-in and external tools and strategies. Here are the general steps you can follow:

1. **Choose a backup strategy**: Depending on your data size, availability requirements, and recovery objectives, you can choose between several backup strategies, such as full backups, incremental backups, continuous backups, or point-in-time backups. Each strategy has its trade-offs in terms of backup size, speed, frequency, and restore capabilities, so you should select the one that best suits your needs.
2. **Configure backup settings**: To enable backups for a TimescaleDB cluster, you need to configure the appropriate settings in the PostgreSQL configuration file, such as **`archive_mode`**, **`archive_command`**, **`wal_level`**, **`max_wal_size`**, **`checkpoint_timeout`**, and **`restore_command`**. These settings control how the database writes and archives WAL (Write-Ahead Log) files, which are used for recovery and replication.
3. **Set up a backup solution**: To automate and manage backups, you can use various backup solutions that integrate with TimescaleDB, such as pgBackRest, Barman, WAL-G, or pg_dump. These tools provide different features and trade-offs in terms of backup speed, compression, encryption, deduplication, and storage options. You can choose one or more of these tools based on your requirements and preferences.
4. **Test backup and restore procedures**: To ensure that your backup solution works as expected, you should regularly test your backup and restore procedures in a safe and isolated environment, such as a staging or development cluster. By testing backups, you can identify and fix potential issues before they affect production data, and verify that you can recover data and restore service in case of failures or disasters.
5. **Monitor backup status and performance**: To ensure the reliability and efficiency of your backup solution, you should monitor the backup status and performance regularly, and configure alerting and reporting mechanisms to notify you of any anomalies or errors. By monitoring backups, you can detect and resolve issues early, optimize backup settings and resources, and ensure that your backup solution meets your SLAs and compliance requirements.

## **Time scale DB backup instance with k8s:**

In summary, to implement database backup instance in Kubernetes, you need to create a backup container, configure backup settings, mount backup volumes, schedule backups, and monitor backup status and performance. By following these steps, you can ensure the availability, reliability, and recoverability of your data, and minimize the risk of data loss or corruption.

To implement database backup instance in Kubernetes, you can follow these general steps:

1. **Create a backup container**: The first step is to create a backup container that can access the database instance and perform backups. You can create this container as a separate deployment or pod in your Kubernetes cluster, and configure it to run backup scripts or backup tools that connect to the database instance and create backup files or snapshots.
2. **Configure backup settings**: You need to configure the backup settings in the database instance to enable backups and ensure that the backup container can access the necessary credentials and resources. You can configure backup settings using environment variables, config maps, or secrets in Kubernetes, and pass them to the database instance through the deployment or stateful set configuration.
3. **Mount backup volumes**: To store the backup files or snapshots, you need to mount backup volumes in the backup container and configure the appropriate storage class, access mode, and persistent volume claim (PVC) in Kubernetes. You can use various storage options in Kubernetes, such as local storage, network-attached storage (NAS), or cloud-based storage, depending on your requirements and budget.
4. **Schedule backups**: You can schedule backups using Kubernetes cron jobs, which allow you to specify the backup frequency, time, and duration. You can also configure backup retention policies, which determine how long to keep the backup files or snapshots before deleting them. By scheduling backups, you can automate the backup process and ensure that backups are performed regularly and reliably.
5. **Monitor backup status and performance**: To ensure that your backup instance works as expected, you should monitor the backup status and performance regularly, and configure alerting and reporting mechanisms to notify you of any anomalies or errors. By monitoring backups, you can detect and resolve issues early, optimize backup settings and resources, and ensure that your backup solution meets your SLAs and compliance requirements.

## Postgres has two mechanisms for backups:

- **Logical** backups contain the raw data in a more portable format (a set of SQL statements such as `CREATE table` and `INSERTs`).
- **Physical** backups are a byte for byte backup of the data on disk.

Read more about the difference between and logical and physical backups **[here](https://www.craigkerstiens.com/2017/09/03/postgres-backups-physical-vs-logical/)**.

## **[Logical backups](https://docs.crunchybridge.com/concepts/backups/#logical-backups)**

You can capture logical backups for your databases directly using **[pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)**.

## **[Physical backups](https://docs.crunchybridge.com/concepts/backups/#physical-backups)**

To perform physical backups of a TimescaleDB cluster, you can use the **`pg_basebackup`** utility, which is a PostgreSQL built-in tool that creates a binary copy of a PostgreSQL instance, including all databases, tables, indexes, and other objects. Since TimescaleDB is an extension of PostgreSQL, **`pg_basebackup`** can be used to create physical backups of a TimescaleDB cluster as well.

Here are the general steps to perform physical backups of a TimescaleDB cluster:

1. **Identify the primary node**: In a TimescaleDB cluster, there is one primary node and one or more standby nodes that replicate the data from the primary node. You need to identify the primary node, which is the node that serves write requests and is the source of truth for the cluster.
2. **Stop writes and flush data**: Before performing a backup, you need to stop all write operations on the primary node and wait until all pending transactions are committed or rolled back. You can use the **`pg_ctl`** utility to stop the PostgreSQL instance gracefully and ensure that all data is flushed to disk.
3. **Create a binary backup**: Once the primary node is stopped, you can use the **`pg_basebackup`** utility to create a binary backup of the entire PostgreSQL instance, including TimescaleDB. You need to specify the **`-xlog`** option to include the transaction log files, and the **`-format=plain`** option to create a plain-text backup file that can be compressed or encrypted later.
4. **Compress and encrypt the backup**: After creating the binary backup file, you can compress it using a compression utility such as gzip or bzip2 to reduce its size and save storage space. You can also encrypt the backup file using a cryptographic algorithm such as AES or RSA to protect it from unauthorized access or theft. Make sure to keep the encryption key safe and secure.
5. **Copy the backup to a remote location**: Finally, you need to copy the compressed and encrypted backup file to a remote location that is outside the TimescaleDB cluster and preferably in a different geographic location. You can use various methods to transfer the backup file, such as scp, rsync, or a cloud-based storage service.

By following these steps, you can create a physical backup of your TimescaleDB cluster that is durable, secure, and recoverable in case of a disaster or data loss event. It is recommended to perform regular backups according to your backup and recovery goals and to test and validate your backups regularly to ensure their reliability and effectiveness.

 For each cluster, a PostgreSQL base backup is captured each day and kept current by streaming the WAL every 60 seconds or 16MB (whichever comes first). A physical backup can be restored as a fork.

Backups are retained for a rolling 10-day period. If you require a longer retention time, you can use the fork feature to create individual backup snapshots. Talk to customer success about additional considerations with backups.

### Open Questions

Ask any unresolved questions about the proposed solution here.

- 

# Follow-up Tasks

What needs to be done next for this proposal? 

- [ ]