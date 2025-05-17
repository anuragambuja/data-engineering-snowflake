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
 
> ## Materialized Views
- A Materialized View is a pre-computed & persisted data set derived from a SELECT query.
- MVs are an enterprise edition and above serverless feature.
- MVs are updated via a background process ensuring data is current and consistent with the base table.
- MVs use compute resources to perform automatic background maintenance.
- MVs use storage to store query results, adding to the monthly storage usage for an account.
- MVs can be created on top of External Tables to improve their query performance.
- snowflake doesn't recommend creating materialized views on base tables with high churn. That is lots of inserts, updates, and deletes as it could consume a lot of credits to keep it up to date.
- Suspending a materialized view pauses the updates, and also makes it inaccessible
- Limitations:
  - Single Table
  - JOIN
  - UDF, HAVING, ORDER BY, LIMIT, WINDOW FUNCTIONS




 
> ## Clustering
- Clustering is a way of describing the distribution of a column's values.
- Clustering aims to co-locate data of the clustering key in the same micro-partitions.
- Clustering improves performance of queries that frequently filter or sort on the clustered keys.
- Clustering should be reserved for large tables in the multi-terabyte range.
- tables that are clustered well can increase the odds of micro-partition pruning. Pruning is the process of ignoring micro-partitions not required to compute the result of a query.
- Snowflake maintains the following clustering metadata for micro-partitions in a table:
  - Total Number of Micropartitions
  - Number of Overlapping Micro-partitions
  - Depth of Overlapping Micro-partitions. Depth is correlated with their average overlap, but measures how many micro-partitions it would have to look into to find one value.
- Higher the overlapping number of micro-partitions and clustering depth, the worse a column is clustered.

    ![image](https://github.com/user-attachments/assets/354767d7-63a7-438f-b4db-db8a91707384)

- The more frequently a table is queried, the more benefit you'll get from clustering. However, the more frequently a table changes, the higher the cost will be to maintain the clustering. Clustering is recommended for large tables which do not frequently change and are frequently queried. 
- Automatic Clustering
  - Snowflake supports specifying one or more table columns/expressions as a clustering key for a table.
  - Clustering keys can also be defined on materialized views.
  - Snowflake recommended a maximum of 3 or 4 columns (or expressions) per key
  - Columns used in common queries which perform filtering and sorting operations.
  - If column's cadinality is too low, it won't allow for effective pruning because the values will exist in many different micro partitions

- As DML operations are performed on a clustered table, the data in the table might become less clustered. Reclustering is a background process which transparently reorganizes data in the micro-partitions by the clustering key.
- Initial clustering and subsequent reclustering operations consume compute & storage credits.

> ## Search Optimization
- Search optimization service is a table level property aimed at improving the performance of selective point lookup queries. These typically return a single row or a small group of rows.
- The search optimization service speeds up equality searches.
- The search optimization service is an enterprise edition and higher feature.
- A background process creates and maintains a search access path to enable search optimization. The search access path records metadata about the entire table to understand where all of the data resides in the underlying micro partitions.
- The access path data structure requires space for each table on which search optimization is enabled. The larger the table, the larger the access path storage costs.


` SELECT NAME, ADDRESS FROM USERS WHERE USER_EMAIL = ‘semper.google.edu’;`



```
CREATE OR REPLACE MATERIALIZED VIEW MV1 AS SELECT COL1, COL2 FROM T1;
ALTER MATERIALIZED VIEW MV1 SUSPEND;
ALTER MATERIALIZED VIEW MV1 RESUME;

# The output of this command includes a column called refreshed on which references the time the materialized view was last updated.
# It also includes a column called behind by, which gives an indication of how outta sync a materialized view is from the base table was created on.
SHOW MATERIALIZED VIEWS LIKE 'MV1%';

SELECT system$clustering_information(‘table’,’(col1,col3)’);
```
