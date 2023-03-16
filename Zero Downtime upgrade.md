# Zero Downtime upgrade

Created: March 11, 2023 4:59 PM
Created By: barak lagziel
Last Edited By: barak lagziel
Last Edited Time: March 11, 2023 5:43 PM
Type: Technical Spec

# Summary

Zero downtime upgrade is a feature that allows users to upgrade their database instance to a new version without experiencing any downtime. With this feature, users can upgrade their database instance to a new version while maintaining availability and without impacting their applications.

# Background

Zero downtime upgrade allows users to perform upgrades with minimal impact on their applications and without any downtime. By using read replicas to test and validate new versions, users can ensure that the upgrade is successful and that their applications will continue to function properly.

# Goals

1. Create a Read Replica: To perform a zero downtime upgrade, users first create a read replica of their existing database instance.
2. Upgrade the Read Replica: Users then upgrade the read replica to the new version of the database engine. This allows them to test the new version and ensure that it is compatible with their application.
3.  Test the New Version: Test the new version of the database engine to ensure compatibility with your application.
4. Promote the Read Replica: Once the read replica has been successfully upgraded and tested, users can promote it to become the new primary database instance. This process typically takes only a few minutes and can be done with minimal impact on the application.
5. Redirect Traffic: Redirect traffic from the old primary database instance to the new primary database instance.
6. Delete the Old Instance: Finally, users can delete the old database instance, which completes the upgrade process.

# Proposed Solution

One of the most difficult parts of any high-volume system to upgrade is the Database layer, as there are two ways to approach it:

1. Easy way – have an outage and do a backup and restore of databases.
2. The Right way – sync up a new database and switch over to it live.

 Many of the things that were painful are now problems that have been solved:

- Backup and restore of DB’s
- Creation of Read Replicas
- Database encryption

However, there are still places where it’s necessary to dive a bit deeper, and I found one of those recently.

### **Migrating to encrypted volumes with zero downtime**

With RDS, there is currently no built-in way to migrate from a non-encrypted to an encrypted volume so we had to go off-piste a bit.

Knowing that it is possible to migrate into RDS using MySQL/MariaDB standard replication – and that it is also possible to migrate out of RDS using the same technique – we figured it would be possible to do this internally between 2 differently configured RDS machines. It turns out we were correct, and here is how we did it.

### **Key components:**

- RDS Master DB – source-master
- RDS Read Only Slave – source-slave
- RDS New Master DB – target-master

### **Setup**

For testing I set up the following machines:

RDS Master DB:

```
Name: source-master
Not encrypted
User: master
Pwd: password
DBName: test
MariaDB 10.0.24
```

RDS Read Only Replica:

Setup via AWS console, so has the same configuration as the RDS Master DB.

```
Name: source-slave
```

The RDS Target Master DB was configured as follows:

```
Name: target-master
Encrypted
User: master
Pwd: password
DBName: test
MariaDB 10.1.23

```

### **Instructions**

1. Wait for source-master and source-slave to both have the Available status.
2. Open a MySQL connection to source-slave and run the following command so that it retains is logs for longer while doing this.:
    
    ```
    call mysql.rds_set_configuration('binlog retention hours', 24);
    ```
    
3. On source-master create a replication user (with more secure passwords)
    
    ```
    CREATE USER 'repl’@‘%’ IDENTIFIED BY 'pass';
    GRANT REPLICATION SLAVE ON *.* TO 'repl’@’%';
    ```
    
4. Open a MySQL connection to source-slave and run the commands:
    
    ```
    call mysql.rds_stop_replication;
    SHOW SLAVE STATUS;
    ```
    
5. Note down the values in Master_Log_File and Read_Master_Log_Pos as shown in the snippet:
    
    ```
    Slave_IO_State:
    Master_Host: 10.0.0.189
    Master_User: rdsrepladmin
    Master_Port: 3306
    Connect_Retry: 60
    Master_Log_File: mysql-bin-changelog.000013
    Read_Master_Log_Pos: 1246
    Relay_Log_File: relaylog.000002
    Relay_Log_Pos: 918
    Relay_Master_Log_File: mysql-bin-changelog.000013
    Slave_IO_Running: No
    .....
    Master_SSL_Crlpath:
    Using_Gtid: Slave_Pos
    Gtid_IO_Pos: 0-1385452505-37
    ```
    
6. Run a mysqldump from source-slave with the following command:
    
    ```
    mysqldump -h source-slave -u user -p password --single-transaction
      --routines --triggers --databases database1 database2
      | gzip > source-slave-dump.sql.gz
    ```
    
7. Import the DB dump to target-master:
    
    ```
    zcat source-slave-dump.sql.gz | mysql -h target-master -u user -p password
    ```
    
8. Open a MySQL connection to target-master and set the source as source-master using the following command:
    
    ```
    CALL mysql.rds_set_external_master (
    'source-master.xxxxxxx.ap-southeast-2.rds.amazonaws.com'
    , 3306
    , 'repl'
    , 'pass'
    , 'mysql-bin-changelog.000013'
    , 1246
    , 0
    );
    ```
    
9. Start replication by running on target-master and monitoring progress using:
    
    ```
    CALL mysql.rds_start_replication;
    SHOW SLAVE STATUS;
    ```
    
10. You can now safely delete the source-slave machine
11. You can now transition your writes over to the new master and proceed with near zero downtime (depending on your application)
12. When you want to disconnect the target-master from the source-master use the following:
    
    ```
    CALL mysql.rds_stop_replication;
    CALL mysql.rds_reset_external_master;
    ```
    

Updating a database schema is pretty easy if you can take your application offline. You shutdown the application, create a backup of the current database schema, perform all required update operations using tools like [Flyway](https://thorben-janssen.com/flyway-getting-started/) or [Liquibase](https://thorben-janssen.com/database-migration-with-liquibase-getting-started/), restart the application and hope that everything works fine. But that changes if your customers don’t accept any downtime. Simple changes, like removing a column or renaming a table, suddenly require a multi-step migration process. The reason for that is that high-available systems make heavy use of redundancy.

**Contents [[hide](https://thorben-janssen.com/update-database-schema-without-downtime/#)]**

- [1 Redundancy – A required evil](https://thorben-janssen.com/update-database-schema-without-downtime/#Redundancy_8211_A_required_evil)
    - [1.1 Rolling Updates](https://thorben-janssen.com/update-database-schema-without-downtime/#Rolling_Updates)
- [2 Multi-Step Migration Process](https://thorben-janssen.com/update-database-schema-without-downtime/#Multi-Step_Migration_Process)
    - [2.1 Backward-Compatible Operations](https://thorben-janssen.com/update-database-schema-without-downtime/#Backward-Compatible_Operations)
    - [2.2 Backward-Incompatible Operations](https://thorben-janssen.com/update-database-schema-without-downtime/#Backward-Incompatible_Operations)
- [3 Summary](https://thorben-janssen.com/update-database-schema-without-downtime/#Summary)

# **Redundancy – A required evil**

If you want to build a high-available system, you need to run at least 2 instances of every subsystem. So, in the simplest case, you need at least 2 instances of your application and 2 instances of your database server.

![https://thorben-janssen.com/wp-content/uploads/2018/07/RedundantDeployments.png](https://thorben-janssen.com/wp-content/uploads/2018/07/RedundantDeployments.png)

The redundancy of all subsystems provides a lot of benefits. The two most important ones are:

- It increases the number of parallel requests that your system can handle.
- It makes sure that your system is still up and running even if an instance of one of your subsystems isn’t available.

But they also create new challenges. I will not dive any deeper into topics like monitoring, tracing, load balancing, and fault tolerance. If you don’t have any experience with high-available systems, you should read about all of them. The good news is that there are several great tools and libraries available that help you solve these challenges.

### **Rolling Updates**

In this article, I want to focus on database schema migration for high-available systems. The redundancy of the application plays a critical role during the migration. It enables you to perform a rolling update.

The implementation of a rolling update depends on your technology stack. But the idea is always the same: You have a bunch of instances of a subsystem and you shutdown, update, and restart one instance after the other. While doing that, you run the old and the new version of your application in parallel. The Kubernetes documentation contains a nice, visual description of the [rolling update concept](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/).

# **Multi-Step Migration Process**

The rolling update adds a few requirements to your database migration. You no longer need to just adapt the database in the way it’s required by your application; you also need to do it in a way that the old and the new version of your application can work with the database. That means that all migrations need to be backward-compatible as long as you’re running at least one instance of the old version of your application. But not all operations, e.g., renaming or removing a column, are backward compatible. These operations require a multi-step process that enables you to perform the migration without breaking your system.

Let’s take a closer look at the backward-compatible operations first.

### **Backward-Compatible Operations**

Backward-compatible operations are all operations that change your database in a way that it can be used by the old and the new version of your application. That means that you can execute them during a migration step and don’t need to split them into multiple operations.

### **Add a table or a view**

Adding new tables or views doesn’t affect the old instances of your application. You can perform them without any risk. Just keep in mind that while you’re performing the rolling update, some users might trigger write operations on old application instances. These old instances, obviously, don’t write any data to the new tables. You might need to clean up your data and add the missing records to the new table after all application instances have been migrated.

### **Add a column**

It can be a little bit harder to add a new column. You don’t need to worry if you add a database column without a not null constraint. In that case, your operation is backward compatible, and you can simply add the column.

That’s not the case for columns with a not null constraint because it will contain null values for all existing records. You can easily fix that by providing a default value; please check your database documentation on how to do that. If you don’t want to define a default value, you need to execute 3 statements to add the column with the constraint:

1. Add the column without a default value and update all application instances.
2. Run a database script to fill that field in all existing records.
3. Add the not null constraint.

The good news is that you can execute all 3 statements within the same migration step.

### **Remove a column that’s not used by the old and the new version of your application**

Removing a database column that is neither accessed by the old nor the new version of your application is also a backward-compatible operation. No application is using that column anymore so there is also no application that could be affected by its removal.

### **Remove constraints**

The removal of the constraint itself is a backward-compatible operation. The old version of your application can still write to the database in the same way as it did before.

But you need to check if there are any old use case implementations that would break if any database record doesn’t fulfill the constraint. During the rolling update, instances of the new version of the application might write some records that do not comply with the no-longer-existing constraint. If that breaks any old code, you’re in trouble, and I don’t know any good way to solve it. You can’t remove the constraint because some reading use cases of the old version will break. You also can’t keep the constraint because some write operations of the new version will fail. Your only option is to remove the constraint and to roll-out the update quickly.

### **Backward-Incompatible Operations**

Backward-incompatible operations are the reason why I wrote this article. These are all the operations that change your database schema in a way that it can no longer be used by the old version of your application. You need to break these operations into a backward-compatible part which you perform before you update your application and a second part that you execute after you updated all application instances. In most cases, that requires you to add a new column or table in the first and to remove the old one in a later step.

![https://thorben-janssen.com/wp-content/uploads/2018/07/3-step-migration.png](https://thorben-janssen.com/wp-content/uploads/2018/07/3-step-migration.png)

This makes the migration process more complex than it would be if you didn’t perform a rolling, zero-downtime update. To make the migration process easier to execute and less error-prone, you should use a that performs automatic, version-based database updates. The two most popular ones are Flyway and Liquibase. I wrote a series of tutorials about both of them:

- [Getting Started with Flyway and Version-Based Database Migration](https://thorben-janssen.com/flyway-getting-started/)
- [Version-Based Database Migration with Liquibase – Getting Started](https://thorben-janssen.com/database-migration-with-liquibase-getting-started/)

And now, let’s take a look at some backward-incompatible operations and how you can split them into parts that don’t break your system.

### **Rename a column, a table or a view**

Renaming a column or table or view sounds simple, but it requires 3-4 steps if you want to use a rolling update that doesn’t cause any downtime. The steps needed for all 3 of them are identical. I, therefore, only explain how to rename a database column. In my experience, this is the most common operation.

The migration always follows the same concept, but the implementation differs based on the capabilities of your database. But more about that later. Let’s first take a look at an example.

The table *review* contains the column *comment* which I want to rename to *message*. This requires multiple steps. In the first one, you need to add the database column and initialize it with the data from the old column; then you need to update all application instances before you can remove the old column.

![https://thorben-janssen.com/wp-content/uploads/2018/07/Example-3-step-migration.png](https://thorben-janssen.com/wp-content/uploads/2018/07/Example-3-step-migration.png)

Unfortunately, the most complicated part isn’t the database migration itself, and it’s, therefore, not visible on this diagram. The main issues occur during the rolling update, which is between step 1 and the new version. While you’re updating your application instances, you’re running old and new versions of your application in parallel. The old version is still using the old database column, and the new one is using the new column. So, you need to make sure that both use the same data and that you don’t lose any write operations. There are 2 general ways to achieve that.

### **Option 1: Sync with database triggers**

The migration process is a little bit easier if your database supports triggers. So let’s start with this one:

1. [Add a column](https://thorben-janssen.com/update-database-schema-without-downtime/#addColumn) with the new name and the same data type as the old one. You then copy all data from the old column to the new one.You also need to add database triggers to keep both columns in sync so that neither the old nor the new version of your application works on outdated data.
2. Perform a rolling update of all application instances.
3. [Remove the old database column](https://thorben-janssen.com/update-database-schema-without-downtime/#removeUnused) and the database triggers.

If you update your database during application startup, step 1 and 2 are executed as 1 step.

### **Option 2: Sync programmatically**

Some databases don’t support triggers, and you need a different approach. In these cases, you need to perform 4 migration steps, and you might lose some write operations during the update if you don’t switch your application into a read-only mode.

1. [Add a column](https://thorben-janssen.com/update-database-schema-without-downtime/#addColumn) with the new name and the same data type as the old one. You then copy all data from the old column to the new one.
2. Make sure that the new version of your application reads from and writes to the old and the new database column. Let’s call this version *new1*.Please also keep in mind that there are still old instances of your application that don’t know about the new column and that can write new and update existing records at any time. As your database doesn’t synchronize the write operations, you need to do that in the code of version *new1*.After you’ve made sure that the *new1* version of your application can handle this situation, you can perform a rolling update of all application instances.
3. All of your application instances now run version *new1* which knows about the new database column. You can now perform a rolling update to application version *new2* which only uses the new database column.
4. [Remove the old database column](https://thorben-janssen.com/update-database-schema-without-downtime/#removeUnused).

Similar to the previous approach, you can reduce the number of required steps if you run the database migration during application startup. In that case, you can execute step 1 and 2 as 1 step.

### **Change the data type of a column**

You can change the data type of a column in almost the same way as you [rename the column](https://thorben-janssen.com/update-database-schema-without-downtime/#renameColumn). The only difference is that you also need to convert all values stored in the old column to the data type of the new column.

### **Remove a column or table or view that’s still used by the old version of your application**

I’m sorry to tell you that you can’t remove that column/table/view. At least not now. You first need to update your application so that there is no running instance of it that still uses it. After you’ve done that, you can [remove the no longer used column/table/view from your database](https://thorben-janssen.com/update-database-schema-without-downtime/#removeUnused).

- 

# Follow-up Tasks

What needs to be done next for this proposal? 

- [ ]