---
title: The neon extension
subtitle: An extension for Neon-specific statistics including the Local File Cache hit
  ratio
enableTableOfContents: true
updatedOn: '2025-06-30T11:30:21.888Z'
---

The `neon` extension provides functions and views designed to gather Neon-specific metrics.

- [The `neon_stat_file_cache` view](#the-neon_stat_file_cache-view)
- [Views for Neon internal use](#views-for-neon-internal-use)

## The neon_stat_file_cache view

The `neon_stat_file_cache` view provides insights into how effectively your Neon compute's Local File Cache (LFC) is being used.

## What is the Local File Cache?

Neon computes have a Local File Cache (LFC), which is a layer of caching that stores frequently accessed data in the local memory of the Neon compute. Like Postgres [shared buffers](/docs/reference/glossary#shared-buffers), the LFC reduces latency and improves query performance by minimizing the need to fetch data from Neon storage. The LFC acts as an add-on or extension of Postgres shared buffers. In Neon computes, the `shared_buffers` parameter [scales with compute size](/docs/reference/compatibility#parameter-settings-that-differ-by-compute-size). The LFC extends the cache memory to approximately 75% of your compute's RAM. To view the LFC size for each Neon compute size, see [How to size your compute](/docs/manage/computes#how-to-size-your-compute).

When data is requested, Postgres checks shared buffers first, then the LFC. If the requested data is not found in the LFC, it is read from Neon storage. Shared buffers and the LFC both cache your most recently accessed data, but they may not cache exactly the same data due to different cache eviction patterns. The LFC is also much larger than shared buffers, so it stores significantly more data.

## Monitoring Local File Cache usage

You can monitor Local File Cache (LFC) usage by installing the `neon` extension on your database and querying the [neon_stat_file_cache](/docs/) view or [using EXPLAIN ANALYZE](#view-lfc-metrics-with-explain-analyze). Additionally, you can monitor the [Local file cache hit rate](/docs/introduction/monitoring-page#local-file-cache-hit-rate) graph on the **Monitoring** page in the Neon console.

## neon_stat_file_cache view

The `neon_stat_file_cache` view includes the following metrics:

- `file_cache_misses`: The number of times the requested page block is not found in Postgres shared buffers or the LFC. In this case, the page block is retrieved from Neon storage.
- `file_cache_hits`: The number of times the requested page block was not found in Postgres shared buffers but was found in the LFC.
- `file_cache_used`: The number of times the LFC was accessed.
- `file_cache_writes`: The number of writes to the LFC. A write occurs when a requested page block is not found in Postgres shared buffers or the LFC. In this case, the data is retrieved from Neon storage and then written to shared buffers and the LFC.
- `file_cache_hit_ratio`: The percentage of database requests that are served from the LFC rather than Neon storage. This is a measure of cache efficiency, indicating how often requested data is found in the cache. A higher cache hit ratio suggests better performance, as accessing data from memory is faster than accessing data from storage. The ratio is calculated using the following formula:

  ```
  file_cache_hit_ratio = (file_cache_hits / (file_cache_hits + file_cache_misses)) * 100
  ```

  For OLTP workloads, you should aim for a cache hit ratio of 99% or better. However, the ideal cache hit ratio depends on your specific workload and data access patterns. In some cases, a slightly lower ratio might still be acceptable, especially if the workload involves a lot of sequential scanning of large tables where caching might be less effective. If you find that your cache hit ration is quite low, your working set may not be fully or adequately in memory. In this case, consider using a larger compute with more memory. Please keep in mind that the statistics are for the entire compute, not specific databases or tables.

### Using the neon_stat_file_cache view

To use the `neon_stat_file_cache` view, install the `neon` extension on your database:

To install the extension on a database:

```sql
CREATE EXTENSION neon;
```

To connect to your database. You can find a connection string for your database on the Neon Dashboard.

```bash shouldWrap
psql postgresql://alex:AbC123dEf@ep-cool-darkness-123456.us-east-2.aws.neon.tech/dbname?sslmode=require&channel_binding=require
```

Issue the following query to view LFC usage data for your compute:

```sql
SELECT * FROM neon_stat_file_cache;
 file_cache_misses | file_cache_hits | file_cache_used | file_cache_writes | file_cache_hit_ratio
-------------------+-----------------+-----------------+-------------------+----------------------
           2133643 |       108999742 |             607 |          10767410 |                98.08
```

<Admonition type="note">
Local File Cache statistics represent the lifetime of your compute, from the last time the compute started until the time you ran the query. Be aware that statistics are lost when your compute stops and gathered again from scratch when your compute restarts. You'll only want to run the cache hit ratio query after a representative workload has been run. For example, say that you increased your compute size after seeing a cache hit ratio below 99%. Changing the compute size restarts your compute, so you lose all of your current usage statistics. In this case, you should run your workload before you try the cache hit ratio query again to see if your cache hit ratio improved.

Remember that Postgres checks shared buffers first before it checks your compute's Local File Cache. If you are only working with a small amount of data, queries may be served entirely from the shared buffers, resulting in no LFC hits.
</Admonition>

## View LFC metrics with EXPLAIN ANALYZE

You can also use `EXPLAIN ANALYZE` with the `FILECACHE` and `PREFETCH` options to view LFC cache hit and miss data, as well as prefetch statistics. Installing the `neon` extension is not required. For example:

```sql {5,6,11,12,15,16,20,21}
EXPLAIN (ANALYZE,BUFFERS,PREFETCH,FILECACHE) SELECT COUNT(*) FROM pgbench_accounts;

 Finalize Aggregate  (cost=214486.94..214486.95 rows=1 width=8) (actual time=5195.378..5196.034 rows=1 loops=1)
   Buffers: shared hit=178875 read=143691 dirtied=128597 written=127346
   Prefetch: hits=0 misses=1865 expired=0 duplicates=0
   File cache: hits=141826 misses=1865
   ->  Gather  (cost=214486.73..214486.94 rows=2 width=8) (actual time=5195.366..5196.025 rows=3 loops=1)
         Workers Planned: 2
         Workers Launched: 2
         Buffers: shared hit=178875 read=143691 dirtied=128597 written=127346
         Prefetch: hits=0 misses=1865 expired=0 duplicates=0
         File cache: hits=141826 misses=1865
         ->  Partial Aggregate  (cost=213486.73..213486.74 rows=1 width=8) (actual time=5187.670..5187.670 rows=1 loops=3)
               Buffers: shared hit=178875 read=143691 dirtied=128597 written=127346
               Prefetch: hits=0 misses=1865 expired=0 duplicates=0
               File cache: hits=141826 misses=1865
               ->  Parallel Index Only Scan using pgbench_accounts_pkey on pgbench_accounts  (cost=0.43..203003.02 rows=4193481 width=0) (actual time=0.574..4928.995 rows=3333333 loops=3)
                     Heap Fetches: 3675286
                     Buffers: shared hit=178875 read=143691 dirtied=128597 written=127346
                     Prefetch: hits=0 misses=1865 expired=0 duplicates=0
                     File cache: hits=141826 misses=1865
```

### PREFETCH option

The `PREFETCH` option provides information about Neon's prefetching mechanism, which predicts which pages will be needed soon and sends prefetch requests to the page server before the page is actually requested by the executor. This helps reduce latency by having data ready when it's needed. The PREFETCH option includes the following metrics:

- `hits` - Number of pages received from the page server before actually requested by the executor. Prefetch distance is controlled by the `effective_io_concurrency` parameter. The larger this value, the more likely the page server will complete the request before it's needed. However, it should not be larger than `neon.prefetch_buffer_size`.
- `misses` - Number of accessed pages that were not prefetched. Prefetch is not implemented for all plan nodes, and even for supported nodes (like sequential scan), some mispredictions can occur.
- `expired` - Pages that were updated since the prefetch request was sent, or results that weren't used because the executor didn't need the page (for example, due to a `LIMIT` clause in the query).
- `duplicates` - Multiple prefetch requests for the same page. For some nodes like sequential scan, predicting next pages is straightforward. However, for index scans that prefetch referenced heap pages, index entries can have multiple references to the same heap page, resulting in duplicate prefetch requests.

### FILECACHE option

The `FILECACHE` option provides information about the Local File Cache (LFC) usage during query execution:

- `hits` - Number of accessed pages found in the LFC.
- `misses` - Number of accessed pages not found in the LFC.

## Views for Neon internal use

The `neon` extension is installed by default to a system-owned `postgres` database in each Neon project. The `postgres` database includes functions and views owned by the Neon system role (`cloud_admin`) that are used to collect statistics. This data helps the Neon team enhance the Neon service.

**Views**:

```sql
postgres=> \dv
                    List of relations
 Schema |            Name            | Type |    Owner
--------+----------------------------+------+-------------
 public | local_cache                | view | cloud_admin
 public | neon_backend_perf_counters | view | cloud_admin
 public | neon_lfc_stats             | view | cloud_admin
 public | neon_perf_counters         | view | cloud_admin
 public | neon_stat_file_cache       | view | cloud_admin
```

**Functions**:

```sql
postgres=> \df
                                                                          List of functions
 Schema |                 Name                 | Result data type |                                    Argument data types                                    | Type
--------+--------------------------------------+------------------+-------------------------------------------------------------------------------------------+------
 public | approximate_working_set_size         | integer          | reset boolean                                                                             | func
 public | approximate_working_set_size_seconds | integer          | duration integer DEFAULT NULL::integer                                                    | func
 public | backpressure_lsns                    | record           | OUT received_lsn pg_lsn, OUT disk_consistent_lsn pg_lsn, OUT remote_consistent_lsn pg_lsn | func
 public | backpressure_throttling_time         | bigint           |                                                                                           | func
 public | get_backend_perf_counters            | SETOF record     |                                                                                           | func
 public | get_perf_counters                    | SETOF record     |                                                                                           | func
 public | local_cache_pages                    | SETOF record     |                                                                                           | func
 public | neon_get_lfc_stats                   | SETOF record     |                                                                                           | func
 public | pg_cluster_size                      | bigint           |                                                                                           | func
```

<NeedHelp/>
