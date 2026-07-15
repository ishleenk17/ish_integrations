# Supabase Integration (OTel)

Collect metrics from [Supabase](https://supabase.com) using the OpenTelemetry Prometheus receiver.

## Overview

This integration scrapes metrics from the Supabase Metrics API (Prometheus-compatible) using the OTel Collector's Prometheus receiver. Metrics are organized into three data streams:

- **Node** (`supabase_integration_otel.node`) — Infrastructure metrics: CPU, memory, load, disk I/O, filesystem, network, vmstat, system
- **PostgreSQL** (`supabase_integration_otel.postgres`) — Database metrics: size, connections, transactions, cache, WAL, replication, bgwriter, statements, activity
- **Services** (`supabase_integration_otel.services`) — Application metrics: Supavisor connection pooler, PostgREST API, GoTrue auth, Realtime subscriptions, Storage

## Prerequisites

| Requirement | Details |
|---|---|
| **Supabase plan** | Pro, Team, or Enterprise (Metrics API is not available on Free tier) |
| **Elastic Stack** | 9.4.0+ |
| **Input package** | `prometheus_input_otel` (installed automatically as a dependency) |

## Setup

1. **Get your Supabase credentials**:
   - Go to your Supabase dashboard → **Settings** → **API**
   - Copy the **Project URL** (e.g., `abcdefghijkl.supabase.co`)
   - Copy the **service_role** key (under "Project API keys")

2. **Add the integration in Kibana**:
   - Go to **Management** → **Integrations** → search for "Supabase"
   - Click **Add Supabase**
   - Fill in the same credentials for each data stream:
     - **Targets**: `<your-project-ref>.supabase.co:443`
     - **Username**: `service_role`
     - **Service Role Key**: paste your service_role JWT

3. **Verify data**:
   - Go to **Discover** and filter on any of:
     - `data_stream.dataset: "supabase_integration_otel.node.otel"`
     - `data_stream.dataset: "supabase_integration_otel.postgres.otel"`
     - `data_stream.dataset: "supabase_integration_otel.services.otel"`

## Dashboards

Kibana dashboards are provided by the **Supabase Dashboards** (`supabase_otel`) content package, which is declared as a dependency and installed automatically.

## Metrics Reference

Metrics are scraped from the Supabase Metrics API endpoint:

```
GET https://<project-ref>.supabase.co/customer/v1/privileged/metrics
```

Authentication uses HTTP Basic Auth with `service_role` as the username and the service_role JWT as the password.

See the [Supabase Metrics documentation](https://supabase.com/docs/guides/telemetry/metrics) for full details on available metrics.

### Node Metrics

**Exported fields**

| Field | Description | Type | Unit | Metric Type |
|---|---|---|---|---|
| @timestamp | Event timestamp. | date |  |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |  |
| data_stream.type | Data stream type. | constant_keyword |  |  |
| supabase.labels.instance | Supabase project instance identifier. | keyword |  |  |
| supabase.labels.job | Prometheus scrape job name. | keyword |  |  |
| supabase.node.boot_time_seconds | Node boot time (unix epoch). | double | s | gauge |
| supabase.node.context_switches.count | Total context switches. | double |  | counter |
| supabase.node.cooling_device_cur_state | Current throttle state of the cooling device. | double |  | gauge |
| supabase.node.cpu.guest_seconds.count | Seconds the CPUs spent in guests (VMs) for each mode. | double | s | counter |
| supabase.node.cpu.seconds.count | Seconds the CPUs spent in each mode (user, system, idle, iowait, etc.). | double | s | counter |
| supabase.node.disk.discards_completed.count | Total completed discard (TRIM) operations. | double |  | counter |
| supabase.node.disk.discards_merged.count | Total merged discard operations. | double |  | counter |
| supabase.node.disk.io_now | Number of I/Os currently in progress. | double |  | gauge |
| supabase.node.disk.io_time_seconds.count | Total seconds spent doing I/Os. | double | s | counter |
| supabase.node.disk.io_time_weighted_seconds.count | Weighted time spent doing I/Os. | double | s | counter |
| supabase.node.disk.read_bytes.count | Total bytes read from disk. | double | byte | counter |
| supabase.node.disk.read_time_seconds.count | Total seconds spent on reads. | double | s | counter |
| supabase.node.disk.reads_completed.count | Total number of reads completed. | double |  | counter |
| supabase.node.disk.reads_merged.count | Total merged read operations. | double |  | counter |
| supabase.node.disk.write_time_seconds.count | Total seconds spent on writes. | double | s | counter |
| supabase.node.disk.writes_completed.count | Total number of writes completed. | double |  | counter |
| supabase.node.disk.writes_merged.count | Total merged write operations. | double |  | counter |
| supabase.node.disk.written_bytes.count | Total bytes written to disk. | double | byte | counter |
| supabase.node.entropy_available_bits | Available entropy bits. | double |  | gauge |
| supabase.node.filefd.allocated | Allocated file descriptors. | double |  | gauge |
| supabase.node.filefd.maximum | Maximum file descriptors. | double |  | gauge |
| supabase.node.filesystem.available_bytes | Filesystem space available to non-root users in bytes. | double | byte | gauge |
| supabase.node.filesystem.device_error | Whether there was an error getting device stats (0/1). | double |  | gauge |
| supabase.node.filesystem.files | Total file nodes (inodes). | double |  | gauge |
| supabase.node.filesystem.files_free | Free file nodes (inodes). | double |  | gauge |
| supabase.node.filesystem.free_bytes | Filesystem free space in bytes. | double | byte | gauge |
| supabase.node.filesystem.readonly | Whether the filesystem is read-only (0/1). | double |  | gauge |
| supabase.node.filesystem.size_bytes | Total filesystem size in bytes. | double | byte | gauge |
| supabase.node.forks.count | Total process forks. | double |  | counter |
| supabase.node.hwmon_temp_celsius | Hardware temperature sensor reading in Celsius. | double |  | gauge |
| supabase.node.interrupts.count | Total interrupts serviced. | double |  | counter |
| supabase.node.load1 | 1-minute load average. | double |  | gauge |
| supabase.node.load15 | 15-minute load average. | double |  | gauge |
| supabase.node.load5 | 5-minute load average. | double |  | gauge |
| supabase.node.memory.active_anon_bytes | Active anonymous memory pages in bytes. | double | byte | gauge |
| supabase.node.memory.active_bytes | Active memory in bytes. | double | byte | gauge |
| supabase.node.memory.active_file_bytes | Active file-backed memory pages in bytes. | double | byte | gauge |
| supabase.node.memory.anon_huge_pages_bytes | Anonymous huge pages in bytes. | double | byte | gauge |
| supabase.node.memory.anon_pages_bytes | Anonymous pages in bytes. | double | byte | gauge |
| supabase.node.memory.bounce_bytes | Bounce buffer memory in bytes. | double | byte | gauge |
| supabase.node.memory.buffers_bytes | Memory used for buffers in bytes. | double | byte | gauge |
| supabase.node.memory.cached_bytes | Memory used for cache in bytes. | double | byte | gauge |
| supabase.node.memory.commit_limit_bytes | Commit limit in bytes. | double | byte | gauge |
| supabase.node.memory.committed_as_bytes | Committed address space in bytes. | double | byte | gauge |
| supabase.node.memory.dirty_bytes | Dirty pages awaiting writeback in bytes. | double | byte | gauge |
| supabase.node.memory.hardware_corrupted_bytes | Hardware-corrupted memory in bytes. | double | byte | gauge |
| supabase.node.memory.huge_pages_free | Free huge pages. | double |  | gauge |
| supabase.node.memory.huge_pages_rsvd | Reserved huge pages. | double |  | gauge |
| supabase.node.memory.huge_pages_surp | Surplus huge pages. | double |  | gauge |
| supabase.node.memory.huge_pages_total | Total huge pages. | double |  | gauge |
| supabase.node.memory.hugepagesize_bytes | Default huge page size in bytes. | double | byte | gauge |
| supabase.node.memory.inactive_anon_bytes | Inactive anonymous memory pages in bytes. | double | byte | gauge |
| supabase.node.memory.inactive_bytes | Inactive memory in bytes. | double | byte | gauge |
| supabase.node.memory.inactive_file_bytes | Inactive file-backed memory pages in bytes. | double | byte | gauge |
| supabase.node.memory.kernel_stack_bytes | Kernel stack memory in bytes. | double | byte | gauge |
| supabase.node.memory.mapped_bytes | Memory-mapped files in bytes. | double | byte | gauge |
| supabase.node.memory.mem_available_bytes | Available memory in bytes. | double | byte | gauge |
| supabase.node.memory.mem_free_bytes | Free memory in bytes. | double | byte | gauge |
| supabase.node.memory.mem_total_bytes | Total memory in bytes. | double | byte | gauge |
| supabase.node.memory.mlocked_bytes | Locked memory in bytes. | double | byte | gauge |
| supabase.node.memory.nfs_unstable_bytes | NFS unstable pages in bytes. | double | byte | gauge |
| supabase.node.memory.page_tables_bytes | Memory used by page tables in bytes. | double | byte | gauge |
| supabase.node.memory.percpu_bytes | Per-CPU allocations in bytes. | double | byte | gauge |
| supabase.node.memory.s_reclaimable_bytes | Reclaimable slab memory in bytes. | double | byte | gauge |
| supabase.node.memory.s_unreclaim_bytes | Unreclaimable slab memory in bytes. | double | byte | gauge |
| supabase.node.memory.shmem_bytes | Shared memory in bytes. | double | byte | gauge |
| supabase.node.memory.slab_bytes | Kernel slab allocator memory in bytes. | double | byte | gauge |
| supabase.node.memory.swap_cached_bytes | Swap cached in bytes. | double | byte | gauge |
| supabase.node.memory.swap_free_bytes | Free swap in bytes. | double | byte | gauge |
| supabase.node.memory.swap_total_bytes | Total swap in bytes. | double | byte | gauge |
| supabase.node.memory.unevictable_bytes | Unevictable memory in bytes. | double | byte | gauge |
| supabase.node.memory.vmalloc_chunk_bytes | Largest contiguous vmalloc block in bytes. | double | byte | gauge |
| supabase.node.memory.vmalloc_total_bytes | Total vmalloc address space in bytes. | double | byte | gauge |
| supabase.node.memory.vmalloc_used_bytes | Used vmalloc space in bytes. | double | byte | gauge |
| supabase.node.memory.writeback_bytes | Pages being written back in bytes. | double | byte | gauge |
| supabase.node.memory.writeback_tmp_bytes | FUSE writeback temp in bytes. | double | byte | gauge |
| supabase.node.network.receive_bytes.count | Total network bytes received. | double | byte | counter |
| supabase.node.network.receive_compressed.count | Total compressed packets received. | double |  | counter |
| supabase.node.network.receive_drop.count | Total received packets dropped. | double |  | counter |
| supabase.node.network.receive_errors.count | Total network receive errors. | double |  | counter |
| supabase.node.network.receive_fifo.count | Total receive FIFO errors. | double |  | counter |
| supabase.node.network.receive_frame.count | Total frame alignment errors on receive. | double |  | counter |
| supabase.node.network.receive_multicast.count | Total multicast packets received. | double |  | counter |
| supabase.node.network.receive_packets.count | Total network packets received. | double |  | counter |
| supabase.node.network.transmit_bytes.count | Total network bytes transmitted. | double | byte | counter |
| supabase.node.network.transmit_carrier.count | Total carrier errors on transmit. | double |  | counter |
| supabase.node.network.transmit_colls.count | Total collisions detected. | double |  | counter |
| supabase.node.network.transmit_compressed.count | Total compressed packets sent. | double |  | counter |
| supabase.node.network.transmit_drop.count | Total transmitted packets dropped. | double |  | counter |
| supabase.node.network.transmit_errors.count | Total network transmit errors. | double |  | counter |
| supabase.node.network.transmit_fifo.count | Total transmit FIFO errors. | double |  | counter |
| supabase.node.network.transmit_packets.count | Total network packets transmitted. | double |  | counter |
| supabase.node.nf_conntrack.entries | Current number of connection tracking entries. | double |  | gauge |
| supabase.node.nf_conntrack.entries_limit | Maximum connection tracking table size. | double |  | gauge |
| supabase.node.power_supply_online | Whether the power supply is online (0/1). | double |  | gauge |
| supabase.node.procs_blocked | Number of processes blocked waiting for I/O. | double |  | gauge |
| supabase.node.procs_running | Number of processes in runnable state. | double |  | gauge |
| supabase.node.scrape_collector_duration_seconds | Duration of node exporter collector scrape. | double | s | gauge |
| supabase.node.scrape_collector_success | Whether the node exporter collector scrape succeeded (0/1). | double |  | gauge |
| supabase.node.vmstat.oom_kill | OOM killer invocations. | double |  | gauge |
| supabase.node.vmstat.pgfault | Page faults (minor + major). | double |  | gauge |
| supabase.node.vmstat.pgmajfault | Major page faults. | double |  | gauge |
| supabase.node.vmstat.pgpgin | Pages paged in from disk. | double |  | gauge |
| supabase.node.vmstat.pgpgout | Pages paged out to disk. | double |  | gauge |
| supabase.node.vmstat.pswpin | Pages swapped in. | double |  | gauge |
| supabase.node.vmstat.pswpout | Pages swapped out. | double |  | gauge |


### PostgreSQL Metrics

**Exported fields**

| Field | Description | Type | Unit | Metric Type |
|---|---|---|---|---|
| @timestamp | Event timestamp. | date |  |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |  |
| data_stream.type | Data stream type. | constant_keyword |  |  |
| supabase.connection_stats.connection_count | Total connection count (pooled). | double |  | gauge |
| supabase.direct_connection_stats.connection_count | Direct (non-pooled) connection count. | double |  | gauge |
| supabase.labels.instance | Supabase project instance identifier. | keyword |  |  |
| supabase.labels.job | Prometheus scrape job name. | keyword |  |  |
| supabase.max_connections.connection_count | Maximum allowed connections. | double |  | gauge |
| supabase.pg.up | Whether the last scrape of metrics from PostgreSQL was able to connect. | double |  | gauge |
| supabase.pg_database_size.bytes | Disk space used by the database. | double | byte | gauge |
| supabase.pg_database_size.mb | Disk space used by the database in MB. | double |  | gauge |
| supabase.pg_exporter.last_scrape_duration_seconds | Last scrape duration. | double | s | gauge |
| supabase.pg_exporter.last_scrape_error | Whether last scrape had an error. | double |  | gauge |
| supabase.pg_exporter.scrapes.count | Total scrapes performed by the postgres exporter. | double |  | counter |
| supabase.pg_ls.archive_statusdir_wal_pending.count | Number of not yet archived WAL files. | double |  | counter |
| supabase.pg_replication_slots.max_lag_bytes | Maximum replication lag across all replication slots. | double | byte | gauge |
| supabase.pg_settings.default_transaction_read_only | Default transaction read-only setting. | double |  | gauge |
| supabase.pg_stat_activity.xact_runtime | Current transaction runtime. | double | s | gauge |
| supabase.pg_stat_bgwriter.buffers_alloc.count | Buffers allocated. | double |  | counter |
| supabase.pg_stat_bgwriter.buffers_backend.count | Buffers written directly by a backend. | double |  | counter |
| supabase.pg_stat_bgwriter.buffers_checkpoint.count | Buffers written during checkpoints. | double |  | counter |
| supabase.pg_stat_bgwriter.buffers_clean.count | Buffers written by the background writer. | double |  | counter |
| supabase.pg_stat_bgwriter.checkpoint_sync_time.count | Time syncing checkpoint files to disk. | double | ms | counter |
| supabase.pg_stat_bgwriter.checkpoint_write_time.count | Time writing checkpoint files to disk. | double | ms | counter |
| supabase.pg_stat_bgwriter.checkpoints_req.count | Requested checkpoints performed. | double |  | counter |
| supabase.pg_stat_bgwriter.checkpoints_timed.count | Scheduled checkpoints performed. | double |  | counter |
| supabase.pg_stat_bgwriter.maxwritten_clean.count | Times bgwriter stopped due to buffer write limit. | double |  | counter |
| supabase.pg_stat_database.blks_hit.count | Disk blocks found in buffer cache (cache hits). | double |  | counter |
| supabase.pg_stat_database.blks_read.count | Number of disk blocks read (cache misses). | double |  | counter |
| supabase.pg_stat_database.conflicts.count | Queries canceled due to conflicts with recovery. | double |  | counter |
| supabase.pg_stat_database.conflicts_confl_bufferpin.count | Conflicts due to buffer pin. | double |  | counter |
| supabase.pg_stat_database.conflicts_confl_deadlock.count | Conflicts due to deadlock. | double |  | counter |
| supabase.pg_stat_database.conflicts_confl_lock.count | Conflicts due to lock. | double |  | counter |
| supabase.pg_stat_database.conflicts_confl_snapshot.count | Conflicts due to snapshot. | double |  | counter |
| supabase.pg_stat_database.conflicts_confl_tablespace.count | Conflicts due to tablespace. | double |  | counter |
| supabase.pg_stat_database.deadlocks.count | Total deadlocks detected. | double |  | counter |
| supabase.pg_stat_database.num_backends | Number of active backends (connections) to the database. | double |  | gauge |
| supabase.pg_stat_database.temp_bytes.count | Temp data written by queries. | double | byte | counter |
| supabase.pg_stat_database.temp_files.count | Temp files created by queries. | double |  | counter |
| supabase.pg_stat_database.tup_deleted.count | Total rows deleted. | double |  | counter |
| supabase.pg_stat_database.tup_fetched.count | Total rows fetched by queries. | double |  | counter |
| supabase.pg_stat_database.tup_inserted.count | Total rows inserted. | double |  | counter |
| supabase.pg_stat_database.tup_returned.count | Total rows returned by queries. | double |  | counter |
| supabase.pg_stat_database.tup_updated.count | Total rows updated. | double |  | counter |
| supabase.pg_stat_database.xact_commit.count | Total transactions committed. | double |  | counter |
| supabase.pg_stat_database.xact_rollback.count | Total transactions rolled back. | double |  | counter |
| supabase.pg_stat_replication.replay_lag | Replication replay lag. | double | s | gauge |
| supabase.pg_stat_replication.send_lag | Replication send lag. | double | s | gauge |
| supabase.pg_stat_statements.total_queries.count | Number of times queries were executed. | double |  | counter |
| supabase.pg_stat_statements.total_time_seconds.count | Total time spent executing queries. | double | s | counter |
| supabase.pg_status.in_recovery | Whether the database is in recovery mode. | double |  | gauge |
| supabase.pg_wal.size | Disk space used by WAL files. | double | byte | gauge |
| supabase.pg_wal.size_mb | WAL files disk usage in MB. | double |  | gauge |
| supabase.physical_replication_lag.is_connected_to_primary | Whether the replica is connected to the primary database. | double |  | gauge |
| supabase.physical_replication_lag.is_wal_replay_paused | Whether WAL replay has been paused. | double |  | gauge |
| supabase.physical_replication_lag_seconds | Physical replication lag in seconds. | double | s | gauge |
| supabase.postgresql.restarts.count | Number of times PostgreSQL has been restarted. | double |  | counter |
| supabase.replication_realtime.lag_bytes | Realtime replication lag in bytes. | double | byte | gauge |
| supabase.replication_realtime.slot_status | Realtime replication slot status. | double |  | gauge |
| supabase.usage_metrics.user_queries.count | Total user queries (usage tracking). | double |  | counter |


### Service Metrics

**Exported fields**

| Field | Description | Type | Unit | Metric Type |
|---|---|---|---|---|
| @timestamp | Event timestamp. | date |  |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |  |
| data_stream.type | Data stream type. | constant_keyword |  |  |
| supabase.auth_users.user_count | Number of users in the project database. | double |  | gauge |
| supabase.db.sql.connection_closed_max_idle.count | Total connections closed due to SetConnMaxIdleTime. | double |  | counter |
| supabase.db.sql.connection_closed_max_lifetime.count | Total connections closed due to SetConnMaxLifetime. | double |  | counter |
| supabase.db.sql.connection_max_open | Maximum number of open connections to the database. | double |  | gauge |
| supabase.db.sql.connection_open | Number of established connections both in use and idle. | double |  | gauge |
| supabase.db.sql.connection_wait.count | Total number of connections waited for. | double |  | counter |
| supabase.db.transmit_bytes.count | Postgres and PgBouncer network transmit bytes. | double | byte | counter |
| supabase.global_auth_version.major | GoTrue major version. | double |  | gauge |
| supabase.global_auth_version.minor | GoTrue minor version. | double |  | gauge |
| supabase.global_auth_version.patch | GoTrue patch version. | double |  | gauge |
| supabase.go.goroutines | Active goroutines. | double |  | gauge |
| supabase.go.memstats.alloc_bytes | Bytes allocated and in use. | double | byte | gauge |
| supabase.go.memstats.gc_sys_bytes | GC metadata bytes. | double | byte | gauge |
| supabase.go.memstats.heap_alloc_bytes | Heap bytes allocated and in use. | double | byte | gauge |
| supabase.go.memstats.heap_idle_bytes | Heap bytes waiting to be used. | double | byte | gauge |
| supabase.go.memstats.heap_inuse_bytes | Heap bytes in use. | double | byte | gauge |
| supabase.go.memstats.heap_objects | Allocated heap objects. | double |  | gauge |
| supabase.go.memstats.heap_sys_bytes | Heap bytes from system. | double | byte | gauge |
| supabase.go.memstats.next_gc_bytes | Heap size at next GC. | double | byte | gauge |
| supabase.go.memstats.stack_inuse_bytes | Stack bytes in use. | double | byte | gauge |
| supabase.go.memstats.sys_bytes | Bytes obtained from system. | double | byte | gauge |
| supabase.go.threads | OS threads created. | double |  | gauge |
| supabase.gotrue.compare_hash_and_password_completed.count | Password comparison attempts completed. | double |  | counter |
| supabase.gotrue.compare_hash_and_password_submitted.count | Password comparison attempts submitted. | double |  | counter |
| supabase.gotrue.generate_from_password_completed.count | Password hash generation attempts completed. | double |  | counter |
| supabase.gotrue.generate_from_password_submitted.count | Password hash generation attempts submitted. | double |  | counter |
| supabase.gotrue.running | GoTrue service health (1 when running). | double |  | gauge |
| supabase.http.server.request_duration_seconds.bucket | Histogram bucket for HTTP request duration. | double |  |  |
| supabase.http.server.request_duration_seconds.count | Total HTTP requests. | double |  |  |
| supabase.http.server.request_duration_seconds.sum | Total HTTP request duration. | double |  |  |
| supabase.http.status_codes.count | HTTP response status codes by route. | double |  | counter |
| supabase.labels.instance | Supabase project instance identifier. | keyword |  |  |
| supabase.labels.job | Prometheus scrape job name. | keyword |  |  |
| supabase.pgrst.db_pool.available_connections | Available connections in the PostgREST pool. | double |  | gauge |
| supabase.pgrst.db_pool.connection_timeouts.count | Total number of PostgREST pool connection timeouts. | double |  | counter |
| supabase.pgrst.db_pool.max_connections | Maximum pool connections for PostgREST. | double |  | gauge |
| supabase.pgrst.db_pool.requests_waiting | Requests waiting to acquire a PostgREST pool connection. | double |  | gauge |
| supabase.pgrst.jwt_cache.evictions.count | Total JWT cache evictions. | double |  | counter |
| supabase.pgrst.jwt_cache.hits.count | Total JWT cache hits. | double |  | counter |
| supabase.pgrst.jwt_cache.requests.count | Total JWT cache requests. | double |  | counter |
| supabase.pgrst.schema_cache.loads.count | Total number of times the PostgREST schema cache was loaded. | double |  | counter |
| supabase.pgrst.schema_cache.query_time_seconds | Query time of the last PostgREST schema cache load. | double | s | gauge |
| supabase.process.cpu_seconds.count | Total CPU time. | double | s | counter |
| supabase.process.max_fds | Max file descriptors. | double |  | gauge |
| supabase.process.open_fds | Open file descriptors. | double |  | gauge |
| supabase.process.resident_memory_bytes | Resident memory (RSS). | double | byte | gauge |
| supabase.process.virtual_memory_bytes | Virtual memory. | double | byte | gauge |
| supabase.realtime_postgres_changes.client_subscriptions | Client subscriptions listening for Postgres changes. | double |  | gauge |
| supabase.realtime_postgres_changes.total_subscriptions | Total subscription records listening for Postgres changes. | double |  | gauge |
| supabase.storage.storage_size | Total size used for all storage buckets. | double | byte | gauge |
| supabase.supavisor.client.connection.duration.bucket | Histogram bucket for client connection duration. | double |  |  |
| supabase.supavisor.client.connection.duration.count | Total count of client connections. | double |  |  |
| supabase.supavisor.client.connection.duration.sum | Total client connection duration. | double |  |  |
| supabase.supavisor.client.joins.fail.count | Total failed client joins to the pooler. | double |  | counter |
| supabase.supavisor.client.joins.ok.count | Total successful client joins to the pooler. | double |  | counter |
| supabase.supavisor.client.network.recv.count | Total bytes received from clients. | double | byte | counter |
| supabase.supavisor.client.network.send.count | Total bytes sent to clients. | double | byte | counter |
| supabase.supavisor.client.queries_count.count | Total queries received from clients. | double |  | counter |
| supabase.supavisor.client.query.duration.bucket | Histogram bucket for client query duration. | double |  |  |
| supabase.supavisor.client.query.duration.count | Total count of client queries. | double |  |  |
| supabase.supavisor.client.query.duration.sum | Total time spent processing client queries. | double |  |  |
| supabase.supavisor.connections.active | Active client connections to the Supavisor pooler for a tenant. | double |  | gauge |
| supabase.supavisor.db.handler.started.count | Database handler processes started. | double |  | counter |
| supabase.supavisor.db.handler.stopped.count | Database handler processes stopped. | double |  | counter |
| supabase.supavisor.db.network.recv.count | Total bytes received from the database. | double | byte | counter |
| supabase.supavisor.db.network.send.count | Total bytes sent to the database. | double | byte | counter |
| supabase.supavisor.pool.checkout.duration.local.bucket | Histogram bucket for local pool checkout latency. | double |  |  |
| supabase.supavisor.pool.checkout.duration.local.count | Total count of local pool checkouts. | double |  |  |
| supabase.supavisor.pool.checkout.duration.local.sum | Total local pool checkout duration. | double |  |  |
| supabase.supavisor.pool.checkout.duration.remote.bucket | Histogram bucket for remote pool checkout latency. | double |  |  |
| supabase.supavisor.pool.checkout.duration.remote.count | Total count of remote pool checkouts. | double |  |  |
| supabase.supavisor.pool.checkout.duration.remote.sum | Total remote pool checkout duration. | double |  |  |
| supabase.supavisor.pool.connections.checked_out | Checked-out (busy) Supavisor pool connections per tenant. | double |  | gauge |
| supabase.supavisor.pool.connections.idle | Idle Supavisor pool connections per tenant. | double |  | gauge |
| supabase.supavisor.tenants.active | Total active tenants in Supavisor. | double |  | gauge |

