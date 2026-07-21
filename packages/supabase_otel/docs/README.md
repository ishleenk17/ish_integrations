# Supabase OpenTelemetry Assets

This package provides Kibana Assets for visualizing Supabase metrics collected by the **Supabase** integration.

## Dashboards

| Dashboard | Description |
|---|---|
| **[Supabase OTel] Node & Infrastructure** | Node-level infrastructure metrics including memory, network, CPU, disk I/O, and filesystem. |
| **[Supabase OTel] Databases & PostgreSQL** | Database internals, PgBouncer connection pooling, Go SQL connection pool, and replication & WAL. |
| **[Supabase OTel] Services & API** | Application service metrics for Auth (GoTrue), PostgREST, Realtime, and Go runtime. |

## Prerequisites

- Install the **Supabase** integration package and configure it with your Supabase project credentials. These dashboards query data collected by that integration.
- Data must be ingested into the `metrics-supabase.metrics.otel-*` index pattern before the dashboards will populate.
