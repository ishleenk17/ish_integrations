# Prometheus Scrape (Native Schema)

## Overview

This package scrapes Prometheus-format metrics endpoints with Elastic Agent and stores them
so that **Prometheus metric names and labels stay queryable by PromQL**. Existing PromQL
expressions, including the label names your dashboards and alert rules group by, resolve
without being rewritten.

It combines the two configuration styles of the earlier Prometheus OTel input packages:
guided fields for straightforward targets, and a full `scrape_configs` escape hatch for
existing Prometheus deployments that rely on service discovery or relabeling.

## Why this package exists

The OpenTelemetry Prometheus receiver translates two Prometheus labels into OTel semantic
conventions while scraping:

| Prometheus label | Becomes |
| --- | --- |
| `job` | `resource.attributes["service.name"]` |
| `instance` | `resource.attributes["service.instance.id"]` |

The Elasticsearch `PROMQL` command does not reverse that translation, and the resulting
failure is silent rather than an error:

- `sum by (job) (...)` returns a single ungrouped total, with no `job` column.
- `some_metric{job="api"}` matches nothing and the panel renders empty.

This package restores both as datapoint attributes, so PromQL resolves them normally. All
other scraped labels already work unmodified, as does the bare metric name.

## Configuration

### Guided configuration

Set **Job Name** and **Scrape Targets** and you are done. Job Name is preserved as the
`job` label, so set it to the value your existing dashboards group by — do not leave every
integration policy on the same job name, or `sum by (job)` will aggregate unrelated targets
into one series.

Optional **Relabel Configs** and **Metric Relabel Configs** fields accept the same YAML lists
as `prometheus.yml`. Use `metric_relabel_configs` to drop high-cardinality metrics before
they are stored.

### Advanced configuration

Paste `scrape_configs` from an existing `prometheus.yml` into **Advanced: Full Scrape
Configuration**. It is used verbatim, so multiple jobs, `kubernetes_sd_configs`,
`ec2_sd_configs`, `file_sd_configs` and relabeling all work:

```yaml
- job_name: node-exporter
  scrape_interval: 15s
  static_configs:
    - targets: ['node-a:9100', 'node-b:9100']
- job_name: kube-pods
  kubernetes_sd_configs:
    - role: pod
  relabel_configs:
    - source_labels: [__meta_kubernetes_namespace]
      target_label: namespace
```

When this field is set it replaces every guided field. Credentials embedded here cannot yet
be stored as Fleet secrets, so prefer the guided **Username** / **Password** / **Bearer
Token** fields when scraping authenticated endpoints.

## Querying with PromQL

Query the stored metrics with the `PROMQL` source command in ES|QL:

```
PROMQL index=metrics-*.otel-* sum by (job, instance) (rate(http_requests_total[5m]))
```

Metric names, scraped labels, and `job` / `instance` all resolve as written.

### Known limitations

- `PROMQL` is an ES|QL source command, not the Prometheus HTTP API. PromQL expressions carry
  over unchanged, but a Grafana dashboard must be repointed at the Elasticsearch data source;
  Elasticsearch cannot yet be used as a Grafana Prometheus data source.
- Some PromQL functions are not yet supported, most notably `histogram_quantile`,
  `predict_linear` and `label_join`, along with the binary set operators `or`, `and` and
  `unless`.

## Compatibility

Requires Kibana 9.2.0 or later and an Elastic Agent with OpenTelemetry Collector support.
PromQL querying requires Elasticsearch 9.4 or later, or Elastic Cloud Serverless.
