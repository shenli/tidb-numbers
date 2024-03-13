# Numbers every TiDB Developer should know

At Google, there was a document put together by Jeff Dean, the legendary engineer, called [Numbers every Engineer should know](http://brenocon.com/dean_perf.html). It’s really useful to have a similar set of numbers for TiDB developers to know that are useful for back-of-the envelope calculations. Here I share particular numbers I at PingCAP use, why the number is important and how to use it to your advantage.

## Data Ingestion

TiDB provides a tool named [TiDB-Lighting](https://docs.pingcap.com/tidb/stable/tidb-lightning-overview) (or Lightning) for data importing. There are two types of importing model:

* [Physical mode](https://docs.pingcap.com/tidb/stable/tidb-lightning-physical-import-mode-usage): convert csv files into RocksDB SST files and ingest the SST files into storage engine. This is good for importing data into a brand new table.
* [Logical mode](https://docs.pingcap.com/tidb/stable/tidb-lightning-logical-import-mode-usage): convert csv data into INSERT statement and use MySQL protocol to ingest data. This is good for importing data into a table which contains data.
  
For Physical import **200GiB/h per Lightning instance**, logical import **50GiB/h per Lightning instance**. Maximum Lightning instance number is 10.

### Two cases for reference

* A top fintech company imported 100T data within 31 hours (with 10 Lightning instances)
14.5TB data took 3h56m
* 2 tables, one with 2 columns, and another one with 4 columns
  - only have PK, no secondary index, and row size is about 2~2.3 KB
  - 10 Lightning instances

### Factors affecting ingesting speed
* Cluster configuration (how many instances/CPUs/disk)
* The schema of the data
    - number of secondary index (encoding and ingesting index data cost extra time)
    - If there is any unique index (uniqueness checking costs extra time)
* Number of Lightning instance (parallel ingestion)

## DDL (Schema Change)
### Indexing

#### Factors affecting indexing speed

* Cluster configuration (TiDB/TiKV instances/CPU/Disk)
* Tabel definition
    - Index type
    - unique or non-unique
    - single-column index or composed index
* Ideal cluster or busy cluster


Here is a benchmark comparison with Aurora MySQL of reference:

Note: there are improvements in v7.5/v8.0, Indexing should be 30% faster compared with v6.5.

TiDB: 
* v6.5
* TiDB Server:  (16c + 32G) * 2 
* TiKV Server:  (16c + 64G + 1T) * 9

Aurora MySQL:
* db.r5.8xlarge

## Columnar Storage Engine (TiFlash)
### Replication Lag
With TiKV and TiFlash, what does the replication lag look like?
Within sub-second if write load is not high, seconds to minutes for high write workload.

For example: with 2000 write TPS, in-place update random records in a huge table, the P99 lag is < 600ms.

### Query Performance
The comfortable zone of a single TiFlash node is tens of QPS.
For example: a TiFlash node with 1TB data, <50 qps would be OK.

If there are proper indexes with good selectivity, the QPS can be much higher. In this case, planner can leverage TiKV to do more efficient filtering with indexes.

## Capacity Planning
### Storage
* Single TiKV instance servers 4TB data.
* Keep the TiKV storage utilization under 80%.
* Data compressing rate of TiKV is 3~6.