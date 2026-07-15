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

{{fields "node"}}

### PostgreSQL Metrics

{{fields "postgres"}}

### Service Metrics

{{fields "services"}}
