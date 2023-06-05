# Advance Delta Lake Concepts

<img src="../.gitbook/assets/file.excalidraw (2).svg" alt="" class="gitbook-drawing">

## Time Travel

Delta’s time travel capabilities simplify building data pipelines for the following use cases.

* Audit data changes

All the transactions for the table are stored within this table including the initial set of insertions, update, delete, merge, and inserts with schema modification. In our previous [employees](hands-on-delta-tables.md#creating-our-first-delta-table) table, we can see all the transactions using the below command\


```sql
DESCRIBE HISTORY employees;
```

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

* Query Older Version of Tables
  * `SELECT * FROM <table> TIMESTAMP AS OF "2023-01-01"`
  * `SELECT * FROM <table> VERSION AS OF 23`
  * `SELECT * FROM <table>@v23`
* Rollback Versions&#x20;
  * RESTORE TABLE \<table> TO TIMESTAMP AS OF "2023-01-01"
  * RESTORE TABLE \<table>@v23

## Optimize

Compacting small files. Data Lake can improve read speed by compacting small files into larger ones.

<img src="../.gitbook/assets/file.excalidraw (3).svg" alt="" class="gitbook-drawing">

## Z-order

Z-ordering is a strategy that allows related information to be stored together within the same set of files. This technique is employed by Delta Lake on Databricks to enhance data-skipping. Data Skipping is an optimization of queries with filter clauses that uses data skipping column statistics to find the set of parquet data files that need to be queried (and prune away files that do not match the filters and contain no rows the query cares about). That means that no filters effectively skips data skipping.

By organizing the data in a specific order, Delta Lake can significantly minimize the amount of data that needs to be read during query execution. To apply Z-ordering, you specify the columns to order on using the `ZORDER BY` clause.

<img src="../.gitbook/assets/file.excalidraw (1).svg" alt="" class="gitbook-drawing">

if we have a numerical column `id` in the data files, like shown above, by applying the Z order on this column, the first compacted file will contain values from 1 to 50, while the other one will contain values from 51 to 100, if we query an ID, say 42, Delta is sure now that ID 42 is in file number one, so it can easily skip the scanning for file number two, which will save a huge amount of time.&#x20;

## Vacuum

For deleting unused files or files which are no longer in the latest table state from the storage `VACUUM` command is used to do this operation.

```sql
-- vacuum files not required by versions older 
-- than the default retention period of 7 days
VACUUM <table>

-- vacuum files not required by versions more than 100 hours old
VACCUM <table> RETAIN 100 

```

`VACUUM` removes all files from directories not managed by Delta Lake, ignoring directories beginning with `_`. The ability to time travel back to a version older than the retention period is lost after running `VACUUM`. So, `VACUUM=NO TIME TRAVEL`

## Clone

We can create a copy of an existing Delta table at a specific version using the clone command. This is very useful to get data from a PROD environment to a STAGING one, or archive a specific version for regulatory reasons.

There are two types of clones:

* A **deep clone** is a clone that copies the source table data to the clone target in addition to the metadata of the existing table.&#x20;
* A **shallow clone** is a clone that does not copy the data files to the clone target. The table metadata is equivalent to the source. These clones are cheaper to create.&#x20;

Any changes made to either deep or shallow clones affect only the clones themselves and not the source table.

Shallow clone are pointers to the main table. Running a `VACUUM` may delete the underlying files and break the shallow clone.

```sql
-- shallow clone zero copy
CREATE TABLE IF NOT EXISTS employees_clone
  SHALLOW CLONE employees
  VERSION AS OF 2;
 
SELECT * FROM employees_clone;
```

```sql
-- deep clone copy data
CREATE TABLE IF NOT EXISTS employees_clone_deep
  DEEP CLONE employees;
 
SELECT * FROM employees_clone_deep;
```

## Partitioning Tables

Work in progress

## Deletion Vectors

Work in progress

