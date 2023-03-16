# Visibility

Created: March 11, 2023 4:19 PM
Created By: barak lagziel
Last Edited By: barak lagziel
Last Edited Time: March 16, 2023 7:31 PM
Type: Technical Spec

# Summary

visibility refers to the ability to monitor and track the performance and health of the RDS instances and databases.

# Background

visibility in TAWSDB enables users to monitor the performance and health of the databases and instances, and take action proactively to avoid issues or fix them quickly if they occur.

- 

# Goals

 Will provides several tools and features for monitoring and visibility, including:

1. Amazon CloudWatch Metrics: CloudWatch Metrics provides performance metrics for RDS instances, such as CPU utilization, free storage space, and database connections. These metrics can be used to set alarms and trigger actions based on predefined thresholds.
2. Enhanced Monitoring: will provides enhanced monitoring, which collects operating system metrics and database-specific metrics at a higher resolution than CloudWatch Metrics. Enhanced Monitoring can be used to diagnose performance issues and identify trends.
3. Performance Insights: Performance Insights is a feature that provides a comprehensive view of the performance of the database. It helps identify the root cause of performance issues and provides actionable insights to improve the performance of the database.
4. Event Notifications: could send event notifications to Amazon SNS, SMS, or email when specific events occur, such as a failover, a snapshot creation, or an instance reboot.
5. Log Files: will generates log files for database instances, which can be used to troubleshoot issues and monitor database activity.

# Proposed Solution

## Datadog Quickstart

This quickstart guide will help you get a **[Datadog](https://docs.datadoghq.com/integrations/postgres/)** agent up and running with Container Apps on Crunchy Bridge. We recommend you start with the **[Container Apps Quickstart](https://docs.crunchybridge.com/container-apps/)** to get a general understanding of Container Apps.

### **[About Datadog](https://docs.crunchybridge.com/container-apps/datadog-quickstart/#about-datadog)**

Datadog collects metrics and **[logs](https://docs.crunchybridge.com/how-to/logging/)** for Postgres and other applications servers and hosts. For Postgres, the agent will monitor PostgreSQL states. You can visualize system metrics including load, cpu usage, i/o wait, memory, and network traffic. You can configure notifications about events and customize to your use cases inside the Datadog dashboard.

### **[Datadog with Postgres Container Apps Quickstart](https://docs.crunchybridge.com/container-apps/datadog-quickstart/#datadog-with-postgres-container-apps-quickstart)**

First, make sure you have Container Apps enabled. As a user with superuser privileges, run `CREATE EXTENSION pgpodman;` to install the extension in the `postgres` database of the desired cluster.

Next, find your cluster’s database URL to include as the `DATABASE_URL` parameter when you run the container. You can find the URL in the Connection section of the cluster dashboard (choose the `application` role), or by running `cb uri --role application <database name>` on the command line.

You’ll also need to locate your API key from Datadog to include as the `DD_API_KEY` parameter when you start the container.

Create a Datadog user in your database, and note the password to include as the `DD_INSTANCES_PASSWORD` parameter when you start the container:

📋

```
create user datadog with password 'Datadog-user-password';
grant pg_monitor to datadog;
grant SELECT ON pg_stat_database to datadog;
```

## **CloudWatch for visibility**

CloudWatch is a monitoring and observability service that collects and tracks metrics, logs, and events for AWS resources, including will.  provides several CloudWatch metrics that can be used to monitor the performance of the database instances, including CPU utilization, disk space usage, and database connections. These metrics can be used to create alarms and trigger notifications when specific thresholds are exceeded.

Will provides enhanced monitoring, which provides a higher resolution of operating system metrics and database-specific metrics than CloudWatch metrics. Enhanced monitoring uses an agent that runs on the RDS instances to collect metrics at a one-second frequency. These metrics are sent to CloudWatch and can be used to diagnose performance issues and identify trends.

In addition to metrics, will generates log files that can be sent to CloudWatch Logs for analysis and monitoring. Log files provide detailed information about database activity and can be used to troubleshoot issues and monitor database behavior.

Overall, will uses CloudWatch to provide visibility into the performance and health of the databases and instances. By leveraging CloudWatch metrics, enhanced monitoring, and log files, users can monitor RDS instances and databases proactively and take action when necessary to ensure optimal performance and availability.

## **Performance Insights**

1. Performance Dashboard: The Performance Dashboard provides a visual representation of the performance of the database instance over a selected time range. It includes graphs and charts that show metrics such as CPU utilization, I/O operations, and network throughput. Users can quickly identify performance bottlenecks and track changes over time.
2. Top SQL: Top SQL provides a list of the top SQL queries that are consuming the most resources on the database instance. Users can see the query text, the number of executions, the average execution time, and more. This information helps users identify slow-running queries and optimize them for better performance.
3. Wait Events: Wait Events provides information on the events that the database instance is waiting for, such as I/O operations or locks. Users can see the event type, the duration, and the number of occurrences. This information helps users identify the root cause of performance issues and take action to resolve them.
4. Recommendations: Recommendations provide actionable advice on how to improve the performance of the database instance. For example, recommendations may suggest increasing the size of the instance, optimizing SQL queries, or adjusting the configuration parameters. Recommendations are based on best practices and can help users optimize the performance of their database instances.

## Event Notifications

Event Notifications will provide a way for users to monitor the health and status of their RDS instances and databases. Event Notifications are triggered when certain events occur, such as when a backup completes, a database fails over, or a parameter group is modified. When an event occurs, RDS can send a notification via email, SMS, or other methods to inform users about the event.

Here are some key features of Event Notifications:

1. Types of Events: will supports a wide range of event categories, including backups, configuration changes, failovers, scaling, security, and more. Users can choose which categories of events to receive notifications for.
2. Notification Options: Users can choose to receive notifications via email, SMS, or other methods, such as SNS or Lambda functions. They can also customize the format of the notification message.
3. Subscription Filters: Subscription Filters allow users to filter events based on their attributes. For example, users can create filters to only receive notifications for specific database instances, regions, or event types.
4. Integration with CloudWatch: could Event Notifications can also be sent to CloudWatch Alarms, which allows users to take automated actions based on the events. For example, users can create an alarm to scale up the RDS instance when CPU utilization reaches a certain threshold.

Overall, RDS Event Notifications provide users with a way to stay informed about the health and status of their RDS instances and databases. By receiving timely notifications about events, users can take proactive actions to maintain the availability and reliability of their RDS resources.

## **Log Files**

 

Will generates log files for database instances that can be used for troubleshooting issues and monitoring database activity. RDS supports several types of log files, including:

1. Error Logs: Error logs contain information about errors that occurred during the operation of the database instance. These logs can help diagnose and resolve issues with the instance, such as failed queries, authentication failures, or disk space errors.
2. General Logs: General logs contain a record of all SQL statements executed on the database instance. These logs can help monitor database activity and identify slow-running queries or other performance issues.
3. Slow Query Logs: Slow query logs contain a record of all SQL queries that take longer than a specified amount of time to execute. These logs can help identify and optimize queries that are causing performance issues.
4. Audit Logs: Audit logs contain a record of all actions performed on the database instance, such as creating or deleting a user account, modifying security groups, or changing parameter groups. These logs can help track changes to the instance and identify potential security issues.

The log files can be accessed and downloaded with AWS CLI, or API. Log files can also be streamed to Amazon CloudWatch Logs, which allows users to monitor logs in real-time, set up alarms, and perform advanced analysis using CloudWatch Metrics.

Overall, RDS log files are an important tool for monitoring and troubleshooting issues with database instances. By analyzing log files, users can diagnose issues, optimize performance, and maintain the availability and reliability of their RDS resources.