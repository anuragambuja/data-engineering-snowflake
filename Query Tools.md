> ## Query Performance Analysis Tools
  - History Tab:
  - History Tab displays query history for the last 14 days.
  - Users can view other users queries but cannot view their query results. 
- Query History Views and Table Functions:
  - for programmatic analysis of query performance.
  - The query history view in the account usage schema in the Snowflake database gives us the ability to investigate programmatically the query history within the last year.
  - By default, only the account admin has permissions to read this view.
  - The query history view has a latency of about 45 minutes. This means if a query is executed, there's no guarantee it will appear until at least 45 minutes later.
  - the information schema table function query history only has data going back seven days.
- Query Profile:
  - which provides a breakdown of the steps involved in executing a query to pinpoint any issues.

> ## Caching
  ![image](https://github.com/user-attachments/assets/7d932b17-73a1-4e23-a9bc-6f5d1a7351a7)

- Results Cache ( result set cache, or the 24 hour cache, or query result cache):
  - It stores the results of queries for reuse by subsequent queries.
  - Each time a persisted result is reused, the 24 hour retention period is extended up to a maximum of 31 days, after which the result is purged from the cache.
  - To reuse a result:
    - New query exactly matches previous query.
    - The underlying table data has not changed.
    - The same role is used as the previous query.
    - If time context functions are used, such as CURRENT_TIME(), the result cache will not be used.
  - Result reuse can be disabled using the session parameter USE_CACHED_RESULT. 
- Metadata Cache ( metadata store or cloud services layer):
  - Snowflake has a high availability metadata store which maintains metadata object information and statistics. e.g. schema of the table or the count in the table, micro partitions of tables, clustering information
  - Some queries can be completed purely using the metadata, not requiring a running virtual warehouse. eg. `SELECT COUNT(*) FROM MY_TABLE;`
- Warehouse Cache (Local Disk Cache, or SSD cache, or data cache, or raw data cache) :
  - refers to the local SSD storage of the nodes in a virtual warehouse cluster.
  - These keep a local version of the raw micro partitions retrieved from the storage layer used for computing the results of queries.
  - The Local Disk Cache can be reused by subsequent queries to the same warehouse.
  - The larger the virtual warehouse the greater the local cache.
  - It is purged when the virtual warehouse is resized, suspended or dropped.
  - Can be used partially, retrieving the rest of the data required for a query from remote storage. 
 
