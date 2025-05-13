## Virtual Warehouse
- A Virtual Warehouse is a named abstraction for a Massively Parallel Processing (MPP) compute cluster.
- Virtual Warehouses execute:
  - DQL operations (SELECT)
  - DML operations (UPDATE)
  - Data loading operations (COPY INTO)
- As a user you only interact with the named warehouse object not the underlying compute resources.
- Virtual Warehouses contain local SSD storage used to store raw data retrieved from the storage layer. It's often referred to as the warehouse cache and can make subsequent queries to a virtual warehouse faster to compute.
- Spin up and shut-down a virtually unlimited number of warehouses without resource contention. Virtual Warehouse configuration can be changed on-the-fly.
- Virtual Warehouses are created via the Snowflake UI or through SQL commands.
- Virtual Warehouse State
  - STARTED: Started, indicates that a virtual warehouse is currently active and it's ready to process queries. In this state, the virtual warehouse is consuming credits. By default when a Virtual Warehouse is created it is in the STARTED state.
  - SUSPENDED: The virtual warehouse is shut down and not currently consuming credits. Suspending a Virtual Warehouse puts it in the SUSPENDED state, removing the compute nodes from a warehouse.
  - RESIZING: A warehouse can be resized up or down at any time, even while running queries. The resizing status indicates when this process is currently in progress.

- Virtual Warehouse State Properties
  - AUTO SUSPEND: Specifies the number of seconds of inactivity after which a warehouse is automatically suspended. Setting zero or null means the warehouse will never suspend, by default when creating a warehouse, auto suspend is enabled and is set to 600 seconds.
  - AUTO RESUME: Specifies whether to automatically resume a warehouse when a SQL statement is submitted to it. If set to true, a warehouse effectively wakes up, if a query uses it while it's in the suspended state, by default, auto suspend is turned on.
  - INITIALLY SUSPENDED: Specifies whether the warehouse is created initially in the ‘Suspended’ state.

> ## Virtual Warehouse Sizes
- Virtual Warehouses can be created in 10 t-shirt sizes.
- Data loading does not typically require large Virtual Warehouses and sizing up does not guarantee increased data loading performance.

    ![image](https://github.com/user-attachments/assets/9898e22d-4232-47ae-ba81-56ab7717eea4)

- The first 60 seconds after a virtual warehouse is provisioned and running are always charged.
- Credit price is determined by region & Snowflake edition.
- Virtual Warehouses can be manually resized via the Snowflake UI or SQL commands.
- Resizing a running warehouse does not impact running queries. The additional compute resources are used for queued and new queries. Decreasing size of running warehouse removes compute resources from the warehouse and clears the warehouse cache.


> ## Resource Monitors
- Resource Monitors are objects allowing users to set credit limits on user managed warehouses.
- Resource Monitors can be set on either the account or individual warehouse level.
- Limits can be set for a specified interval or data range.
- When limits are reached an action can be triggered, such as notify user or suspend warehouse.
- Resource Monitors can only be created by account administrators. 


```sql
DROP WAREHOUSE MY_WAREHOUSE;
CREATE WAREHOUSE MY_MED_WH WAREHOUSE_SIZE=‘MEDIUM’;
ALTER WAREHOUSE MY_WH SUSPEND;
ALTER WAREHOUSE MY_WH_2 SET WAREHOUSE_SIZE=MEDIUM;
CREATE WAREHOUSE MY_WH_3 MIN_CLUSTER_COUNT=1 MAX_CLUSTER_COUNT=3 SCALING_POLICY=STANDARD;

CREATE WAREHOUSE MY_MED_WH WITH WAREHOUSE_SIZE=‘MEDIUM’;
ALTER WAREHOUSE MY_WH SUSPEND;
ALTER WAREHOUSE MY_WH RESUME;
CREATE WAREHOUSE MY_MED_WH AUTO_SUSPEND=300;
CREATE WAREHOUSE MY_MED_WH AUTO_RESUME=TRUE;
CREATE WAREHOUSE MY_MED_WH INITIALLY_SUSPENDED=TRUE;

CREATE RESOURCE MONITOR ANALYSIS_RM
  WITH CREDIT_QUOTA=100
  FREQUENCY=MONTHLY
  START_TIMESTAMP=‘2023-01-04 00:00 GMT'
  TRIGGERS ON 50 PERCENT DO NOTIFY
  ON 75 PERCENT DO NOTIFY
  ON 95 PERCENT DO SUSPEND
  ON 100 PERCENT DO SUSPEND_IMMEDIATE;

```

