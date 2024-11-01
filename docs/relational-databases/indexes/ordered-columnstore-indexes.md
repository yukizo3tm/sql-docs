---
title: "Performance tuning with ordered columnstore indexes"
description: "Learn more about how ordered columnstore indexes can benefit your query performance."
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: nibruno; xiaoyul
ms.date: 10/23/2024
ms.service: sql
ms.subservice: performance
ms.topic: conceptual
monikerRange: "=azuresqldb-current||>=sql-server-ver16||>=sql-server-linux-ver16||=azuresqldb-mi-current"
---
# Performance tuning with ordered clustered columnstore indexes

[!INCLUDE [sqlserver2022-asdb-asmi](../../includes/applies-to-version/sqlserver2022-asdb-asmi.md)]

<!-- Adapted from \azure-docs-pr\articles\synapse-analytics\sql-data-warehouse\performance-tuning-ordered-cci.md -->

By enabling efficient segment elimination, ordered clustered columnstore indexes (CCI) provide much faster performance by skipping large amounts of ordered data that don't match the query predicate. Loading data into an ordered CCI table can take longer than a non-ordered CCI table because of the data sorting operation, however queries can run faster afterwards with ordered CCI.

When users query a columnstore table, the optimizer checks the minimum and maximum values stored in each segment. Segments that are outside the bounds of the query predicate aren't read from disk to memory. A query can finish faster if the number of segments to read and their total size are small.

For ordered columnstore index availability, see [Ordered column index availability](columnstore-indexes-overview.md#ordered-columnstore-index-availability).

## Ordered vs. non-ordered clustered columnstore index

By default, for each table created without an index option, an internal component (index builder) creates a non-ordered clustered columnstore index (CCI) on it.  Data in each column is compressed into a separate CCI rowgroup segment.  There's metadata on each segment's value range, so segments that are outside the bounds of the query predicate aren't read from disk during query execution.  CCI offers the highest level of data compression and reduces the size of segments to read so queries can run faster. However, because the index builder doesn't sort data before compressing them into segments, segments with overlapping value ranges could occur, causing queries to read more segments from disk and take longer to finish.

When creating an ordered CCI, the SQL Database Engine sorts the existing data in memory by the order key(s) before the index builder compresses them into index segments. With sorted data, segment overlapping is reduced allowing queries to have a more efficient segment elimination and thus faster performance because the number of segments to read from disk is smaller. If all data can be sorted in memory at once, then segment overlapping can be avoided. Due to large tables in data warehouses, this scenario doesn't happen often.

To check the segment ranges for a column, run the following command with your table name and column name:

```sql
SELECT
o.name, pnp.index_id,
cls.row_count, pnp.data_compression_desc,
cls.segment_id,
cls.column_id,
cls.min_data_id, cls.max_data_id,
cls.max_data_id-cls.min_data_id as difference 

FROM sys.partitions AS pnp
 INNER JOIN sys.tables AS t ON pnp.object_id = t.object_id 
 INNER JOIN sys.objects AS o ON t.object_id = o.object_id
 INNER JOIN sys.column_store_segments AS cls ON pnp.partition_id = cls.partition_id
 INNER JOIN sys.columns as cols ON o.object_id = cols.object_id AND cls.column_id = cols.column_id
WHERE o.name = '<Table Name>' and cols.name = '<Column Name>'  
ORDER BY o.name, pnp.index_id, cls.min_data_id;
```

> [!NOTE]  
> In an ordered CCI table, the new data resulting from the same batch of DML or data loading operations are sorted within that batch, there is no global sorting across all data in the table. Users can REBUILD the ordered CCI to sort all data in the table. For a partitioned table, the REBUILD is done one partition at a time.  Data in the partition that is being rebuilt is "offline" and unavailable until the REBUILD is complete for that partition.

## Query performance

A query's performance gain from an ordered CCI depends on the query patterns, the size of data, how well the data is sorted, the physical structure of segments, and the DWU and resource class chosen for the query execution. Users should review all these factors before choosing the ordering columns when designing an ordered CCI table.

Queries with all these patterns typically run faster with ordered CCI.  

- The queries have equality, inequality, or range predicates
- The predicate columns and the ordered CCI columns are the same.

In this example, table `T1` has a clustered columnstore index ordered in the sequence of `Col_C`, `Col_B`, and `Col_A`.

```sql
CREATE CLUSTERED COLUMNSTORE INDEX MyOrderedCCI ON  T1
ORDER (Col_C, Col_B, Col_A);
```

The performance of query 1 and query 2 can benefit more from ordered CCI than the other queries, as they reference all the ordered CCI columns.

```sql
-- Query #1:

SELECT * FROM T1 WHERE Col_C = 'c' AND Col_B = 'b' AND Col_A = 'a';

-- Query #2

SELECT * FROM T1 WHERE Col_B = 'b' AND Col_C = 'c' AND Col_A = 'a';

-- Query #3
SELECT * FROM T1 WHERE Col_B = 'b' AND Col_A = 'a';

-- Query #4
SELECT * FROM T1 WHERE Col_A = 'a' AND Col_C = 'c';
```

## Data loading performance

The performance of data loading into an ordered CCI table is similar to a partitioned table. Loading data into an ordered CCI table can take longer than a non-ordered CCI table because of the data sorting operation, however queries can run faster afterwards with ordered CCI.

## Reduce segment overlapping

The number of overlapping segments depends on the size of data to sort, the available memory, and the maximum degree of parallelism (MAXDOP) setting during ordered CCI creation. The following strategies reduce segment overlapping when creating ordered CCI.

- Create ordered CCI with `OPTION (MAXDOP = 1)`. Each thread used for ordered CCI creation works on a subset of data and sorts it locally.  There's no global sorting across data sorted by different threads. Using parallel threads can reduce the time to create an ordered CCI but will generate more overlapping segments than using a single thread. Using a single threaded operation delivers the highest compression quality. You can specify MAXDOP with the `CREATE INDEX` or `CREATE TABLE` commands. For example:

```sql
CREATE TABLE Table1 WITH (DISTRIBUTION = HASH(c1), CLUSTERED COLUMNSTORE INDEX ORDER(c1) )
AS SELECT * FROM ExampleTable
OPTION (MAXDOP 1);
```

- Pre-sort the data by the sort key(s) before loading them into tables.

Here's an example of an ordered CCI table distribution that has zero segment overlapping following above recommendations. The ordered CCI is ordered on a **bigint** column with no duplicates.

:::image type="content" source="media/ordered-columnstore-indexes/perfect-sorting-example.png" alt-text="Screenshot of text data showing no segment overlapping.":::

## Create ordered CCI on large tables

Creating an ordered CCI is an offline operation.  For tables with no partitions, the data won't be accessible to users until the ordered CCI creation process completes. For partitioned tables, since the engine creates the ordered CCI partition by partition, users can still access the data in partitions where ordered CCI creation isn't in process. You can use this option to minimize the downtime during ordered CCI creation on large tables:

1. Create partitions on the target large table (called `Table_A`).
1. Create an empty ordered CCI table (called `Table_B`) with the same table and partition schema as `Table_A`.
1. Switch one partition from `Table_A` to `Table_B`.
1. Run `ALTER INDEX <Ordered_CCI_Index> ON <Table_B> REBUILD PARTITION = <Partition_ID>` to rebuild the switched-in partition on `Table_B`.  
1. Repeat step 3 and 4 for each partition in `Table_A`.
1. Once all partitions are switched from `Table_A` to `Table_B` and have been rebuilt, drop `Table_A`, and rename `Table_B` to `Table_A`.

## SQL Server 2022 capabilities

SQL Server 2022 (16.x) introduced ordered clustered columnstore indexes similar to the feature in Azure Synapse dedicated SQL pools.

- SQL Server 2022 (16.x) and later versions and other SQL platforms support clustered columnstore enhanced [segment elimination](/sql/relational-databases/indexes/columnstore-indexes-design-guidance) capabilities for string, binary, and guid data types, and the **datetimeoffset** data type for scale greater than two. Previously, this segment elimination applies to numeric, date, and time data types, and the **datetimeoffset** data type with scale less than or equal to two.  
- Currently, only SQL Server 2022 (16.x) and later versions and other SQL platforms support clustered columnstore rowgroup elimination for the prefix of `LIKE` predicates, for example `column LIKE 'string%'`. Segment elimination is not supported for non-prefix use of LIKE such as `column LIKE '%string'`.

For ordered columnstore index availability, see [Ordered column index availability](columnstore-indexes-overview.md#ordered-columnstore-index-availability).

For more information, see [What's New in Columnstore Indexes](/sql/relational-databases/indexes/columnstore-indexes-what-s-new).

For information on ordered columnstore indexes in dedicated SQL pools in Azure Synapse Analytics, see [Performance tuning with ordered clustered columnstore indexes](ordered-columnstore-indexes.md).

## Examples

**A. To check for ordered columns and order ordinal:**

```sql
SELECT object_name(c.object_id) table_name, c.name column_name, i.column_store_order_ordinal
FROM sys.index_columns i
JOIN sys.columns c ON i.object_id = c.object_id AND c.column_id = i.column_id
WHERE column_store_order_ordinal <>0;
```

**B. To change column ordinal, add or remove columns from the order list, or to change from CCI to ordered CCI:**

```sql
CREATE CLUSTERED COLUMNSTORE INDEX InternetSales ON dbo.InternetSales
ORDER (ProductKey, SalesAmount)
WITH (DROP_EXISTING = ON);
```

## Related content

- [Columnstore Index Design Guidelines](../../relational-databases/sql-server-index-design-guide.md#columnstore_index)
- [Columnstore indexes - Data loading guidance](columnstore-indexes-data-loading-guidance.md)
- [Get started with Columnstore for real-time operational analytics](get-started-with-columnstore-for-real-time-operational-analytics.md)
- [Columnstore indexes in data warehousing](columnstore-indexes-data-warehouse.md)
- [Optimize index maintenance to improve query performance and reduce resource consumption](reorganize-and-rebuild-indexes.md)
- [Columnstore Index Architecture](../../relational-databases/sql-server-index-design-guide.md#columnstore_index)
- [CREATE INDEX (Transact-SQL)](../../t-sql/statements/create-index-transact-sql.md)
- [ALTER INDEX (Transact-SQL)](../../t-sql/statements/alter-index-transact-sql.md)
