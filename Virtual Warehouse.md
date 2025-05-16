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
- A multi-cluster warehouse is a named group of virtual warehouses which can automatically scale in and out based on the number of concurrent users/queries.
  - MIN_CLUSTER_COUNT specifies the minimum number of warehouses for a multi-cluster warehouse.
  - MAX_CLUSTER_COUNT specifies the maximum number of warehouses for a multi-cluster warehouse.
  - Setting these two values the same will put the multicluster warehouse in MAXIMIZED mode.
  - Setting these two values differently will put the multicluster warehouse in AUTO-SCALE mode.
  - The total credit cost of a multi-cluster warehouse is the sum of all the individual running warehouses that make up that cluster.
  - Scaling Policy
    - Standard Scaling Policy
      - Scaling Out: When a query is queued a new warehouse will be added to the group immediately.
      - Scaling In: Every minute a background process will check if the load on the least busy warehouse can be redistributed to another warehouse. If this condition is met after 2 consecutive minutes a warehouse will be marked for shutdown.
    - Economy Scaling policy
      - Scaling Out: When a query is queued the system will estimate if there’s enough query load to keep a new warehouse busy for 6 minutes.
      - Scaling In: Every minute a background process will check if the load on the least busy warehouse can be redistributed to another warehouse. If this condition is met after 6 consecutive minutes a warehouse will be marked for shutdown.
   - Concurrency Behaviour Properties
     - MAX CONCURRENCY LEVEL: Specifies the number of concurrent SQL statements that can be executed against a warehouse before either it is queued or additional compute power is provided. Default is 8.
     - STATEMENT QUEUED TIMEOUT IN SECONDS: Specifies the time, in seconds, a SQL statement can be queued on a warehouse before it is aborted. The default is no timeout.
     - STATEMENT TIMEOUT IN SECONDS: It specifies the time, in seconds, after which any running SQL statement on a warehouse is aborted.


> ## Resource Monitors
- Resource Monitors are objects allowing users to set credit limits on user managed warehouses.
- Resource Monitors can be set on either the account or individual warehouse level.
- Limits can be set for a specified interval or data range.
- When limits are reached an action can be triggered, such as notify user or suspend warehouse.
- Resource Monitors can only be created by account administrators. 

> ## Query Acceleration Service
- The Query Acceleration Service is a feature which can be enabled on a virtual warehouse.
- It dynamically add serverless compute power to a warehouse when a complex query needs it.
- In contrast with multi-cluster warehouses, which are created and defined by us, as users, how additional compute resources are allocated but when the query acceleration service is completely controlled by Snowflake.
- Snowflake will analyze the query plan of a submitted query, and will offload fragments of it, which can be run in parallel to the dynamically requisitioned serverless compute.
- There are 2 primary factors dictating which queries can be accelerated:
  - Some part of the query must be able to be run in parallel, like a scan with an aggregation.
  - The number of partitions to be scanned, so the size of the data being queried.
- `QUERY_ACCELERATION_ELIGIBLE` VIEW and `ESTIMATE_QUERY_ACCELERATION` system function - are used to verify if a query is eligible to make use of QAS.

  ![image](https://github.com/user-attachments/assets/32414868-775a-42d6-af41-c024123befed)






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

ALTER WAREHOUSE MY_WH SET WAREHOUSE_SIZE=LARGE;

CREATE WAREHOUSE MY_MCW_1 MIN_CLUSTER_COUNT=1 MAX_CLUSTER_COUNT=4 SCALING_POLICY=STANDARD; // SCALING_POLICY=ECONOMY;


```

